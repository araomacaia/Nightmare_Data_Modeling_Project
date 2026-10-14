# Validation Checklist

A practical checklist for checking a Power BI model while it is being restructured. It has two parts, so it is clear what the tutorial showed and what is only suggested.

[Back to README](../README.md)

## Part 1: Checks shown in the tutorial

**Before changing a table**
- [ ] Say what one row represents (the grain) for every table involved.
- [ ] Note the row count of the main table (for example, 60 customers after removing the test row).
- [ ] Pick one important total from the fact table (in the tutorial, the order line total) and record it.

**Before and after each merge**
- [ ] Use a left join, so rows from the main table are never lost.
- [ ] Check cardinality in the model view. One-to-one, or one-to-many from the lookup table, is safe.
- [ ] Group by the key and count rows. Every count should be 1.
- [ ] Check the row count of the main table again.
- [ ] For fact tables, apply the changes and compare the protected total with the recorded number.

**When a total changes**
- [ ] Group by the join columns to find which rows are duplicated.
- [ ] Look at one affected record (for example, one order) to see why it repeats.
- [ ] Check the dimension for duplicate names or codes.
- [ ] Fix, then compare the total again.

**Relationships**
- [ ] One-to-many, single direction, from dimension to fact.
- [ ] No relationship between two fact tables.
- [ ] Role-playing relationships marked active or inactive on purpose.

**Standards review (Phase 4)**
- [ ] Table prefixes (`dim_`, `fact_`) and snake_case column names.
- [ ] `_key` for created keys, `_id` for source identifiers.
- [ ] One date format across the model.
- [ ] Summarization turned off for numeric columns that should not be summed.

**Security**
- [ ] Use "View as" with the role and a sample user.
- [ ] Totals and region lists show only that user's region.
- [ ] Stop "View as" and confirm the full totals return.

## Part 2: Recommended additions (not part of the tutorial)

These are suggestions. Tick them only when they have actually been done, and record the result.

- [ ] Write down every protected total with the date it was checked.
- [ ] Test "View as" with at least two users from different regions.
- [ ] Test a user who is not in the security table, and note what happens.
- [ ] Record the grain, primary key and relationships for each table in one document.
- [ ] Check that every fact key has a match in its dimension (no orphan keys).
- [ ] Re-run the full checklist after any new source table or column is added.
- [ ] Measure file size and refresh time before and after cleanup, if performance is going to be discussed.
