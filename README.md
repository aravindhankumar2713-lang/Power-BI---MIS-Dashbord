# Power BI Executive MIS Dashboard & KPI Tracker

A dynamic, multi-tab Power BI dashboard tracking **supplier diversity spend**, **certification compliance percentages**, and **supplier performance metrics** across enterprise clients — built with interactive drill-down filters to improve executive reporting visibility.

This project reflects the kind of MIS/KPI reporting work used to support enterprise supplier diversity programs across sectors like healthcare, aviation, finance, and manufacturing.

---

## Business Problem

Enterprise procurement and compliance teams need a single, executive-facing view to answer:
- How much diversity spend are we driving, and is it trending toward annual targets?
- What percentage of our supplier base holds active, compliant certifications (MBE, WBE, VOB, DOBE, SME)?
- Which vendors and clients are driving the most spend, and how are vendors performing?
- Where are the compliance risks (expiring or lapsed certifications) that need attention before an audit?

This dashboard consolidates multi-source vendor, client, and transaction data into one governed model so leadership can self-serve these answers instead of waiting on ad-hoc reports.

---

## Data Model (Star Schema)

| Table | Type | Description |
|---|---|---|
| `supplier_diversity_fact.csv` | Fact | Monthly vendor-client spend transactions, certification status, and performance scores |
| `vendor_dimension.csv` | Dimension | Vendor master data — certification type, sector, region |
| `client_dimension.csv` | Dimension | Enterprise client master data — sector, annual diversity spend target |
| `date_dimension.csv` | Dimension | Calendar table for time intelligence (month, quarter, year) |

The fact table connects to each dimension table on its respective key (`VendorID`, `Client`, `Month`/`Date`), following a standard star schema — this keeps relationships simple and DAX calculations fast, per Power BI modeling best practice.

---

## Repository Structure

```
powerbi-mis-dashboard/
├── README.md                          <- you are here
├── data/
│   ├── supplier_diversity_fact.csv    <- transactional fact table
│   ├── vendor_dimension.csv           <- vendor master data
│   ├── client_dimension.csv           <- client master data + targets
│   └── date_dimension.csv             <- calendar table
├── docs/
│   ├── dax_measures.md                <- all DAX measures used in the dashboard
│   ├── power_query_steps.md           <- Power Query (M) transformation steps
│   └── dashboard_build_guide.md       <- step-by-step guide to rebuild the dashboard in Power BI Desktop
```

> Note: the `.pbix` file itself isn't included in this repo (Power BI binary files are large and don't diff well in Git). Instead, this repo contains everything needed to **rebuild the exact dashboard from scratch** in Power BI Desktop — the source data, the Power Query transformation logic, and every DAX measure — which is also a better portfolio artifact since it documents your actual process.

---

## Dashboard Pages

1. **Executive Overview** — total diversity spend, overall compliance %, spend vs. target (card + gauge visuals)
2. **Spend Analysis** — spend by sector, client, and month (stacked bar + line chart), with drill-down from Sector → Client → Vendor
3. **Compliance Tracker** — compliance % by certification type and region, with a table of vendors nearing certification expiry
4. **Supplier Performance** — average performance score by vendor and sector, ranked to flag top and at-risk suppliers

All pages share a common slicer panel (Client, Sector, Region, Month) so any filter selection cross-filters every visual on the page.

---

## Tools Used
Power BI Desktop · Power Query (M) · DAX · Star schema data modeling

---

## How to Use This Repo
See `docs/dashboard_build_guide.md` for full step-by-step instructions to load this data into Power BI Desktop and rebuild the dashboard, including exact DAX formulas and Power Query steps.
