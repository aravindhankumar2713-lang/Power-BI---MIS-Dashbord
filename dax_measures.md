# DAX Measures

All measures below are written against the star schema described in the main README: `supplier_diversity_fact` connected to `vendor_dimension`, `client_dimension`, and `date_dimension`.

## Core KPIs

```dax
Total Diversity Spend =
SUM(supplier_diversity_fact[Spend])
```

```dax
Total Annual Target =
SUM(client_dimension[AnnualDiversityTarget])
```

```dax
% of Target Achieved =
DIVIDE([Total Diversity Spend], [Total Annual Target], 0)
```

```dax
Compliant Vendor Records =
CALCULATE(
    COUNTROWS(supplier_diversity_fact),
    supplier_diversity_fact[IsCompliant] = 1
)
```

```dax
Total Vendor Records =
COUNTROWS(supplier_diversity_fact)
```

```dax
Compliance % =
DIVIDE([Compliant Vendor Records], [Total Vendor Records], 0)
```

```dax
Average Performance Score =
AVERAGE(supplier_diversity_fact[PerformanceScore])
```

## Time Intelligence

```dax
Spend Last Month =
CALCULATE(
    [Total Diversity Spend],
    DATEADD(date_dimension[Date], -1, MONTH)
)
```

```dax
Spend MoM % Change =
DIVIDE(
    [Total Diversity Spend] - [Spend Last Month],
    [Spend Last Month],
    0
)
```

```dax
YTD Diversity Spend =
TOTALYTD([Total Diversity Spend], date_dimension[Date])
```

## Compliance Risk Flag

```dax
Certifications Expiring in 60 Days =
CALCULATE(
    DISTINCTCOUNT(supplier_diversity_fact[VendorID]),
    supplier_diversity_fact[CertExpiryDate] <= TODAY() + 60,
    supplier_diversity_fact[CertExpiryDate] >= TODAY()
)
```

This measure powers a conditional-formatted table on the Compliance Tracker page, flagging vendors whose certification will lapse within 60 days — so the team can proactively renew before an audit finds it.

## Ranking (for Top/At-Risk Supplier Tables)

```dax
Vendor Spend Rank =
RANKX(
    ALL(vendor_dimension[VendorName]),
    [Total Diversity Spend],
    ,
    DESC
)
```

```dax
Vendor Performance Rank =
RANKX(
    ALL(vendor_dimension[VendorName]),
    [Average Performance Score],
    ,
    DESC
)
```

## Notes on Measure Design

- Every KPI is written as a **measure**, not a calculated column — measures respect the current filter/slicer context, so the same `Compliance %` measure correctly recalculates whether the user has filtered to one client, one sector, or the whole business. This matters for a dashboard leadership will slice interactively.
- `DIVIDE()` is used instead of the `/` operator everywhere division happens, since `DIVIDE` safely returns a defined default (0 here) instead of throwing a division-by-zero error when a filter context returns no rows.
