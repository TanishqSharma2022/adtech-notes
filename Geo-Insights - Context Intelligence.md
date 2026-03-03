- Each Designated Marketing Area (DMA) is a region where the **population receives the same (or very similar) TV/content and ads**.

it’s basically listing **geo levels** we might support:
- High level: region/state
- Medium: city / DMA
- Fine: postal/ZIP

Issues right now
- **Expectation gap vs old CI**  
    Users are used to slicing insights by **regions, provinces, states, cities** and using that in RFP decks. That’s missing in the new Intent UI. In new UI it is just list states/cities and index of relevancy.
- **No strategy framing**  
    Even where we have geo data, we don’t help users answer:
    - “Where do we double down?”    
    - “Where do we test?”    
    - “Where is not worth it right now?”
Current Sigma Data CSV export
- region
- region index
- city
- city type
- city index
- city total users
- city overlap users
- region total users
- region overlap users

- A **brand** search:
    - Example: “Ford”, “Apple”, “McDonald’s”.        
    - Geo Insights would show: where _that brand’s_ intent is strongest.
        
- A **vertical** search:
    - Example: “Auto”, “Insurance”, “Retail”, “Travel”, “CPG snacks”.
    - Geo Insights would show: where **that whole category** is strong.
        - e.g. for Insurance in CA: which provinces/cities over‑index for “insurance shoppers” relative to baseline.
- **Pixel search** = search using **a specific first‑party pixel** as the anchor.
	- Geo Insights then answers:
	    _“Where do people who actually **hit this pixel** (visit or convert) cluster geographically?”_


### Audience Geo Insights

[Geo Insights in Audience Search Doc]([Geo Insights in Audience Search Doc](https://mediaiq.atlassian.net/wiki/spaces/MIP/pages/3502407740/Geo+Insights+for+Audience+Search+Automation))
![[Pasted image 20260303122213.png]]

For each **geo** (ZIP / city / DMA / state), you usually have:

- **total_users** – everyone we see in that geo
- **overlap_users** – people in that geo who match the **brand / audience** (e.g. visited brand site, in “in‑market SUV buyers”, etc.)

From this you can get:

- **Geo share of audience** =  
    `overlap_users in this geo / overlap_users across all geos`    
- **Geo share of population** =  
    `total_users in this geo / total_users across all geos`

In the automation doc, those are exactly the fields:
- `overlap_users_percent` ≈ geo share of audience
- `total_users_percent` ≈ geo share of population

Then the **index** is:
> **index = overlap_users_percent / total_users_percent**

Interpretation:
- **index = 1.0 (or 100)** → geo is **average** vs the country
- **index > 1.0 (or >100)** → the brand/audience is **more concentrated** in that geo than average → **over‑indexing**
- **index < 1.0 (or <100)** → **under‑indexing** (less relevant than average)

So “**over‑indexing vs baseline**” just means:
> “In this geo, the brand/audience shows up **more often than you’d expect** given its national average.”

