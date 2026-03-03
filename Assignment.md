
DSP Daily Stats data 
**aggregated data** for usage in **pacing** or decision making because it is nearly **realtime**
As the DSP operates, it emits **EVENTS** like:
- Bid request
- Bid response
- Win notification
- Impression
- Click
- Conversion
#SEE_HOW_THIS_DATA_LOOKS_LIKE

uses time windows (5-minutes, etc), and aggregate functions like `SUM(IMPRESSIONS)`, etc.

Streaming is done through APACHE KAFKA, etc. These distributed, real time systems offer
**At least once delivery, not exactly once**

Deduplication happens in Daily Stats data but gets the data fast. 
Traders want to see if there budget is rougly at correct pace as it should be. Optimizer need to see which creative works better / get more clicks. Duplication will not change the overall results, just here and there +- something. 

Log Level Data 
**clean perfect data** which is used to billing and stuff but it is **slow**. 

###  Why does Duplication happens in data?
In real-time systems, let say kafka for streaming. If the page gets reload, or crashes, or request times out. The impressions maybe counted twice. Also, winning bid sent by ad exchange maybe counted twice. 
We aggregate the data in fixed time windows (say 1 minute). Let say an impression at 10:00 reaches at 10:02 due to network lag. Then, some count it twice. 
1. Retries create duplicates for impressions say due to network issues. 
2. Late Events, some impressions arrive late due to network issues. DSP daily stats might count them in wrong time bucket

“DSP Daily Stats count fast and may include duplicates or late data; LLD cleans everything and is the final source of truth, usually within 1–3% difference once settled.”

Nothing new is invented in LLD. It just:
- Removes duplicates
- Throws away bad/fraud traffic
- Puts things in the right time bucket
- Finalizes spend and conversions



## Underpacing & Overpacing Campaigns 

### Core concept: pacing

You’re comparing:

> **How much should we have spent by today?**  
> vs  
> **How much did we actually spend?**

---

### Step 1: Expected spend till today

For a campaign:

`Expected Spend = (Total Budget) × (Days elapsed / Total campaign duration)`

---

### Step 2: Actual spend

`Actual Spend = cumulative spend till today`

---

### Step 3: Pacing ratio

`Pacing Ratio = Actual Spend / Expected Spend`

### Classification (example thresholds)

|Pacing Ratio|Label|
|---|---|
|< 0.9|Underpacing|
|0.9 – 1.1|On Track|
|> 1.1|Overpacing|

### Why this matters
- **Underpacing** → campaign may not deliver full budget
- **Overpacing** → campaign may end early

Pacing Rate has two options: ASAP=spend as fast as possible and Even=smooth the spend over time.
Pacing amount is the max amount or impressions spend during the pacing period(usually a day).



## Costs associated with a campaign
| Cost Type                     | Vendor     | Source            | Deal      | Rate | Cost | Total Cost |
| ----------------------------- | ---------- | ----------------- | --------- | ---- | ---- | ---------- |
| Media                         | —          | DSP               | —         |      |      |            |
| Data                          | —          | Xandr Market      | —         |      |      |            |
| Data                          | —          | 3rd Party Segment | —         |      |      |            |
| Data                          | —          | 3rd Party Pixel   | —         |      |      |            |
| Insight Cost                  | MIQ        | MIQ               | —         |      |      |            |
| Fixed Cost                    | MIQ        | Campaign          | Fixed     |      |      |            |
| Ad-serving                    | MIQ        | Campaign          | —         |      |      |            |
| Additional Vendor (Always On) | Similarweb | Insight           | Rev_share |      |      |            |

sl
#### 5) Insight Cost - MiQ
Can include custom insight and analytics platforms that MIQ provides. 

#### 6) Fixed Cost - MiQ
Any flat fee that does not scale with impressions or spend. 

#### 7) Ad-serving - MiQ
Cost of serving ads through an ad server which is billed per 1k impressions. 

#### 8) Additional Vendor -
External Insight vendor.


## DV360

* Media cost - the cost DSP pays the exchange or MIQ pays the DSP for the impression
* Total Media cost (to use) - Media Cost + External Fees. In total this is paid by MIQ to the DSP.
* Data Fees - Third party data fee (like xandr marketplace ig not sure)
* External Vendor costs - `master_catalog.lab_datasets.lab_vendor_billable_cost`
* IO-level cost data (YouTube ROC cost) - `master_catalog.dsp_stats.dv360_roc_stats`

Got this documentation for the vendor cost - https://mediaiq.atlassian.net/wiki/spaces/MIP/pages/4789404557/Existing+-+Lab-cost-service+Vendor+Cost

US measurement we have total media spend for a campaign here. `derived_data_catalog.us_measurement.spend_by_campaign`


exclusively for xandr - `master_catalog.dsp_stats.xandr_buyer_vendor_usage_analytics`


NOTES:
`dsp_entities.dv360_insertion_order` unable to read teh table because of version issues. 
`dsp_campaign_settings_datasets.dv360_insertion_order` can get them. Not the ideal table for this lookup. also not sure if i could get all the io . But i wont be able to get the dsp from here. 
`master_catalog.lab_datasets.lab_campaign_io_association` campaign_id is bigint (not str)
`derived_data_catalog.us_measurement.spend_by_campaign` reports cost spend and campaign name. For `US Xandr Measurement.`

`derived_data_catalog.trading_support_agent.associations` has everything but just i cannot get real time data from dv360 to check.



Challenges 
Which dataset to use



### Xandr
* `media_cost_dollar_cpm`:  cost MIQ pays for 1000 impressions. so calculated as amount x 1000
* `data_costs_cpm`: 
* `segment_data_costs`: Xandr marketplace cost. This is 3rd Party Segment Cost.Unit is **microcents** (1 microcent = 1e‑8 USD).
* `feature_costs`: Cost of Xandr platform “features” used on that impression, such as cross‑device, Nielsen DAR, NCS, etc.


The **Re-arch Campaign Cost Service** design makes:

- **`daily_vendor_cost`** the **single source of truth** for all vendor costs, with columns like `campaign_id, vendor_id, date, cost, impressions, media_cost, cost_type, billing_model, rate, currency…`
- Old **`vendor_billable_cost`** is explicitly marked as **removed** in the new design.
- All reports and APIs (`/vendor-cost/daily`, `/monthly`, etc.) read from `daily_vendor_cost`, and monthly billable cost is derived by `SUM(cost)` plus billing-model logic.
`Daily_vendor_cost_stats` does not contain any data. 



### Cost Breakdown 

Media cost is the total cost MiQ pays to the DSP per impression.
Data Cost is the cost MiQ pays for buying 3rd party data segments from places like Xandr marketplace.
Insight Cost is the cost for MiQ's own analytics/insight platforms.
Fixed Cost 
Ad-Serving Cost is the cost serve impressions through an ad server. 
Additional Vendors that MIQ pays. 
Vendor means who is being paid. 


Lab Campaign Table
* fixed_cost_value - fixed flat fee 
* ad_serving_rate_cost_value - cost to serve ads through an ad server
* 


Vendor Cost 
* Always_on - MiQ pays fixed amount monthly. Always on Contract
* Rev_share - Vendor shares revenue of the media cost
* Fixed - Fix fee that vendor charges
* CPM - Vendor charges via Cost per impressions




trading_center.daily_stats - not data for feb






So for t








# Costs

Adserving Cost = Impressions * ad_serving_rate / 1000
ad_serving_rate is in the campaign table. 





[Cost Calculation Doc 1](https://mediaiq.atlassian.net/wiki/spaces/MIQ/pages/3022618737/Cost+Calculation)

* Data cost Brand Safety = [Impressions * 0.04 * Exchange_rate]/1000_
* Total Data cost = Data Cost Third Party + Data Cost Brand Safety_
* Total cost = [Total Media Cost + Total Technology Cost + Total Data Cost]/(1-miq_service_rate)_
* Total miq service cost = Total Cost - [Total Media Cost + Total Technology Cost + Total Data Cost]_



[DSP Costs metrics map](https://docs.google.com/spreadsheets/d/1qcRInW2C1dzSNY53nwjaOu9L0Cja_anizkXNF4E_F6U/edit?gid=0#gid=0)





[Direct Querying Final Run](https://miqdigital-hub.cloud.databricks.com/jobs/812481160322491/runs/155780152161903?o=2820278049549475)

# SIGMA Actual DATA

* Media Cost - directly comes from DSP. We do not calculate it. LLD feeds include duplication and also some external fee that we cannot match exactly. Get it from `trading_agent.lab_monthly_info`
* Additional Vendor - This is something we calculate , data is present in `cost.daily_vendor_stats`.
* Insight Cost, Ad Serving Cost, Fixed Cost
* Data Cost 

DIsclosed vs Non-Disclosed campaigns

From DV360 we only get media cost. Xandr Market and 3rd party segments, etc are just 0.0. we only get media cost only directly. 


### Additional vendor costs 
Rev_share = Rate * Media Cost /100
Fixed Cost = fixed_cost = ( fixed_rate_cost / total_no_days_in_campaign ) * no_of_days_passed
Ad Serving Cost = Impressions x ad_serving_rate / 1000




Raw columns (Advertiser currency):

- **Media cost:**
    - `Media Cost (Adv Currency)` – raw media cost.
- **Data fees (dataCost):**
    - Many separate fee columns:
        - `adloox_fee`, `adloox_prebid_fee`, `adsafe_fee`, `adXpose_fee`,  
            `aggregate_knowledge_fee`, `data_fee`, `double_verify_fee`,  
            `double_verify_prebid_fee`, `evidon_fee`, `integral_adscience_pre_bid_fee`,  
            `integral_adscience_video_fee`, `moat_video_fee`, `nielsen_ad_rating_fee`,  
            `shoplocal_fee`, `teracent_fee`, `trust_metrics_fee`, `adlingo_fee`, `vizu_fee`, etc.
- **Tech fee (techCost):**
    - `Media Fee 1 (Adv Currency)` – partner/media fee % applied on media.

Plus there may be **Regulatory Operating Costs (ROC)** at IO level (DST / operating surcharges).

[](https://mediaiq.atlassian.net/wiki/spaces/MGP/pages/3714613262)[](https://mediaiq.atlassian.net/wiki/spaces/MGP/pages/4002480150)

### What Lab/Sigma calculates for DV360

Post re‑arch mapping (non‑YouTube):

[](https://mediaiq.atlassian.net/wiki/spaces/MGP/pages/3714613262)

- **DSP Media Cost (netMediaCost)**  
    - `Media Cost (Adv Currency) + ROC (Regulatory Operating Costs)`
- **DSP Data Cost (dataCost)**  
    - Sum of all the data/measurement fee columns listed above.
- **DSP Tech Cost (techCost)**  
    - `Media Fee 1 (Adv Currency)`.

If there are **static vendor costs or ad‑serving costs** that we model internally (e.g. Evidon/IAS brand safety for disclosed, or MiQ ad‑serving), those pieces are stored separately in Lab and **subtracted** when we want to reconstruct pure “DV360 total media cost” for validation:

```text
totalMediaCost_DV360
  = netMediaCost (Media + ROC)
  + techCost (Media Fee 1)
  + dataCost (sum of vendor/measurement fees)
  - staticCost (static data provider cost we add)
  - adServingCost (MiQ ad-serving CPM we add)
```

text

[](https://mediaiq.atlassian.net/wiki/spaces/MIP/pages/4789404557)

That’s the formula you quoted.

So:

- **DSP gives you:** Media, Media Fee 1, all data fee columns.
- **We derive:**  
    - `netMediaCost` bucket, `dataCost` bucket, `techCost` bucket,
    - then **adjust by subtracting internal static/ad‑serving bits** we layered on top.

---

## 3) TTD – what DSP gives vs what we derive

### Non‑disclosed TTD

For **non‑disclosed**, TTD console “My Reports” gives us a single **Partner Cost**:

[](https://mediaiq.atlassian.net/wiki/spaces/MGP/pages/4060151975)

- `Partner Cost (Adv Currency)` = **media + data + fee features + tech fees** (all rolled up).

For non‑disclosed we do not split this further in the DSP bucket:

- **DSP Media Cost** (non‑disclosed) = `Partner Cost (Adv Currency)`  
    → one number used for media cost in Lab.

No “netMediaCost + tech + data – static – adServing” breakdown here in the DSP bucket; that formula is used for disclosed validation.

---

### Disclosed TTD

For **disclosed** campaigns, TTD gives **separate columns**:

[](https://mediaiq.atlassian.net/wiki/spaces/MGP/pages/4060151975)

- `Media Cost (Adv Currency)` – raw media (netMediaCost).
- `Data Cost (Adv Currency)` – 3P data / brand safety etc.
- `Fee Features Cost (Adv Currency)` – ad‑serving, Nielsen, weather, etc.
- `TTD Margin (Adv Currency)` – tech/platform fee.

So

- **netMediaCost** = `Media Cost (Adv Currency)`
- **dataCost** = `Data Cost + Fee Features Cost`
- **techCost** = `TTD Margin`

**Total TTD cost for a disclosed campaign** (from DSP perspective) is:

```text
totalPartnerCost = netMediaCost + dataCost + techCost
```

text

But we have extra **internal “static” and “adserving” costs** in Lab/Sigma that are _not_ part of TTD’s own cost:

- Some static data provider costs
- Our ad‑serving CPM cost if we’re adding it on top

When validating disclosed media cost in cost‑service, they use:

[](https://mediaiq.atlassian.net/wiki/spaces/MIP/pages/4789404557)

```text
totalMediaCost_TTD
  = netMediaCost + techCost + dataCost - (staticCost + adServingCost)
```

text

The idea:

- Start from the DSP‑side components: `Media Cost`, `TTD Margin`, `Data + Fee Features`.
- Subtract the parts we inject ourselves (`staticCost`, Lab ad‑serving) so we don’t double‑count.
- Compare that to what’s in `daily_transparent_stats` / cost tables.

**DSP gives:** Media Cost, Data Cost, Fee Features Cost, TTD Margin.  
**We derive:** data bucket, tech bucket, then adjust by subtracting our own static/ad‑serving costs for clean comparison.