# Power Query (M) Transformation Steps

These are the transformation steps applied in Power Query Editor before the data reaches the model, following the "shape before you model" principle — cleaning happens here, calculations happen later in DAX.

## `supplier_diversity_fact` query

1. **Source** — import `supplier_diversity_fact.csv`
2. **Changed Type** — set `Month` and `CertExpiryDate` to Date, `Spend` and `PerformanceScore` to Decimal Number, `IsCompliant` to Whole Number
3. **Removed Duplicates** — on the full row, to guard against any duplicate transaction records from source extracts
4. **Filtered Rows** — removed any row where `Spend` is null or negative (defensive check, mirroring the same data-quality logic used in the earlier SQL/Excel scrub — this dataset is generated clean, but the query step is included so the pipeline is production-ready against real, messier source extracts)
5. **Renamed Columns** — standardized casing across all column headers to match the data model naming convention
6. **Merged Queries** — no merge needed here; relationships are handled in the model view via the star schema instead of pre-joining in Power Query, which keeps the fact table lean and lets DAX handle cross-filtering

Example M code for the negative-spend guard (step 4):
```m
= Table.SelectRows(#"Changed Type", each [Spend] <> null and [Spend] >= 0)
```

## `vendor_dimension` query

1. **Source** — import `vendor_dimension.csv`
2. **Changed Type** — `VendorID`, `VendorName`, `CertificationType`, `Region`, `Sector` all set to Text
3. **Removed Duplicates** — on `VendorID`, since this is meant to be a clean one-row-per-vendor dimension table

## `client_dimension` query

1. **Source** — import `client_dimension.csv`
2. **Changed Type** — `Client`, `Sector` as Text; `AnnualDiversityTarget` as Decimal Number
3. **Removed Duplicates** — on `Client`

## `date_dimension` query

1. **Source** — import `date_dimension.csv`
2. **Changed Type** — `Date` as Date; `MonthNum`, `Year` as Whole Number; `Month`, `Quarter` as Text
3. Marked as a **Date Table** in Model View (Table Tools → Mark as Date Table), which is required for the `DATEADD` and `TOTALYTD` time-intelligence DAX measures to calculate correctly

## Why this structure

Splitting the source data into one fact table and three dimension tables — rather than working from one wide flat file — is what enables the star schema in the Data Model step. It keeps each Power Query step focused on cleaning a single table, makes the model easier to debug, and keeps DAX measures fast even as the fact table grows.
