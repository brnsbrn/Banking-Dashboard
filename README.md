# Laporan Analisis Business Intelligence
## Banking Performance, Credit Funnel, Marketing Efficiency, AO & Branch Performance, serta Product & Customer Insights

> **Catatan konteks data:** dataset yang digunakan bersifat sintetis dan ditujukan untuk pembelajaran/portfolio. Karena itu, seluruh insight dalam laporan ini merupakan simulasi analisis bisnis dan tidak merepresentasikan kondisi bank riil. Periode utama data adalah Juli 2025–Juni 2026, dengan 7 cabang, 28 Account Officer, 50.000 prospek, 10.872 aplikasi, 7.528 aplikasi disetujui, 6.510 pencairan, total pencairan Rp860,905 miliar, dan marketing spend Rp3,495 miliar.

---

# 1. Business Problem

Masalah bisnis utama bukan sekadar bagaimana meningkatkan jumlah prospek, tetapi bagaimana memastikan bahwa aktivitas akuisisi menghasilkan **aplikasi yang berkualitas, approval yang tinggi, pencairan yang optimal, serta biaya akuisisi yang efisien**. Volume prospek yang besar tidak otomatis menghasilkan pencairan yang besar apabila kualitas sumber prospek rendah, tindak lanjut tidak efektif, kualitas aplikasi lemah, atau terdapat leakage setelah approval.

Terdapat lima pertanyaan bisnis utama yang harus dijawab oleh dashboard:

1. Seberapa efektif 50.000 prospek dikonversi menjadi aplikasi, approval, dan akhirnya pencairan?
2. Channel atau source mana yang menghasilkan volume besar tetapi kualitas funnel rendah, dan source mana yang memberikan kualitas terbaik?
3. AO dan cabang mana yang benar-benar produktif setelah mempertimbangkan conversion, approval, dan realisasi pencairan, bukan hanya volume prospek?
4. Produk mana yang menghasilkan nominal pencairan terbesar, produk mana yang paling efisien secara funnel, dan bagaimana trade-off antara volume, approval, ticket size, rate, dan tenor?
5. Apakah marketing spend sebesar Rp3,495 miliar dialokasikan ke channel yang menghasilkan nilai bisnis terbaik?

Masalah tambahan yang ditemukan dari hasil audit dashboard adalah **ketepatan filter context pada beberapa visual lintas fact table**. Pada halaman Source Funnel, Marketing Efficiency, dan Product Performance, beberapa metrik downstream seperti Total Applications, Approved Applications, dan Total Disbursement muncul berulang dengan nilai grand total untuk setiap Source/Product. Ini menunjukkan bahwa beberapa visual belum memiliki jalur filter yang benar antar tabel fakta. Artinya, desain visualnya sudah informatif, tetapi sebagian scorecard belum boleh dijadikan dasar keputusan sebelum model diperbaiki.

---

# 2. Business Understanding

Tujuan bisnis yang tepat adalah memaksimalkan **quality-adjusted disbursement**, bukan sekadar memaksimalkan lead. Oleh karena itu, kinerja harus dibaca sebagai satu rantai nilai:

**Prospect → Application → Approval → Disbursement → Portfolio Performance**

Dalam dataset saat ini, analisis dapat dilakukan sampai tahap Disbursement. Namun, data pasca pencairan seperti outstanding, DPD, kolektibilitas, NPL, repayment behavior, write-off, cost of fund, dan margin bersih belum tersedia. Karena itu, dashboard dapat menilai **acquisition quality dan funnel quality**, tetapi belum dapat menyimpulkan profitabilitas atau kualitas risiko kredit setelah pencairan.

Secara manajerial, indikator dibagi menjadi enam perspektif:

- **Acquisition:** jumlah prospek, source, segment, occupation, product, branch, AO.
- **Conversion:** Prospect → Application.
- **Credit Decision:** approval, rejection, cancellation, processing time.
- **Realization:** approved → disbursed dan nilai pencairan.
- **Efficiency:** ad spend, cost per prospect, cost per application, cost per disbursement, disbursement-to-spend ratio.
- **Productivity:** perbandingan AO dan cabang berdasarkan output funnel, bukan aktivitas semata.

Dengan kerangka ini, dashboard tidak hanya menjadi alat reporting, tetapi menjadi alat diagnosis: **di mana leakage terjadi, siapa yang berkontribusi, source apa yang berkualitas, serta intervensi apa yang harus dilakukan.**

---

# 3. Data Understanding

Dataset terdiri dari dua tabel dimensi utama dan empat fact table:

| Tabel | Peran | Informasi utama |
|---|---|---|
| `dim_branch` | Dimensi cabang | branch_id, branch_name, city, region |
| `dim_ao` | Dimensi AO | ao_id, ao_name, branch, status |
| `fact_campaign_spend` | Aktivitas marketing | tanggal campaign, branch, source, campaign type, campaign name, ad spend |
| `fact_prospects` | Funnel awal | prospect_id, tanggal prospek, AO, branch, source, segment, occupation, age, income, requested amount, tenor, product, prospect status |
| `fact_applications` | Pengajuan kredit | application_id, prospect_id, application/decision date, AO, branch, product, segment, requested amount, application status, processing days |
| `fact_disbursements` | Realisasi kredit | disbursement_id, application_id, prospect_id, date, branch, AO, product, segment, disbursed amount, interest rate, tenor |

### Skala data

- Prospects: **50.000**
- Applications: **10.872**
- Approved applications: **7.528**
- Disbursements: **6.510**
- Total disbursement: **Rp860,905 miliar**
- Campaign records: **528**
- Marketing spend: **Rp3,495 miliar**
- Branches: **7**
- Account Officers: **28**

### Data quality yang terverifikasi

Pemeriksaan terhadap CSV menunjukkan tidak terdapat missing value pada tabel utama, tidak terdapat duplicate primary key, dan seluruh referential key utama konsisten: seluruh application memiliki prospect yang valid, seluruh disbursement memiliki application dan prospect yang valid, serta seluruh AO dan branch ID pada fact table memiliki pasangan pada tabel dimensinya.

Periode prospek, aplikasi, dan campaign berada pada rentang Juli 2025–Juni 2026. Tabel disbursement memiliki sebagian kecil transaksi sampai **1 Juli 2026**, yang secara logis merupakan efek lag proses dari approval akhir Juni. Hal ini penting saat membuat monthly trend agar bulan akhir tidak salah dibaca sebagai periode penuh.

---

# 4. Data Preparation

Tahapan data preparation yang digunakan untuk menghasilkan analisis yang konsisten meliputi:

1. **Validasi key dan referential integrity** untuk prospect_id, application_id, ao_id, dan branch_id.
2. **Konversi kolom tanggal** menjadi date type agar dapat digunakan untuk period analysis dan aging.
3. **Standardisasi kategori** Source, Product, Segment, Application Status, Prospect Status, Occupation, Branch, dan AO.
4. **Pemetaan Source ke downstream funnel** melalui `prospect_id`. Hal ini diperlukan karena `source` tersedia pada `fact_prospects`, sedangkan application dan disbursement tidak memiliki source secara langsung.
5. **Pembuatan derived metrics**, antara lain:
   - Prospect-to-Application Conversion Rate
   - Approval Rate
   - Approved-to-Disbursement Rate
   - End-to-End Prospect-to-Disbursement Rate
   - Average Ticket Size
   - Cost per Prospect
   - Cost per Application
   - Cost per Disbursement
   - Disbursement-to-Spend Ratio
   - Average/Weighted Interest Rate
   - Average/Weighted Tenor
   - Funding Fulfillment Ratio
   - Approved Not Disbursed Aging
6. **Pembuatan Year-Month sort key** agar urutan bulan mengikuti kronologi Juli 2025 → Juni 2026, bukan January → December tanpa konteks tahun.
7. **Pemisahan cohort date dan transaction date**. Monthly funnel sebaiknya menggunakan prospect cohort apabila tujuan analisis adalah conversion; sedangkan transaction trend menggunakan application/disbursement date apabila tujuan analisis adalah workload atau realisasi bulanan.

---

# 5. Data Modeling

Model saat ini sudah memiliki `dim_branch`, `dim_ao`, fact tables, dan `DimDate`. Namun, hasil dashboard menunjukkan bahwa **Source, Product, dan Segment belum berfungsi sebagai shared dimensions lintas fact table**.

Contoh masalah yang terlihat langsung pada dashboard:

- Pada `Source Funnel Scorecard`, Total Applications = 10.872, Approved Applications = 7.528, dan Total Disbursement = Rp860,905 miliar muncul berulang pada hampir setiap Source.
- Pada halaman `Marketing Efficiency`, bar `Applications by Source` dan `Disbursement by Source` mempunyai nilai yang sama untuk setiap source.
- Pada `Product Performance Scorecard`, Total Applications, Approved Applications, Approval Rate, dan Total Disbursement berulang dengan nilai grand total untuk setiap Product.

Secara teknis, hal tersebut terjadi ketika kolom kategori berasal dari satu fact table, sedangkan measure berasal dari fact table lain tanpa shared dimension atau filter propagation yang valid.

### Model yang direkomendasikan

Gunakan model star schema dengan dimensi bersama:

- `DimDate`
- `DimBranch`
- `DimAO`
- `DimSource`
- `DimProduct`
- `DimSegment`
- opsional `DimProspect` untuk atribut prospect-level dan source lineage

Fact tables:

- `FactProspect`
- `FactApplication`
- `FactDisbursement`
- `FactCampaignSpend`

`DimProduct` dan `DimSegment` dapat dihubungkan one-to-many ke Prospect, Application, dan Disbursement. Untuk Source, source dapat diturunkan ke Application dan Disbursement saat Power Query melalui `prospect_id`, kemudian seluruh fact menggunakan `DimSource` yang sama. Alternatifnya, Source dapat mengalir melalui `DimProspect`, tetapi desain harus tetap menghindari many-to-many dan bidirectional relationship yang tidak terkontrol.

Relationship ideal adalah **one-to-many, single direction dari dimension ke fact**. Untuk tanggal Application Date dan Decision Date, gunakan role-playing date atau inactive relationship dengan measure khusus. Ini menjaga filter context tetap deterministik dan menghindari angka grand total yang berulang pada kategori.

---

# 6. Analysis

## 6.1 Overall Funnel Performance

| Stage | Jumlah | Rate dari tahap sebelumnya |
|---|---:|---:|
| Prospect | 50.000 | - |
| Application | 10.872 | **21,74%** |
| Approved | 7.528 | **69,24%** |
| Disbursed | 6.510 | **86,48% dari approved** |

End-to-end Prospect → Disbursement adalah **13,02%**. Artinya, dari setiap 100 prospek yang masuk, secara rata-rata hanya sekitar 13 yang berakhir sebagai pencairan.

Funnel menunjukkan tiga leakage utama:

- **39.128 prospek** tidak menjadi application.
- **3.344 application** tidak mencapai approval, terdiri dari rejected dan cancelled.
- **1.018 approved application** belum memiliki disbursement yang tercatat dalam periode data.

Prospect status menunjukkan **9.869 Follow-up Pending**, setara **19,74%** seluruh prospek. Angka ini penting karena dapat merepresentasikan pipeline produktif atau justru backlog yang menua. Tanpa aging, angka pending tidak dapat langsung dianggap sebagai peluang.

Dari 1.018 approved yang belum tercatat disbursed, **909 kasus berusia lebih dari 30 hari** dihitung dari tanggal keputusan sampai tanggal maksimum data disbursement. Nilai requested amount kelompok ini sekitar **Rp134,325 miliar**. Ini merupakan pipeline at risk yang jauh lebih actionable dibanding hanya melihat approval rate.

Untuk kredit yang akhirnya cair, rata-rata durasi proses adalah:

- Prospect → Application: **1,95 hari**
- Application → Decision: **3,27 hari**
- Decision → Disbursement: **2,97 hari**
- Prospect → Disbursement: **8,20 hari**

Dengan demikian, bottleneck utama secara proses bukan terlihat pada durasi keputusan yang ekstrem, melainkan pada **jumlah prospek yang tidak berhasil naik tahap dan approved case yang tidak berakhir pada pencairan**.

## 6.2 Marketing & Source Quality

Secara total, marketing spend sebesar **Rp3,495 miliar** menghasilkan:

- Cost per Prospect: **Rp69,9 ribu**
- Cost per Application: **Rp321,5 ribu**
- Cost per Disbursement: **Rp536,9 ribu**
- Disbursement-to-Spend Ratio: **246,3x**

Namun, distribusi efisiensi antar source sangat tidak merata.

| Source | Prospect | Conv. | Approval | E2E | Spend | Cost/Disb. | Disb./Spend |
|---|---:|---:|---:|---:|---:|---:|---:|
| Referral | 6.922 | **30,94%** | **82,21%** | **24,91%** | Rp163,2M | **Rp94,7K** | **1.429x** |
| Event | 4.539 | 28,51% | 81,92% | 22,47% | Rp219,8M | Rp215,5K | 633x |
| Walk-in | 5.481 | 26,66% | 77,62% | 19,50% | Rp68,7M | **Rp64,2K** | **1.917x** |
| WhatsApp | 8.571 | 21,71% | 70,50% | 12,79% | Rp596,2M | Rp544,0K | 236x |
| Telemarketing | 4.796 | 18,72% | 60,91% | 8,61% | Rp249,5M | Rp604,0K | 219x |
| Instagram Ads | 9.064 | 16,63% | 55,67% | 6,53% | Rp737,1M | Rp1,245M | 108x |
| Facebook Ads | 10.627 | **16,08%** | **51,20%** | **5,61%** | **Rp1,461B** | **Rp2,451M** | **56x** |

Temuan paling kritis adalah **Facebook Ads menghasilkan volume prospect tertinggi tetapi kualitas downstream terendah**. Facebook menggunakan **41,80% dari total spend**, menghasilkan 21,25% prospek, tetapi hanya berkontribusi sekitar **9,52% total nominal disbursement**.

Sebaliknya, Referral hanya menggunakan sekitar **4,67% spend**, tetapi menghasilkan sekitar **27,10% nominal disbursement**. Walk-in hanya menggunakan sekitar 1,96% spend dan menghasilkan sekitar 15,29% nominal disbursement.

Pada level campaign type, Digital menggunakan **79,94% total spend** namun menghasilkan sekitar **35,12% total nominal disbursement**. Organic/Referral + Organic/Branch menggunakan sekitar **6,63% spend** tetapi menyumbang sekitar **42,38% nominal disbursement**.

Interpretasinya bukan bahwa anggaran digital harus dihentikan. Referral dan walk-in memiliki keterbatasan skalabilitas dan tidak dapat diasumsikan mampu menyerap budget secara linear. Tetapi data secara tegas menunjukkan bahwa **channel optimization harus beralih dari cost-per-lead ke quality-adjusted acquisition cost**.

## 6.3 Branch Performance

| Branch | Conversion | Approval | E2E | Disbursement | Cost/Disb. |
|---|---:|---:|---:|---:|---:|
| Samarinda Pusat | **23,66%** | **72,65%** | **15,00%** | **Rp188,109B** | Rp357K |
| Samarinda Utara | 22,18% | 71,81% | 14,11% | Rp142,919B | Rp545K |
| Bontang | 22,37% | 69,51% | 13,55% | Rp117,629B | Rp522K |
| Tenggarong | 21,50% | 70,00% | 12,97% | Rp112,475B | Rp597K |
| Sangatta | 21,48% | 66,49% | 12,12% | Rp98,907B | Rp626K |
| Balikpapan Tengah | 20,37% | 67,20% | 11,81% | Rp125,240B | Rp556K |
| Kutai Barat | **19,82%** | **63,44%** | **10,42%** | **Rp75,628B** | **Rp729K** |

Samarinda Pusat merupakan benchmark terbaik: nominal pencairan tertinggi, funnel rate tertinggi, serta cost per disbursement paling rendah. Kutai Barat berada pada sisi berlawanan: conversion, approval, E2E, dan nominal disbursement terendah dengan cost per disbursement tertinggi.

Namun, perbedaan cabang tidak boleh langsung disimpulkan sebagai masalah eksekusi. Perlu dikontrol terhadap product mix, source mix, karakteristik pasar lokal, dan profil nasabah. Dashboard berikutnya idealnya menggunakan **peer-adjusted benchmark**, sehingga cabang dibandingkan pada kondisi yang sebanding.

## 6.4 AO Performance

Fajar Nugroho adalah performer terkuat secara gabungan dengan:

- 2.257 prospect
- 604 application
- Conversion **26,76%**
- Approval **75,83%**
- 418 disbursement
- E2E **18,52%**
- Total disbursement **Rp60,210 miliar**

Sebaliknya, Rani Sari dan Raka Prabowo memiliki E2E masing-masing **8,05%** dan **8,38%**, dengan total disbursement sekitar Rp14,0–15,0 miliar. Keduanya berada di Kutai Barat, yaitu cabang dengan kinerja funnel terendah. Hal ini mengindikasikan bahwa **AO performance harus dianalisis bersama branch context**; tidak tepat langsung mengatribusi seluruh gap kepada individu.

Contoh lain adalah Dimas Fadillah. Ia berada di Samarinda Pusat, cabang benchmark tertinggi, namun E2E-nya sekitar **11,94%**, jauh di bawah Fajar Nugroho pada cabang yang sama. Kasus seperti ini lebih kuat sebagai kandidat coaching individual karena efek lingkungan cabang relatif lebih terkontrol.

Dengan demikian, evaluasi AO sebaiknya menggunakan kombinasi:

- volume prospect,
- conversion,
- approval,
- E2E,
- total disbursement,
- average ticket,
- product/source mix,
- serta benchmark terhadap peer dalam cabang yang sama.

## 6.5 Product Performance

| Product | Prospect | Conversion | Approval | E2E | Disbursement | Avg Ticket |
|---|---:|---:|---:|---:|---:|---:|
| Kredit Produktif SME | 9.146 | 20,49% | 66,17% | 11,28% | **Rp386,426B** | **Rp374,4M** |
| Kredit Payroll | 10.676 | **26,87%** | **77,52%** | **17,88%** | Rp214,335B | Rp112,3M |
| Kredit Konsumtif | 16.754 | 21,76% | 68,51% | 13,17% | Rp190,220B | Rp86,2M |
| Kredit Mikro | 13.424 | **18,50%** | **63,07%** | **10,15%** | Rp69,925B | Rp51,3M |

Kredit Produktif SME adalah **value driver**: hanya sekitar 18,3% dari prospek, tetapi menghasilkan sekitar **44,89% total nominal pencairan** karena average ticket sangat tinggi. Konsekuensinya, funnel tidak seefisien Payroll dan approval lebih rendah.

Payroll merupakan produk dengan **funnel quality terbaik**, dengan conversion 26,87%, approval 77,52%, dan E2E 17,88%. Ini menjadikannya kandidat kuat untuk scalable growth apabila kualitas portofolio pasca pencairan juga baik.

Mikro memiliki funnel terlemah dan ticket size paling kecil. Walaupun rate nominal lebih tinggi, dataset belum memiliki cost of risk, collection cost, NPL, dan operating cost sehingga tidak dapat disimpulkan apakah Mikro lebih atau kurang profitable.

Weighted average interest rate seluruh pencairan adalah sekitar **18,07% p.a.**, sedangkan weighted average tenor sekitar **32,1 bulan**. Funding fulfillment ratio, yaitu disbursed amount terhadap requested amount untuk kasus yang cair, sekitar **96,06%**, menunjukkan haircut nominal relatif moderat.

## 6.6 Customer Profile

Wiraswasta adalah occupation terbesar dengan sekitar **24,88%** seluruh prospek. Namun, volume tidak identik dengan kualitas. Karyawan BUMN dan Dokter/Tenaga Medis memiliki E2E sekitar **15,72%** dan **15,96%**, lebih tinggi dibanding Wiraswasta sekitar 12,47%.

Tenaga Kontrak memiliki E2E sekitar **8,58%**, paling rendah di antara occupation utama. Informasi ini sebaiknya digunakan untuk memahami profil akuisisi dan kebutuhan produk, bukan sebagai dasar otomatis untuk menolak kelompok tertentu. Keputusan kredit tetap harus mengikuti kebijakan underwriting dan prinsip fairness yang berlaku.

Berdasarkan income band, kelompok Rp10–20 juta per bulan memiliki conversion tertinggi sekitar **23,94%** dan E2E sekitar **14,03%**. Kelompok Rp20–50 juta tidak otomatis memiliki approval tertinggi, yang konsisten dengan kemungkinan adanya requested amount yang lebih besar dan karakter SME/high-ticket.

---

# 7. Dashboard

## Dashboard 1 — Banking Performance
<img width="1327" height="738" alt="image" src="https://github.com/user-attachments/assets/4c7d540b-1fab-4e8f-a2b0-b955cd931df8" />

Halaman ini tepat sebagai executive overview karena menyajikan KPI utama, monthly trend, source, product, dan AO performance. Namun terdapat tiga penyempurnaan penting:

- Total Ad Spend sebaiknya diformat **Rp3,50B**, bukan hanya `3bn`, karena pembulatan saat ini menyembunyikan sekitar Rp495 juta.
- Urutan bulan harus menggunakan **Jul-2025 → Jun-2026**, bukan January → December. Dataset melewati dua tahun kalender sehingga Month Name tanpa Year dapat menghasilkan chronological order yang salah.
- Tambahkan KPI **End-to-End Conversion, Approved-to-Disbursement, Average Ticket, dan Approved Not Disbursed** agar halaman eksekutif menunjukkan kualitas funnel, bukan hanya stage metrics.

## Dashboard 2 — Funnel & Conversion

<img width="1318" height="738" alt="image" src="https://github.com/user-attachments/assets/af90549f-e74b-4dfe-a1c9-9f92b37692f0" />


Halaman ini sudah menjawab status prospek, conversion by source, approval by product/segment, serta application decision. Kekurangannya adalah funnel belum divisualisasikan sebagai satu alur eksplisit dan Source Funnel Scorecard masih mengalami repeated grand total untuk downstream measure.

Prioritas perbaikan adalah menambahkan:

- funnel visual Prospect → Application → Approved → Disbursed,
- Drop-off Count dan Drop-off Rate per stage,
- E2E conversion,
- Approved Not Disbursed dan aging 7/14/30+ hari,
- Reason Code untuk dropped, rejected, cancelled, dan non-disbursement apabila tersedia.

## Dashboard 3 — Marketing Efficiency

<img width="1323" height="730" alt="image" src="https://github.com/user-attachments/assets/b672f253-b53a-4685-a9eb-9011607a1941" />


Halaman ini memiliki potensi business value terbesar karena menghubungkan spend dengan acquisition quality. Tetapi `Applications by Source`, `Disbursement by Source`, dan scorecard saat ini belum valid secara filter context. Setelah model diperbaiki, halaman ini sebaiknya berfokus pada:

- Spend Share vs Disbursement Share,
- Cost per Prospect,
- Cost per Application,
- Cost per Disbursement,
- E2E Conversion by Source,
- Disbursement-to-Spend Ratio,
- Source Quality Matrix.

Dengan struktur tersebut, perbedaan antara source volume tinggi dan source berkualitas tinggi akan terlihat secara langsung.

## Dashboard 4 — AO & Branch Performance

<img width="1310" height="727" alt="image" src="https://github.com/user-attachments/assets/01dc3d83-3f56-491f-bdee-c44e1fbd4107" />


Halaman ini paling siap digunakan untuk performance review karena AO dan Branch merupakan shared dimensions. Visual sudah menunjukkan disbursement, conversion, approval, dan scorecard.

Pengembangan berikutnya:

- AO quadrant: Conversion vs Disbursement dengan bubble = applications.
- Rank dan percentile AO.
- Peer benchmark dalam branch yang sama.
- Conditional formatting untuk E2E, approval, dan processing time.
- Branch scorecard dengan Cost per Disbursement dan E2E.

## Dashboard 5 — Product & Customer Insights

<img width="1313" height="731" alt="image" src="https://github.com/user-attachments/assets/411de230-a0c4-4655-b5ea-86aeff83eeb9" />


Halaman ini sudah menunjukkan product mix, conversion, approval, disbursement, segment, dan occupation. Namun Product Performance Scorecard juga mengalami repeated downstream totals karena field Product pada `fact_prospects` belum menjadi shared dimension terhadap application/disbursement.

Setelah model diperbaiki, halaman ini sebaiknya menampilkan:

- Product Funnel,
- Average Ticket,
- Weighted Rate,
- Weighted Tenor,
- Funding Fulfillment,
- Rejection Rate,
- Processing Time,
- Customer income/occupation profile,
- kontribusi nominal vs funnel efficiency.

---

# 8. Insights

### Insight 1 — Funnel masih kehilangan nilai terbesar sebelum application
Hanya **21,74%** prospect menjadi application. Artinya sekitar 78 dari 100 lead berhenti sebelum tahap pengajuan. Fokus improvement terbesar bukan sekadar approval, tetapi **prospect qualification dan follow-up discipline**.

### Insight 2 — Approved belum berarti realized
Sebanyak **1.018 approved application** belum memiliki disbursement tercatat; 909 di antaranya lebih dari 30 hari dan mewakili requested amount sekitar **Rp134,325 miliar**. Ini merupakan leakage yang langsung dekat dengan revenue realization dan layak menjadi priority queue.

### Insight 3 — Facebook Ads over-index pada spend tetapi under-index pada outcome
Facebook menyerap **41,8% marketing spend**, tetapi hanya memberikan sekitar **9,5% nominal disbursement** dan memiliki E2E hanya **5,61%**. Ini menunjukkan bahwa optimasi berbasis jumlah lead berpotensi mengarahkan budget ke source yang kurang berkualitas.

### Insight 4 — Referral, Walk-in, dan Event memiliki acquisition quality superior
Referral mempunyai E2E **24,91%**, Event **22,47%**, dan Walk-in **19,50%**, jauh di atas Facebook dan Instagram. Kualitas ini harus dimanfaatkan, tetapi strategi scaling perlu mempertimbangkan kapasitas channel dan tidak boleh diasumsikan linear terhadap tambahan budget.

### Insight 5 — Branch gap bersifat material
Samarinda Pusat mencatat E2E **15,00%**, sedangkan Kutai Barat **10,42%**. Selisih ini terlalu besar untuk diabaikan dan perlu dibedah berdasarkan source mix, product mix, dan aktivitas AO.

### Insight 6 — Kinerja AO perlu dipisahkan antara faktor individu dan faktor pasar
AO dengan hasil rendah terkonsentrasi pada cabang yang juga lemah, sehingga performance management harus menggunakan peer benchmark. Dimas Fadillah menjadi contoh kandidat evaluasi individual yang lebih kuat karena berada di cabang benchmark tetapi E2E-nya masih jauh di bawah AO terbaik pada cabang yang sama.

### Insight 7 — SME adalah value driver, Payroll adalah efficiency driver
SME menghasilkan **44,89% nominal disbursement** karena ticket size besar, sedangkan Payroll memiliki funnel terbaik. Strategi bisnis harus memperlakukan keduanya secara berbeda: SME untuk value, Payroll untuk scalable conversion.

### Insight 8 — Dashboard saat ini belum sepenuhnya decision-safe untuk analisis Source/Product downstream
Repeated totals pada source/product scorecard adalah masalah model, bukan sekadar kosmetik. Apabila tidak diperbaiki, manajemen dapat membaca angka yang tampak presisi tetapi secara filter context salah.

### Insight 9 — Tampilan Month Name berpotensi menghasilkan kesimpulan trend yang salah
Karena data mencakup Juli 2025–Juni 2026, urutan January–December bukan urutan kronologis periodenya. Trend harus menggunakan Year-Month agar tidak terjadi false sequence.

### Insight 10 — Dataset belum dapat mengukur quality after disbursement
Tidak ada DPD, outstanding, kolektibilitas, NPL, default, repayment, atau profitability. Karena itu, channel atau produk dengan pencairan tinggi belum tentu paling sehat secara portofolio. Tahap berikutnya adalah mengintegrasikan loan performance data.

---

# 9. Recommendations

| Priority | Recommendation | PIC Utama | KPI Keberhasilan | Monitoring | Risiko/Mitigasi |
|---|---|---|---|---|---|
| **P0** | Perbaiki semantic model dengan shared `DimSource`, `DimProduct`, `DimSegment`, relationship one-to-many, dan Year-Month | Data/BI | Tidak ada repeated grand total; seluruh slicer lolos filter propagation test | Validation matrix per page sebelum publish | Risiko perubahan measure; gunakan regression test angka terhadap CSV |
| **P0** | Tambahkan funnel E2E dan Approved Not Disbursed Aging | Credit Ops + Marketing | E2E rate, approved→disb rate, jumlah/value aging >7/14/30 hari | Weekly funnel review | Pending akhir periode bisa belum mature; gunakan maturity cutoff |
| **P1** | Ubah marketing evaluation dari cost-per-lead menjadi quality-adjusted cost | Head Marketing | Cost/Disb, E2E by source, Disb/Spend | Monthly channel review | Referral/walk-in tidak scalable linear; gunakan controlled pilot |
| **P1** | Buat recovery workflow untuk Approved Not Disbursed | Branch Head + AO + Credit Ops | Penurunan aging >30 hari dan peningkatan approved→disb | Weekly case list dengan reason code | Risiko mengejar case yang sudah tidak feasible; klasifikasikan lost vs pending |
| **P1** | Benchmark AO berdasarkan peer branch + product/source mix | Head Marketing + Branch Head + HRD | E2E, approval, disbursement, gap-to-peer | Weekly/monthly performance review | Hindari ranking tunggal yang bias volume |
| **P1** | Diferensiasi strategi produk: scale Payroll, jaga quality SME, evaluasi economics Mikro | Business/Product + Credit | Growth, approval, E2E, ticket size per product | Monthly product committee | Belum ada NPL/profitability; keputusan final menunggu portfolio data |
| **P2** | Tambahkan reason code untuk dropped, rejected, cancelled, dan approved-not-disbursed | Operations + IT | ≥95% case memiliki reason code | Data quality dashboard | Beban input manual; gunakan daftar reason standar |
| **P2** | Integrasikan data loan performance pasca pencairan | Risk + IT/Data | DPD, PAR, NPL, outstanding, yield, margin by source/product/AO | Monthly portfolio review | Kompleksitas data; mulai dari snapshot bulanan |
| **P2** | Tambahkan target dan variance vs target pada KPI | Management + HRD + BI | Actual vs Target, attainment %, forecast | Weekly/Monthly | Target tidak comparable antar branch; gunakan target per kapasitas |

### Urutan implementasi yang disarankan

**Minggu 1–2:** model correction + KPI validation.  
**Minggu 2–3:** funnel E2E, aging, source efficiency.  
**Minggu 3–4:** AO/branch benchmarking dan management review template.  
**Bulan 2+:** portfolio quality integration dan profitability layer.

---

# 10. Business Impact

## 10.1 Decision accuracy

Perbaikan semantic model adalah dampak pertama dan paling fundamental. Saat ini beberapa visual Source/Product memberikan repeated grand total. Setelah diperbaiki, dashboard akan berfungsi sebagai **single source of truth** untuk analisis funnel, bukan sekadar visual presentation.

## 10.2 Recovery of near-realization pipeline

Terdapat **909 approved case >30 hari** tanpa disbursement tercatat, dengan requested amount sekitar **Rp134,325 miliar**. Angka ini bukan otomatis loss, tetapi merupakan value-at-risk yang dapat diprioritaskan.

Sebagai **skenario ilustratif, bukan forecast**, jika 10%–20% dari nominal aged pipeline tersebut dapat direalisasikan dengan funding fulfillment sekitar 96%, potensi tambahan pencairan berada di kisaran **Rp12,9–25,8 miliar**. Realisasi aktual tetap tergantung reason code, eligibility, customer intent, dan maturity data.

## 10.3 Conversion uplift

Jika Prospect→Application Conversion meningkat dari **21,74% menjadi 23,00%** dengan volume 50.000 prospect tetap, kualitas approval dan approved-to-disbursement tetap, maka secara matematis dapat menghasilkan sekitar:

- **+628 applications**
- **+376 disbursements**
- sekitar **Rp49,7 miliar** tambahan disbursement pada average ticket saat ini

Ini juga merupakan scenario-based estimate, bukan proyeksi, karena peningkatan volume dapat mengubah mix dan quality.

## 10.4 Marketing efficiency

Pergantian orientasi dari lead volume menjadi downstream quality dapat menurunkan cost per disbursement dan mengurangi ketergantungan pada channel mahal. Potensi terbesar berasal dari pengurangan overspend pada source yang memiliki approval/E2E rendah, lalu menggunakan pilot reallocation ke channel yang lebih berkualitas tanpa mengasumsikan skalabilitas linear.

## 10.5 Performance management

AO dan branch dapat dievaluasi berdasarkan outcome yang lebih adil: conversion, approval, E2E, disbursement, dan peer benchmark. Ini mengurangi risiko keputusan performance hanya berdasarkan jumlah prospect atau nominal pencairan tanpa mempertimbangkan kualitas proses.

## 10.6 Strategic product management

Dashboard memperjelas perbedaan fungsi produk: **SME sebagai value driver, Payroll sebagai efficiency driver, Konsumtif sebagai volume/value balance, dan Mikro sebagai high-rate low-ticket business**. Dengan tambahan data portfolio quality, strategi produk dapat berkembang dari growth dashboard menjadi profitability-and-risk dashboard.

---

# Kesimpulan

Secara keseluruhan, dataset memperlihatkan bahwa persoalan utama bukan kekurangan data, melainkan bagaimana mengubah data menjadi **alur keputusan yang konsisten dari acquisition sampai realization**. Funnel 50.000 prospect menghasilkan 6.510 pencairan dengan nilai Rp860,905 miliar; tetapi kualitas hasil sangat berbeda antar source, branch, AO, dan product.

Tiga prioritas terbesar adalah:

1. **Perbaiki semantic model** agar Source/Product/Segment benar-benar memfilter seluruh funnel.
2. **Kelola leakage**, terutama prospect yang tidak naik ke application dan approved case yang tidak menjadi disbursement.
3. **Optimalkan resource allocation** berdasarkan quality-adjusted outcomes, bukan volume lead semata.

Jika tiga hal tersebut dilakukan, dashboard akan berubah dari descriptive reporting menjadi **management decision system** yang mampu menjelaskan apa yang terjadi, mengapa terjadi, di mana masalahnya, dan tindakan apa yang paling layak diprioritaskan.
