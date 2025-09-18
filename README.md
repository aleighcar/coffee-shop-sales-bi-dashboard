# coffee-shop-sales-bi-dashboard
**Daily Grind Coffee Sales BI Dashboard**

*A Power BI case study: from flat file → star schema → actionable insights*

**Overview**

This project transforms raw point-of-sale transactions into a scalable business intelligence solution. Beyond visualizing a flat CSV, the dataset was re-engineered into a star schema, enhanced with DAX measures, and delivered as interactive Power BI dashboards covering sales, product performance, and customer behavior.

**Why it matters**: This mirrors how analytics is done in the real world—clean data model, robust KPIs, and decision-ready insights.

**Table of Contents**
- Overview
- Dataset
- Methodology
- Data Model (Star Schema)
- Key DAX Measures
- Dashboards
- Insights
- Limitations & Next Steps
- How to Use
- Repo Structure
- Skills Demonstrated
- Author

**Dataset**

**File**: *coffee_shop_sales.csv*

- **Rows / Columns**: 4,998 transactions, 15 original columns

- **Scope**: Multiple locations; includes products, customers (loyalty/guest), employees (baristas), timestamps, prices, and payment methods

- **Nature**: Fictional business (Daily Grind Coffee) with realistic POS fields

**Data fields include**:
- **Transaction**s: ID, Date, Time, Location, Payment Method

- **Products**: ID, Name, Category, Size, Unit Price, Quantity, Total Price

- **Customers**: Customer ID, Loyalty Points Earned

- **Employees**: Barista ID

**Methodology**

**1. Data Cleaning (Power Query)**

- Standardized types (Date, Time, numeric).

- Addressed missing values:

- size → “OneSize” for non-beverage items

- missing customer_id → mapped to Guest for analysis
- Verified uniqueness of transaction_id.

**2. Data Modeling (Power Query → Model View)**

- Converted the flat file into a star schema (FactSales + dimensions).
- Added derived attributes: hour, minute, weekday, AM/PM, time-of-day buckets, weekend flags.
- Created a proper DimDate table for time intelligence.

**3. DAX (Power BI)**
- Authored measures for Total Sales, AOV, YTD/PY sales, YoY%, transactions, customer counts, loyalty vs. non-loyalty, new vs. repeat, retention rate, etc.

**4. Dashboards**
- Built three pages: Sales Overview, Product Performance, Customer Insights with slicers, drill-through, and narrative flow.

**Data Model (Star Schema)**

**FactSales**
transaction_id, transaction_date, transaction_time, product_id, customer_id, barista_id, location, payment_method, quantity, unit_price, total_price, loyalty_points_earned, customer_type, hour, minute, second, weekday_name, is_weekend_flag, am_pm_flag, time_of_day

**DimProducts**
product_id, product_name, product_category, size

**DimCustomers**
customer_id, lifetime_spend (calc), lifetime_points (calc), is_repeat_customer_flag (calc), customer_type (Guest/Loyalty)

**DimEmployees**
barista_id

**DimDate**
date, year, month, monthNumber, quarter, weekday, weekNumber

**DimLocation**
location, store_type (Airport, University, Mall, Downtown)

**Relationships**

- FactSales[product_id] → DimProducts[product_id]

- FactSales[customer_id] → DimCustomers[customer_id]

- FactSales[barista_id] → DimEmployees[barista_id]

- FactSales[location] → DimLocation[location]

- FactSales[transaction_date] → DimDate[Date]

- Why star schema? Flexibility, performance, clarity, and scalability (future integration with SQL Server / warehouse).

**Key DAX Measures**

All formulas use a monospaced font for readability.

-- Core
total_sales = SUM(FactSales[total_price])

count_transactions = DISTINCTCOUNT(FactSales[transaction_id])

average_order_value = DIVIDE([total_sales], [count_transactions])

-- Time Intelligence (requires related DimDate)
total_sales_ytd =
TOTALYTD( [total_sales], DimDate[Date] )

total_sales_py =
CALCULATE( [total_sales], SAMEPERIODLASTYEAR(DimDate[Date]) )

total_sales_yoy_pct =
DIVIDE( [total_sales_ytd] - [total_sales_py], [total_sales_py] )

-- Loyalty splits
total_sales_loyalty =
CALCULATE( [total_sales], FactSales[customer_type] IN { "Loyalty" } )

total_sales_nonloyalty =
CALCULATE( [total_sales], FactSales[customer_type] IN { "Non-Loyalty" } )

-- Customers
count_customers = DISTINCTCOUNT(FactSales[customer_id])

count_customers_loyalty =
CALCULATE( [count_customers], FactSales[customer_type] IN { "Loyalty" } )

count_customers_nonloyalty =
CALCULATE( [count_customers], FactSales[customer_type] IN { "Non-Loyalty" } )

-- Repeat / New (requires DimCustomers flag)
count_customer_repeat =
CALCULATE( [count_customers], DimCustomers[is_repeat_customer_flag] = "Repeat" )

count_customer_new =
CALCULATE( [count_customers], DimCustomers[is_repeat_customer_flag] = "New" )

customer_retention_rate =
DIVIDE( [count_customer_repeat], [count_customers] )

**Dashboards**

**1) Sales Overview**

**Filters**: Product Category, Date, Location

**KPIs**: Total Sales, YoY Growth %, Average Order Value, Total Transactions

**Visuals**:

- Sales by Month (line)

- Payment Method Share (donut)

- Sales by Customer Type (donut)

- Sales by Barista (bar)

- Insights card (narrative summary)

**2) Product Performance**

**Filters**: Location, Date

**KPIs**: Total Sales, Average Basket Size, Total Transactions

**Visuals**:

- Top Products by Sales (bar)

- Sales by Product Category (stacked column)

- Product Drill-Through (matrix)

- Coffee Size Preference / Tea Size Preference (pie)

**3) Customer Insights**

**Filters**: Date, Location

**KPIs**: Total Customers, Total Repeat, Total New, Retention Rate

**Visuals**:

- Customer Favorite / Least Favorite (cards)

- Loyalty vs. Non-Loyalty Sales over Time (line)

- Retention Rate over Time (line)

- Top Customers by Spend (bar)

- Customer Type Sales by Location (stacked column)

- Most Popular Time of Day (stacked)

**Insights**

- **Sales**: Weekend/holiday spikes; University & Airport growth driven by higher traffic (transactions), not AOV.

- **Products**: Coffee + Bakery lead; Medium size preference → pricing/upsell opportunity; seasonal items create promotional bursts.

- **Customers**: Loyalty members deliver higher AOV and revenue share; frequent but lower-value guest purchases → loyalty enrollment opportunity; Downtown & University show stronger repeat behavior.

**Business impact**: Optimize staffing/inventory for peaks; refine pricing & promos (sizes, seasonal); strengthen loyalty conversion; address underperforming locations (e.g., Mall).

**Limitations & Next Steps**

**Limitations**

- Flat-file origins; limited customer/employee attributes (IDs only).

- No supplier cost or marketing data → profitability/ROAS not measured.

**Next Steps**

- **Database Integration**: Load star schema into SQL Server for scale and cross-domain reporting.

- **Data Enrichment**: Add demographics, supplier pricing, marketing campaigns.

- **Automation**: Scheduled refresh for near real-time KPIs.

- **Predictive Analytics**: Forecast sales; model retention/churn and uplift.

**How to Use**
1. Clone/Download this repo.
   
2. Open CoffeeShop_Dashboard.pbix in Power BI Desktop (latest version).
   
3. **(Optional)** Place coffee_shop_sales.csv in the repo’s /data folder and update the file path in Power Query if prompted.
   
4. Use slicers (Date, Location, Product Category) and drill-through to explore.

**Repo Structure**
/DailyGrindCoffee-BI

├─ CoffeeShop_Dashboard.pbix

├─ data/

│  └─ coffee_shop_sales.csv

├─ docs/

│  └─ Documentation.pdf

├─ screenshots/

│  ├─ sales-overview.png

│  ├─ product-performance.png

│  └─ customer-insights.png

└─ README.md


**Skills Demonstrated**
**1. Power Query**: cleansing, denormalization → star schema derivation

**2. Data Modeling**: fact/dim design, relationships, date table

**3. DAX**: time intelligence, segmentation, retention, KPI design

**4. Power BI**: dashboard UX, slicers, drill-through, narrative flow

**5. Business Analysis**: sales, product, and customer insights; recommendations

**6. Scalability Mindset**: SQL Server-ready schema & roadmap

**Author**

Ardonna Cardines — Data & Decision Analyst
