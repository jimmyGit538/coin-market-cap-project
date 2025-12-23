CoinMarketCap Category Pipeline 

This folder contains the CoinMarketCap crypto data pipeline. 

It includes the original notebook, the refined version focused on the selected categories, and the full historical backfill.

The walkthrough below explains the porocess step by step.

**1. Version 1 – Initial Pipeline Setup (CMC_api_project.ipynb)**


This was the first working version of the pipeline. 
The goal here was simply to connect to the CoinMarketCap API, explore the structure of the data, and confirm we could load everything into BigQuery.
What this version does
  - Loads the API key and sets up headers.
  - Calls multiple CMC endpoints, including:
    --categories
    --category
    --map
  - Normalizes the JSON into clean tables.
  - dds an imported_at_utc timestamp to every load.
  - Writes everything into BigQuery using WRITE_APPEND.
  
This version pulled broad category data so we could see what was available before narrowing things down.



**2. Version 2 – Focused Pipeline for the 29 Selected Categories (CMC_api_project_V2.ipynb)**



Why we changed the approach : After reviewing the output from Version 1, we narrowed the scope. 
Instead of pulling the entire categories list, we moved to using only the 29 categories that were selected for the project

What changed in this version

  - We stopped using the general categories API.
  - We now only use the Category endpoint, targeted directly at the categories we selected.
  - We built a master coin list by pulling the top coins inside each category.
  - We deduped that list so each coin appears once, even if it belongs to multiple categories.
  - This list became the foundation for all future pulls (daily + historical).


What the revised code does

  - Loads the list of 29 category IDs that were chosen.
  - Calls the CMC Category endpoint to get the coins inside each category.
  - Flattens and merges all results into one master coin list.
  - Saves the coin list so it can be reused by all other parts of the pipeline.
  - Writes cleaned category coin data into BigQuery.

This version keeps the pipeline smaller, faster, cleaner, and directly aligned to the categories we care about.


**3. Historical Backfill – Full Year of Daily Quotes (CMC_Category_Focused_Historical.ipynb)**


Goal : Bring in a full year of daily historical data for every coin in the master list, without overloading the API or hitting rate limits.

How we pulled the data

The historical endpoint works best with smaller ranges, so instead of trying to pull an entire year at once, we used:
  - continuous 30-day windows, one month at a time.
  - Each run used the same script — we only adjusted:
  --time_start
  --time_end
  - Then we appended the results into the same BigQuery table.
    
What the historical script does

  - Takes the master coin list in batches.
  - Builds a request for the historical quotes endpoint.
  - Includes daily quote metrics (open, high, low, close, volume, market cap).
  - Adds supply values and the official CMC quote_timestamp.
  - Handles rate limits with a retry loop.
  - Flattens the JSON into rows.
  - Appends everything into BigQuery using WRITE_APPEND.
  - Adds an imported_at_utc timestamp so we know when each batch was loaded.

    
Historical coverage

We successfully pulled data month by month for 2025.

This gives us a full clean year of daily historical data for every coin in our category list.
