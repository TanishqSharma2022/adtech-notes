
[Geo Insights - Phase 1](https://mediaiq.atlassian.net/wiki/spaces/MIP/pages/4895408145/Geo+Insights+-+Phase+1)

Major Datasets 

Experian - US - [master_catalog.geolocation_datasets.experian_uk_scv](master_catalog.geolocation_datasets.experian_uk_scv)
Experian/Cohorts - UK - [s3://miq-local-products/EMEA/Cohorts/Cohorts_Custom_Master_Data/](s3://miq-local-products/EMEA/Cohorts/Cohorts_Custom_Master_Data/) includes postalcode, segment name, feature, value, normalized index.
Experian/Cohorts - AU - [s3://miq-local-products/ANZ/Cohorts/cohorts_master_data/](s3://miq-local-products/ANZ/Cohorts/cohorts_master_data/) - only includes segment_name, segment share, segment_total, postcode population, total population and normalized index.

Environics - CA

Lifesight - (SEA,India)


Gives DV360 report metrics for all regions (US, CA, AU, GB) - [[zip_code_metrics](s3://miq-prod-precampaign/feature_store/dv360/zip_code_metrics/)](s3://miq-prod-precampaign/feature_store/dv360/zip_code_metrics/)
issues is many of the zipcode can be null.


Brand to IAB Mapping - 

[Brands Normalization Mapping](https://mediaiq.atlassian.net/wiki/spaces/MIP/pages/4716265477/Brands+Normalization+Mapping)
[Brands Tables - 2](https://mediaiq.atlassian.net/wiki/spaces/MIP/pages/4229038094/Brand+Tables+-+2)


MiQ AU Cohorts Taxonomy Experian and Census - [https://docs.google.com/spreadsheets/d/14FqctvKt_8Q2ql8NsOUR0mcQo06pEKwY/edit?gid=1792430159#gid=1792430159](https://docs.google.com/spreadsheets/d/14FqctvKt_8Q2ql8NsOUR0mcQo06pEKwY/edit?gid=1792430159#gid=1792430159)

[Affinity Score Calculation](https://mediaiq.atlassian.net/wiki/spaces/MIP/pages/4562550788/Affinity+Score)


Brand Enrichment done by someone - [s3://miq-context-intelligence/adhoc/tbiegus/brands_enrichment/v4/internal_brand_names/missign_enrichments_filled/full_enriched_df.csv](s3://miq-context-intelligence/adhoc/tbiegus/brands_enrichment/v4/internal_brand_names/missign_enrichments_filled/full_enriched_df.csv)


### Intelligence Brands - IAB Tier list
[https://docs.google.com/spreadsheets/d/1HBWds2PPikabyFNiQI724oW2RJoJ26EhHIHNbc-SZCk/edit?gid=1591017368#gid=1591017368](https://docs.google.com/spreadsheets/d/1HBWds2PPikabyFNiQI724oW2RJoJ26EhHIHNbc-SZCk/edit?gid=1591017368#gid=1591017368)
This spreadsheet appears to be a mapping of advertising content categories, likely following the Interactive Advertising Bureau (IAB) structure, and is divided into two parts:

**Main IAB Tier Mapping (A1:D236):**
This section lists categorized content using three tiers, likely for targeting or brand safety:

- **Tier 1:** The broadest category (e.g., Attractions, Automotive, Sports).
- **Tier 2 (This should be considered for Verticals):** A more specific sub-category under Tier 1 (e.g., Amusement and Theme Parks, Auto Buying and Selling, Baseball).
- **Tier 3 (Required?):** Indicates whether a third tier (further breakdown) is necessary for that category (Yes/No).
- **Custom mapping:** Shows the hierarchical mapping of the categories (e.g., Attractions > Amusement and Theme Parks).

**Excluded Tier 1 Categories (G6:H20):**
This secondary table lists IAB Tier 1 categories that have been excluded or deemed "Not valid for brand," along with the **Reason** for their exclusion (e.g., "Not appropriate for brand," or "Related to personal life events").




## Cosine Similarity vs Cross Encoder Similarity

1. Get both features into a feature space as a vector and find cosine similarity between them
2. feed `feature 1 | seperator | feature 2` into a sentence transformer trained on computing similarity between two texts. Is a bit computationally expensive but gives score between 0 to 1 and has more accuracy.
