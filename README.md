Laporan Analisis Business Intelligence
Banking Performance, Credit Funnel, Marketing Efficiency, AO & Branch Performance, serta Product & Customer Insights

---
1. Business Problem
Masalah utama bukan hanya bagaimana mendapatkan lebih banyak prospek, tetapi bagaimana memastikan prospek tersebut benar-benar berkembang menjadi pengajuan, disetujui, dan akhirnya dicairkan dengan biaya pemasaran yang efisien.
Dari data yang tersedia, ada beberapa pertanyaan utama yang perlu dijawab:
Seberapa efektif 50.000 prospek berubah menjadi aplikasi, persetujuan, dan pencairan?
Sumber prospek mana yang hanya menghasilkan volume tinggi, dan mana yang benar-benar menghasilkan kualitas terbaik?
AO dan cabang mana yang produktif jika dilihat dari hasil akhir, bukan hanya jumlah prospek?
Produk mana yang memberi nilai pencairan terbesar dan produk mana yang paling efisien dalam proses konversi?
Apakah biaya pemasaran sebesar Rp3,495 miliar sudah dialokasikan ke sumber yang memberikan hasil terbaik?
Selain itu, ditemukan masalah pada beberapa visual Power BI. Beberapa nilai seperti Total Applications, Approved Applications, dan Total Disbursement muncul sama pada setiap Source atau Product. Ini menunjukkan bahwa hubungan antar tabel pada model data belum sepenuhnya tepat. Sebelum dashboard digunakan sebagai dasar keputusan, bagian ini perlu diperbaiki.
---
2. Business Understanding
Tujuan bisnis yang lebih tepat adalah menghasilkan pencairan yang berkualitas dan efisien, bukan sekadar memperbanyak lead.
Alur bisnis dibaca sebagai berikut:
Prospect → Application → Approval → Disbursement → Portfolio Performance
Dataset saat ini baru dapat dianalisis sampai tahap pencairan. Data setelah pencairan seperti outstanding, DPD, kolektibilitas, NPL, pembayaran, write-off, cost of fund, dan laba bersih belum tersedia. Karena itu, analisis ini dapat menilai kualitas akuisisi dan proses kredit, tetapi belum dapat menilai kualitas portofolio maupun profitabilitas kredit secara penuh.
Analisis dibagi ke dalam enam sisi:
Akuisisi: jumlah prospek, source, segment, product, branch, dan AO.
Konversi: prospek yang berhasil menjadi aplikasi.
Keputusan kredit: aplikasi yang disetujui, ditolak, atau dibatalkan.
Realisasi: aplikasi yang disetujui dan akhirnya dicairkan.
Efisiensi: biaya untuk menghasilkan prospek, aplikasi, dan pencairan.
Produktivitas: hasil AO dan cabang berdasarkan kualitas funnel, bukan hanya aktivitas.
Dengan pendekatan ini, dashboard dapat membantu menjawab bukan hanya “berapa hasilnya?”, tetapi juga “di mana masalahnya?” dan “apa yang harus dilakukan?”.
---
3. Data Understanding
Dataset terdiri dari enam tabel utama:
Tabel	Fungsi
`dim_branch`	Data cabang
`dim_ao`	Data Account Officer
`fact_campaign_spend`	Biaya dan aktivitas pemasaran
`fact_prospects`	Data prospek
`fact_applications`	Data pengajuan kredit
`fact_disbursements`	Data pencairan kredit
Skala data
50.000 prospek
10.872 aplikasi
7.528 aplikasi disetujui
6.510 pencairan
Total pencairan Rp860,905 miliar
528 catatan kampanye
Marketing spend Rp3,495 miliar
7 cabang
28 Account Officer
Pemeriksaan data menunjukkan bahwa struktur datanya cukup bersih. Tidak ditemukan missing value utama, primary key ganda, atau hubungan ID yang terputus antara prospek, aplikasi, pencairan, AO, dan cabang.
Periode utama data adalah Juli 2025 sampai Juni 2026. Sebagian kecil pencairan masuk sampai 1 Juli 2026 karena adanya jeda waktu dari proses persetujuan akhir Juni.
---
4. Data Preparation
Beberapa tahap persiapan data dilakukan agar analisis konsisten:
Memastikan `prospect_id`, `application_id`, `ao_id`, dan `branch_id` saling terhubung dengan benar.
Mengubah seluruh kolom tanggal menjadi tipe Date.
Menyamakan penulisan kategori Source, Product, Segment, Status, Occupation, Branch, dan AO.
Menghubungkan Source dari tabel prospek ke application dan disbursement melalui `prospect_id`.
Membuat metrik turunan seperti:
Conversion Rate
Approval Rate
Approved-to-Disbursement Rate
End-to-End Conversion
Average Ticket Size
Cost per Prospect
Cost per Application
Cost per Disbursement
Disbursement-to-Spend Ratio
Weighted Interest Rate
Weighted Tenor
Funding Fulfillment
Aging untuk Approved Not Disbursed
Membuat kolom Year-Month agar tren waktu mengikuti urutan Juli 2025 sampai Juni 2026.
---
5. Data Modeling
Model Power BI saat ini sudah memiliki tabel cabang, AO, tanggal, prospek, aplikasi, pencairan, dan campaign spend.
Namun, Source, Product, dan Segment belum sepenuhnya bekerja sebagai dimensi bersama. Akibatnya, beberapa measure dari tabel aplikasi atau pencairan tidak ikut terfilter ketika Source atau Product dipilih.
Contohnya, Total Applications sebesar 10.872 dan Total Disbursement sebesar Rp860,905 miliar muncul berulang pada beberapa Source atau Product.
Model yang lebih tepat
Gunakan star schema dengan dimensi bersama:
`DimDate`
`DimBranch`
`DimAO`
`DimSource`
`DimProduct`
`DimSegment`
Sedangkan tabel fakta tetap:
`FactProspect`
`FactApplication`
`FactDisbursement`
`FactCampaignSpend`
Hubungan ideal adalah one-to-many dari dimension ke fact dengan arah filter satu arah. Dengan model ini, Source, Product, Segment, AO, Branch, dan Date dapat memfilter seluruh funnel secara konsisten.
---
6. Analysis
6.1 Overall Funnel
Tahap	Jumlah	Tingkat dari tahap sebelumnya
Prospect	50.000	-
Application	10.872	21,74%
Approved	7.528	69,24%
Disbursed	6.510	86,48% dari approved
End-to-End Conversion dari Prospect sampai Disbursement adalah 13,02%.
Artinya, dari setiap 100 prospek, sekitar 13 yang akhirnya menjadi pencairan.
Kehilangan terbesar terjadi sebelum tahap aplikasi. Sebanyak 39.128 prospek tidak berkembang menjadi aplikasi. Selain itu, terdapat 3.344 aplikasi yang tidak disetujui dan 1.018 aplikasi yang sudah disetujui tetapi belum tercatat sebagai pencairan.
Prospect Status juga menunjukkan 9.869 Follow-up Pending. Angka ini besar dan perlu dibedakan antara prospek yang masih aktif, terlambat ditindaklanjuti, atau sudah tidak potensial.
Dari 1.018 approved yang belum dicairkan, 909 kasus sudah lebih dari 30 hari, dengan requested amount sekitar Rp134,325 miliar. Ini merupakan salah satu area yang paling layak diprioritaskan karena proses kreditnya sudah hampir selesai tetapi belum menghasilkan pencairan.
Untuk kredit yang berhasil cair, rata-rata waktu proses adalah sekitar:
Prospect → Application: 1,95 hari
Application → Decision: 3,27 hari
Decision → Disbursement: 2,97 hari
Prospect → Disbursement: 8,20 hari
Dengan demikian, masalah terbesar bukan terlihat pada lamanya proses, tetapi pada besarnya jumlah prospek yang berhenti di tengah funnel.
6.2 Marketing Efficiency
Total marketing spend adalah Rp3,495 miliar.
Secara keseluruhan:
Cost per Prospect: sekitar Rp69,9 ribu
Cost per Application: sekitar Rp321,5 ribu
Cost per Disbursement: sekitar Rp536,9 ribu
Disbursement-to-Spend Ratio: sekitar 246,3 kali
Namun, kualitas tiap sumber sangat berbeda.
Source	Conversion	Approval	E2E	Cost/Disb.
Referral	30,94%	82,21%	24,91%	Rp94,7 ribu
Event	28,51%	81,92%	22,47%	Rp215,5 ribu
Walk-in	26,66%	77,62%	19,50%	Rp64,2 ribu
WhatsApp	21,71%	70,50%	12,79%	Rp544,0 ribu
Telemarketing	18,72%	60,91%	8,61%	Rp604,0 ribu
Instagram Ads	16,63%	55,67%	6,53%	Rp1,245 juta
Facebook Ads	16,08%	51,20%	5,61%	Rp2,451 juta
Facebook Ads
Facebook menghasilkan prospek terbanyak, tetapi kualitas akhirnya paling rendah. Channel ini menggunakan sekitar 41,8% total marketing spend, namun hanya menghasilkan sekitar 9,5% total nominal pencairan.
Artinya, Facebook bagus dalam menghasilkan volume, tetapi belum efisien dalam menghasilkan pencairan.
Referral dan Walk-in
Referral menggunakan porsi biaya jauh lebih kecil, tetapi menghasilkan conversion dan approval yang tinggi. Walk-in juga sangat efisien dari sisi biaya per pencairan.
Namun, channel seperti Referral dan Walk-in tidak selalu mudah ditingkatkan skalanya seperti iklan digital. Karena itu, strategi yang lebih tepat bukan menghentikan digital, tetapi memperbaiki kualitas targeting, follow-up, dan alokasi anggarannya.
6.3 Branch Performance
Samarinda Pusat menjadi cabang dengan kinerja terbaik secara keseluruhan. Cabang ini memiliki conversion 23,66%, approval 72,65%, E2E 15,00%, dan total pencairan Rp188,109 miliar.
Kutai Barat berada di posisi terendah dengan conversion 19,82%, approval 63,44%, E2E 10,42%, serta total pencairan Rp75,628 miliar.
Perbedaan ini cukup besar, tetapi tidak boleh langsung dianggap sebagai masalah kinerja orang. Perlu dilihat juga product mix, source mix, kondisi pasar, dan profil calon nasabah di masing-masing wilayah.
6.4 AO Performance
Fajar Nugroho menjadi salah satu AO dengan hasil terbaik:
2.257 prospek
604 aplikasi
Conversion 26,76%
Approval 75,83%
418 pencairan
E2E 18,52%
Total pencairan Rp60,210 miliar
Sebaliknya, beberapa AO dengan hasil lebih rendah berada di cabang yang juga memiliki performa rendah. Ini berarti evaluasi AO tidak cukup hanya menggunakan ranking nasional. AO sebaiknya dibandingkan dengan rekan yang bekerja dalam cabang dan kondisi pasar yang serupa.
6.5 Product Performance
Produk	Conversion	Approval	E2E	Total Disbursement	Avg Ticket
Kredit Produktif SME	20,49%	66,17%	11,28%	Rp386,426 miliar	Rp374,4 juta
Kredit Payroll	26,87%	77,52%	17,88%	Rp214,335 miliar	Rp112,3 juta
Kredit Konsumtif	21,76%	68,51%	13,17%	Rp190,220 miliar	Rp86,2 juta
Kredit Mikro	18,50%	63,07%	10,15%	Rp69,925 miliar	Rp51,3 juta
SME
SME adalah penggerak nilai. Jumlah prospeknya tidak terbesar, tetapi menyumbang sekitar 44,89% total nominal pencairan karena rata-rata ticket sangat besar.
Payroll
Payroll adalah penggerak efisiensi. Produk ini memiliki conversion, approval, dan E2E tertinggi. Jika kualitas kredit setelah pencairan juga baik, Payroll layak menjadi produk yang dikembangkan lebih agresif.
Mikro
Mikro memiliki funnel paling lemah dan ticket size paling kecil. Walaupun tingkat bunga lebih tinggi, belum dapat disimpulkan apakah produk ini lebih menguntungkan karena belum ada data NPL, biaya penagihan, dan cost of fund.
6.6 Customer Profile
Wiraswasta merupakan kelompok pekerjaan terbesar dengan sekitar 24,88% dari seluruh prospek.
Namun, volume tidak selalu berarti kualitas tertinggi. Karyawan BUMN dan Dokter/Tenaga Medis memiliki E2E yang lebih tinggi dibanding Wiraswasta. Sementara itu, Tenaga Kontrak memiliki E2E yang lebih rendah.
Informasi ini sebaiknya digunakan untuk memahami pola pasar dan kebutuhan produk, bukan sebagai dasar otomatis untuk menerima atau menolak calon nasabah.
---
7. Dashboard
Dashboard 1 — Banking Performance
<img width="1327" height="738" alt="Banking Performance Dashboard" src="https://github.com/user-attachments/assets/4c7d540b-1fab-4e8f-a2b0-b955cd931df8" />
Halaman ini tepat digunakan sebagai ringkasan eksekutif karena menyajikan KPI utama, tren bulanan, sumber prospek, produk, dan performa AO dalam satu tampilan. Manajemen dapat melihat kondisi bisnis secara cepat sebelum masuk ke analisis yang lebih rinci pada halaman berikutnya.
Namun, ada beberapa penyempurnaan penting:
Total Ad Spend sebaiknya ditampilkan sebagai Rp3,50 miliar, bukan hanya `3bn`. Pembulatan menjadi `3bn` menyembunyikan selisih sekitar Rp495 juta dan membuat angka kurang presisi untuk kebutuhan manajemen.
Urutan bulan sebaiknya menggunakan Jul-2025 → Jun-2026, bukan January → December. Dataset melewati dua tahun kalender sehingga Month Name tanpa Year dapat menghasilkan urutan waktu yang kurang tepat.
Tambahkan KPI End-to-End Conversion, Approved-to-Disbursement Rate, Average Ticket, dan Approved Not Disbursed agar halaman eksekutif tidak hanya menunjukkan jumlah pada setiap tahap, tetapi juga kualitas funnel secara keseluruhan.
Dengan perbaikan tersebut, halaman pertama akan lebih efektif sebagai tampilan utama untuk membaca kondisi bisnis secara cepat.
Dashboard 2 — Funnel & Conversion
<img width="1318" height="738" alt="Funnel and Conversion Dashboard" src="https://github.com/user-attachments/assets/af90549f-e74b-4dfe-a1c9-9f92b37692f0" />
Halaman ini sudah menjawab beberapa pertanyaan penting, seperti status prospek, conversion berdasarkan source, approval berdasarkan product dan segment, serta hasil keputusan aplikasi.
Kekurangannya adalah keseluruhan funnel belum terlihat sebagai satu alur yang utuh. Pengguna masih perlu membaca beberapa visual secara terpisah untuk memahami perjalanan dari prospek sampai pencairan. Selain itu, Source Funnel Scorecard masih menunjukkan nilai total yang berulang pada beberapa metrik downstream karena filter context antar tabel belum bekerja dengan benar.
Prioritas pengembangannya adalah:
menambahkan visual funnel Prospect → Application → Approved → Disbursed;
menampilkan Drop-off Count dan Drop-off Rate pada setiap tahap;
menambahkan End-to-End Conversion;
menampilkan Approved Not Disbursed beserta aging 7/14/30+ hari;
menambahkan Reason Code untuk dropped, rejected, cancelled, dan non-disbursement apabila data tersebut tersedia.
Dengan tambahan ini, dashboard akan lebih mudah digunakan untuk mengetahui di tahap mana prospek paling banyak berhenti dan bagian mana yang perlu segera ditindaklanjuti.
Dashboard 3 — Marketing Efficiency
<img width="1323" height="730" alt="Marketing Efficiency Dashboard" src="https://github.com/user-attachments/assets/b672f253-b53a-4685-a9eb-9011607a1941" />
Halaman ini memiliki potensi nilai bisnis yang sangat besar karena menghubungkan biaya pemasaran dengan kualitas hasil akuisisi. Dengan halaman ini, marketing tidak hanya dinilai dari jumlah prospek yang diperoleh, tetapi juga dari kemampuan setiap source menghasilkan aplikasi, approval, dan pencairan.
Saat ini, visual `Applications by Source`, `Disbursement by Source`, dan beberapa bagian scorecard belum sepenuhnya valid karena masalah filter context. Setelah model data diperbaiki, halaman ini sebaiknya berfokus pada:
Spend Share vs Disbursement Share;
Cost per Prospect;
Cost per Application;
Cost per Disbursement;
E2E Conversion by Source;
Disbursement-to-Spend Ratio;
Source Quality Matrix.
Dengan struktur tersebut, perbedaan antara source yang menghasilkan volume tinggi dan source yang benar-benar menghasilkan kualitas bisnis tinggi akan terlihat lebih jelas. Hal ini akan membantu manajemen menentukan apakah anggaran pemasaran perlu dipertahankan, dikurangi, atau dialihkan ke channel lain.
Dashboard 4 — AO & Branch Performance
<img width="1310" height="727" alt="AO and Branch Performance Dashboard" src="https://github.com/user-attachments/assets/01dc3d83-3f56-491f-bdee-c44e1fbd4107" />
Halaman ini paling siap digunakan untuk evaluasi performa AO dan cabang karena AO dan Branch sudah berfungsi sebagai shared dimensions. Visual yang ada telah menunjukkan pencairan, conversion, approval, serta scorecard performa.
Pengembangan berikutnya yang disarankan:
membuat AO Quadrant dengan Conversion sebagai sumbu X, Disbursement sebagai sumbu Y, dan jumlah Applications sebagai ukuran bubble;
menambahkan ranking dan percentile AO;
menggunakan peer benchmark dalam cabang yang sama agar penilaian lebih adil;
menambahkan conditional formatting untuk E2E, Approval Rate, dan Processing Time;
membuat Branch Scorecard yang juga menampilkan Cost per Disbursement dan E2E.
Tujuannya adalah agar evaluasi AO tidak hanya melihat siapa yang memiliki pencairan terbesar, tetapi juga siapa yang paling efektif mengubah prospek menjadi hasil bisnis. Untuk cabang, analisis juga dapat menunjukkan apakah masalah lebih banyak berasal dari kualitas pasar, source mix, product mix, atau produktivitas tim.
Dashboard 5 — Product & Customer Insights
<img width="1313" height="731" alt="Product and Customer Insights Dashboard" src="https://github.com/user-attachments/assets/411de230-a0c4-4655-b5ea-86aeff83eeb9" />
Halaman ini sudah memberikan gambaran yang baik mengenai product mix, conversion, approval, disbursement, segment, dan occupation. Halaman ini membantu memahami bahwa setiap produk mempunyai karakter bisnis yang berbeda dan tidak seharusnya dinilai hanya berdasarkan volume prospek atau nominal pencairan.
Namun, Product Performance Scorecard masih menunjukkan beberapa nilai downstream yang berulang karena field Product pada `fact_prospects` belum berfungsi sebagai shared dimension terhadap tabel application dan disbursement.
Setelah model diperbaiki, halaman ini sebaiknya menampilkan:
Product Funnel;
Average Ticket;
Weighted Rate;
Weighted Tenor;
Funding Fulfillment;
Rejection Rate;
Processing Time;
Customer income dan occupation profile;
perbandingan kontribusi nominal vs funnel efficiency.
Dengan tambahan tersebut, manajemen dapat membedakan produk yang berfungsi sebagai penggerak volume, penggerak nilai, atau penggerak efisiensi, serta melihat profil calon nasabah yang paling sesuai untuk masing-masing produk.
---
8. Insights
Kebocoran terbesar ada sebelum tahap aplikasi. Hanya 21,74% prospek yang menjadi aplikasi.
Approval belum tentu menjadi pencairan. Terdapat 1.018 approved application yang belum tercatat dicairkan.
Facebook Ads menghasilkan volume tinggi tetapi kualitas akhir rendah. Porsi biaya sangat besar, tetapi kontribusi pencairannya relatif kecil.
Referral, Event, dan Walk-in memiliki kualitas funnel yang jauh lebih baik.
Ada perbedaan nyata antar cabang. Samarinda Pusat menjadi benchmark, sedangkan Kutai Barat perlu dianalisis lebih dalam.
Evaluasi AO harus mempertimbangkan kondisi cabang. Ranking tanpa konteks dapat menghasilkan kesimpulan yang kurang adil.
SME adalah penggerak nilai, sedangkan Payroll adalah penggerak efisiensi.
Sebagian visual Source dan Product belum aman digunakan untuk keputusan karena masalah filter context.
Urutan bulan pada dashboard perlu diperbaiki agar sesuai kronologi Juli 2025 sampai Juni 2026.
Analisis belum mencakup kualitas kredit setelah pencairan karena belum ada data NPL, DPD, dan pembayaran.
---
9. Recommendations
Prioritas	Rekomendasi	PIC	Indikator Keberhasilan
P0	Perbaiki model Source, Product, Segment, dan Year-Month	Data/BI	Tidak ada repeated grand total
P0	Tambahkan funnel E2E dan aging Approved Not Disbursed	Credit Ops + Marketing	E2E naik, aging >30 hari turun
P1	Ubah evaluasi marketing dari Cost per Lead menjadi Cost per Disbursement	Head Marketing	Biaya per pencairan turun
P1	Buat daftar tindak lanjut Approved Not Disbursed	Branch Head + AO + Credit Ops	Lebih banyak approval menjadi pencairan
P1	Evaluasi AO dengan peer benchmark	Head Marketing + Branch Head + HRD	Gap terhadap peer membaik
P1	Bedakan strategi Payroll, SME, Konsumtif, dan Mikro	Business/Product + Credit	KPI per produk membaik
P2	Tambahkan reason code pada dropped, rejected, cancelled, dan belum cair	Operations + IT	Mayoritas kasus memiliki alasan jelas
P2	Integrasikan data NPL, DPD, outstanding, dan repayment	Risk + IT/Data	Dashboard kualitas portofolio tersedia
P2	Tambahkan target dan actual vs target	Management + HRD + BI	Pencapaian dapat dipantau rutin
Urutan implementasi
Minggu 1–2: perbaikan model dan validasi KPI.
Minggu 2–3: funnel E2E, aging, dan efisiensi source.
Minggu 3–4: benchmark AO/cabang dan format review manajemen.
Bulan 2 dan seterusnya: integrasi kualitas portofolio dan profitabilitas.
---
10. Business Impact
10.1 Keputusan menjadi lebih akurat
Perbaikan model data akan mengurangi risiko membaca angka yang salah karena filter tidak bekerja dengan benar. Dashboard dapat menjadi sumber data yang lebih konsisten untuk evaluasi manajemen.
10.2 Potensi recovery dari approved pipeline
Terdapat 909 approved case berusia lebih dari 30 hari dengan requested amount sekitar Rp134,325 miliar.
Sebagai ilustrasi, jika 10%–20% dari nilai tersebut dapat direalisasikan dengan pola pencairan yang sama, potensi tambahan pencairan berada di kisaran Rp12,9–25,8 miliar. Angka ini adalah simulasi, bukan proyeksi pasti.
10.3 Potensi kenaikan conversion
Jika Prospect-to-Application Conversion meningkat dari 21,74% menjadi 23,00% dengan jumlah prospek tetap, secara matematis dapat menghasilkan sekitar:
628 aplikasi tambahan
376 pencairan tambahan
sekitar Rp49,7 miliar tambahan pencairan pada average ticket saat ini
Ini juga merupakan simulasi dengan asumsi kualitas approval dan pencairan tidak berubah.
10.4 Efisiensi pemasaran lebih baik
Evaluasi berdasarkan Cost per Disbursement akan membantu mengurangi pemborosan pada channel yang mahal tetapi menghasilkan kualitas rendah.
10.5 Evaluasi AO lebih adil
AO dapat dinilai berdasarkan conversion, approval, E2E, pencairan, dan perbandingan dengan rekan dalam kondisi pasar yang serupa.
10.6 Strategi produk lebih jelas
Dashboard menunjukkan bahwa SME dan Payroll memiliki peran bisnis yang berbeda. SME kuat dari sisi nilai pencairan, sedangkan Payroll kuat dari sisi efisiensi funnel. Dengan tambahan data kualitas portofolio, strategi produk dapat dibuat lebih lengkap.
---
Kesimpulan
Data menunjukkan bahwa masalah utama bukan kekurangan prospek. Dari 50.000 prospek, hanya 6.510 yang berakhir menjadi pencairan, sehingga End-to-End Conversion berada di 13,02%.
Tiga prioritas utama adalah:
Memperbaiki model data agar seluruh filter bekerja dengan benar.
Mengurangi kebocoran funnel, terutama dari Prospect ke Application dan dari Approved ke Disbursement.
Mengalokasikan sumber daya berdasarkan kualitas hasil, bukan hanya jumlah lead.
Jika tiga hal ini dilakukan, dashboard tidak hanya berfungsi sebagai laporan, tetapi dapat menjadi alat bantu keputusan yang menjelaskan apa yang terjadi, di mana masalahnya, dan tindakan apa yang perlu diprioritaskan.
