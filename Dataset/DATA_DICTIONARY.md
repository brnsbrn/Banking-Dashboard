# Data Dictionary

## dim_branch
branch_id, branch_name, city, region

## dim_ao
ao_id, ao_name, branch_id, branch_name, status

## fact_campaign_spend
campaign_id, campaign_date, month, branch_id, source, campaign_type, campaign_name, ad_spend

## fact_prospects
prospect_id, prospect_date, ao_id, branch_id, source, segment, occupation, city,
age, monthly_income, requested_amount, tenor_months, product, prospect_status

## fact_applications
application_id, prospect_id, application_date, decision_date, ao_id, branch_id,
product, segment, requested_amount, monthly_income, tenor_months,
application_status, processing_days

## fact_disbursements
disbursement_id, application_id, prospect_id, disbursement_date, branch_id, ao_id,
product, segment, disbursed_amount, interest_rate_pa, tenor_months
