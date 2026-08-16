# PT Bank Nusa Karya Sejahtera — Large Synthetic Banking BI Dataset

This dataset is 100% synthetic and intended only for portfolio and learning purposes.

## Dataset scale
- Period: July 2025 – June 2026
- Branches: 7
- Account Officers: 28
- Prospects: 50,000
- Applications: 10,872
- Approved applications: 7,528
- Disbursements: 6,510
- Disbursement amount: Rp 860.90 B
- Campaign records: 528
- Marketing spend: Rp 3.50 B

## Files
- dim_branch.csv
- dim_ao.csv
- fact_campaign_spend.csv
- fact_prospects.csv
- fact_applications.csv
- fact_disbursements.csv

## Why this is better for a portfolio
The fact tables contain thousands to tens of thousands of records, making the dataset suitable for:
- Power BI data modeling
- Power Query transformation
- DAX measures
- SQL aggregation and joins
- Python EDA
- Funnel analysis
- AO / branch performance analysis
- Marketing efficiency analysis
- Time-series trends
- Drill-through and slicer interaction

## Business logic intentionally embedded
- Referral and WhatsApp leads have higher conversion quality.
- Payroll generally converts better.
- Large SME requests are harder to approve.
- AO productivity differs.
- Not every prospect becomes an application.
- Not every application is approved.
- Not every approved application is disbursed.
- Campaign spend varies by channel, branch and month.

## Recommended Power BI pages
1. Executive Overview
2. Marketing Funnel
3. AO & Branch Performance
4. Marketing Efficiency
5. Optional: AO Detail Drill-through
