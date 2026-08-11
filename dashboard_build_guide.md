# Step-by-Step: Rebuilding the Dashboard in Power BI Desktop

This guide walks through loading the data in this repo into Power BI Desktop and rebuilding the full dashboard from scratch.

## 1. Load the data

1. Open Power BI Desktop → **Get Data** → **Text/CSV**
2. Import all four files from `/data`: `supplier_diversity_fact.csv`, `vendor_dimension.csv`, `client_dimension.csv`, `date_dimension.csv`
3. In the import preview, click **Transform Data** (not Load) to open Power Query Editor — apply the steps in `docs/power_query_steps.md` to each query
4. Click **Close & Apply**

## 2. Build the data model (star schema)

1. Go to **Model view** (left sidebar)
2. Drag to create relationships:
   - `supplier_diversity_fact[VendorID]` → `vendor_dimension[VendorID]`
   - `supplier_diversity_fact[Client]` → `client_dimension[Client]`
   - `supplier_diversity_fact[Month]` → `date_dimension[Date]`
3. Confirm each relationship is **one-to-many**, filtering from the dimension table to the fact table (arrow points toward the fact table)
4. Right-click `date_dimension` → **Mark as Date Table** → select `Date` column

## 3. Add DAX measures

1. Right-click `supplier_diversity_fact` in the Fields pane → **New Measure**
2. Add each measure from `docs/dax_measures.md` one at a time — paste the formula, name it exactly as shown, and set the correct format (currency for spend measures, percentage for compliance/target measures)
3. Organize measures into a display folder named "KPI Measures" for a cleaner Fields pane (right-click a measure → Properties → Display Folder)

## 4. Build Page 1 — Executive Overview

1. Insert a **Card** visual → `Total Diversity Spend` measure
2. Insert a **Gauge** visual → value: `% of Target Achieved`, target: 1.0 (100%)
3. Insert a **Card** visual → `Compliance %`
4. Insert a **Line Chart** → axis: `date_dimension[Month]`, values: `Total Diversity Spend` — this is the trend view
5. Add a slicer panel on the left: `Client`, `Sector`, `Region`, `Month` (use dropdown-style slicers to keep the layout clean)

## 5. Build Page 2 — Spend Analysis

1. Insert a **Stacked Bar Chart** → axis: `Sector`, values: `Total Diversity Spend`, legend: `CertificationType`
2. Set up drill-down: right-click the axis field well → add a hierarchy `Sector → Client → VendorName`, then enable the drill-down arrows on the visual (Format → Drill mode)
3. Insert a **Table** visual → columns: `VendorName`, `Total Diversity Spend`, `Vendor Spend Rank` — sort descending by rank

## 6. Build Page 3 — Compliance Tracker

1. Insert a **Donut Chart** → legend: `CertificationType`, values: `Compliance %`
2. Insert a **Stacked Column Chart** → axis: `Region`, values: `Compliance %`
3. Insert a **Table** visual → columns: `VendorName`, `CertificationType`, `CertExpiryDate`, `Certifications Expiring in 60 Days`
4. Apply **conditional formatting** on `CertExpiryDate`: Format → Conditional Formatting → Background Color → rule: if expiry date is within 60 days of today, fill red

## 7. Build Page 4 — Supplier Performance

1. Insert a **Bar Chart** → axis: `VendorName`, values: `Average Performance Score`, sorted descending
2. Insert a **Scatter Chart** → X: `Total Diversity Spend`, Y: `Average Performance Score`, size: number of transactions — this quadrant view flags high-spend/low-performance vendors that need attention
3. Insert a **Table** → `VendorName`, `Average Performance Score`, `Vendor Performance Rank`

## 8. Add interactivity & polish

1. Select all slicers on each page → **Format** → **Sync Slicers** (View ribbon → Sync Slicers pane) so filters apply consistently across all four pages
2. Add a consistent header/title bar and company color theme (View → Themes → customize to a consistent palette)
3. Set default landing page to **Executive Overview** (File → Options → Report Settings → default page)
4. Publish or save as `.pbix` for sharing

## 9. Validate before presenting

- Click through every slicer combination once to confirm no visual breaks or shows blank
- Spot-check 2–3 numbers against the raw CSV manually (e.g., total spend for one client) to confirm the model calculates correctly
- Turn on **Performance Analyzer** (View ribbon) to check no visual takes unusually long to render
