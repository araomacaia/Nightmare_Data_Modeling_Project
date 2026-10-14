# Modeling Decisions

This page explains the main decisions made while turning the 23-table source model into a star schema. All decisions follow the approach taught in Baraa's Nightmare Data Model tutorial. They fit this dataset and scenario, and a different business question could justify different choices.

[Back to README](../README.md)

## Rules set before building

1. Build a star schema. Facts sit in the middle, dimensions around them, and facts never connect to each other.
2. Understand the grain of each table before changing it.
3. Every column must earn its place.
4. Protect the numbers: know the key totals and recheck them after each change.
5. Follow the naming standards from the start (English, snake_case, `dim_` / `fact_`, `_key` / `_id`, friendly names).

## Decisions

| Area | Decision | Why | Trade-off |
|---|---|---|---|
| Query organization | Group queries into staging, dimensions, facts and support. Work on references, never on the raw queries. | Raw and new tables stay separate, and the raw data is still there if needed later. | A few more queries to manage. |
| Customer entity | Keep the name `customer`, not `user`. | Most source tables use "customer". | A source table that says "user" needs explaining. |
| Customer contacts | Merge only the primary contact into `dim_customer`. | A customer has several contacts. Merging all of them duplicates customers. Primary-only keeps one row per customer. | Other contacts are not in the model. |
| Customer address | Merge address, city and region into `dim_customer` instead of a separate address dimension. | Keeps a star schema rather than a snowflake. | The dimension is wider. It can be split later if needed. |
| Join type | Use left joins for lookups and merges. | An inner join can silently remove customers or products that lack a match. | Unmatched rows show as nulls and need checking. |
| Test records | Remove the dummy customer (ID 999) and the dummy product. | They are not real business data. | None noted. |
| Product keys | Create `product_key` with an index column, because the source only has a product code. | Gives a clean numeric key for relationships. | The key is model-specific and is not a source value. |
| Category column | Split the combined category and subcategory value on the pipe, then capitalize words so keys match before merging. | Keys must match exactly or the merge returns nulls. | None noted. |
| Order flags | Combine order channel, status and priority into one junk dimension, `dim_order_flag`. | They are unrelated low-cardinality fields. One dimension for each would add clutter. | The dimension is a list of combinations rather than one entity. |
| Channel codes | Add a hand-entered mapping from channel code to channel name. | Raw codes such as 10 and 20 can be mistaken for numbers to sum. | The mapping must be maintained. New codes in the source will show blanks. |
| Header and detail | Build `fact_sales` from order lines and use the order header for keys and context. | The numbers live in the lines. Header and detail as two facts joined together causes confusion and wrong numbers. | Counting orders needs a distinct count. |
| Orders by year | Append the 2025 and 2026 order tables, and drop columns that did not match (legacy reference, gift message, order notes). | Same grain and same shape, so they belong together. | The source year is not kept, since the order date already shows it. |
| Pre-aggregated values | Drop the order total from the header. | It can be recalculated from the lines and may become out of date. | Needs a measure to get order totals. |
| Fact keys | Look up keys from the dimensions, then remove the text columns used for the lookup. | Facts should hold keys, dates and numbers. | Lookups on text are risky (see the product name issue in the README). |
| Duplicate product names | Remove two duplicated product rows in the model, using the source identifier. | A name-based lookup duplicated sales rows. | Better fixed in the source system. |
| Geography | Create `dim_geo` and connect it twice to `fact_sales` (ship-to active, bill-to inactive). | Keeps region and city detail in one place and keeps the fact table small. | The inactive relationship needs DAX to use. |
| Customer city in the fact | Remove customer city and region from the fact table, but keep ship-to and bill-to cities as keys. | Customer location belongs to the customer dimension. Ship-to and bill-to locations belong to the order event. | Requires understanding the business meaning of each field. |
| Campaign | Split the campaign log into `dim_campaign` and `fact_campaign_spend`. | Budget and dates describe the campaign. Daily clicks and spend are events. | None noted. |
| Campaign products | Explode the product list into rows, trim spaces, and keep it as `fact_promotion_coverage`. | It links campaigns and products and has no measures, so it is a factless fact. | The tutorial calls it an unusual case. It can be removed if not needed. |
| Inventory | Unpivot month columns into rows and convert text to dates. | Dates should be values, not column names. | None noted. |
| Order process | Use one accumulating snapshot, `fact_order_process`, instead of one fact table for each step. | The same money would repeat in orders, invoices and payments. The date questions matter more. | Amounts for invoices and payments are not in the model. |
| Unused tables | Disable load for exchange rates, invoices, invoice lines, payments and shipments. Delete the duplicate shipments table and the junk order list. | Keep the model limited to what is needed. | Specific future questions may need these tables back. |
| Sales targets | Make a small `fact_sales_target` with date and target revenue. | It is a monthly plan, so it is a small fact. It has no keys to other dimensions. | It only connects through the date dimension. |
| Date dimension | Create `dim_date` at the end with `CALENDARAUTO()` and connect it only where facts have dates. | It grows with the data and acts as a shared dimension. | Connecting every date column adds clutter and extra DAX. |
| Measures | Hold core measures in a `_measures` table. | One place for shared calculations, so people do not write different versions. | Only the obvious measures are included. |
| Summarization | Turn off default summarization for numeric columns that should not be summed. Use one short date format. | Avoids mistakes when people build visuals. | A manual pass over every table. |
| Security | Put the security relationship on `dim_customer` and filter with a `regional_access` role. | `dim_customer` filters two fact tables. | Products and other tables are not restricted. |
