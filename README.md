# 🛍️ Superstore Products Sales Analytics

## 📌 Project Overview

This project is an end-to-end Data Analytics solution built on the **Superstore** retail dataset. It analyzes sales, profit, orders, customers, returns, shipping behavior, regional performance and product performance using **Excel, Power Query and Power BI (DAX)**.

The workbook contains three raw sheets (**Orders**, **Returns**, **People**). They were cleaned and loaded into Power BI, linked through a star-style data model with a custom **Calendar** table, and turned into an interactive **Products Sales Dashboard**.

---

## 📂 Dataset

Source file: `Superstore.xlsx`

| Sheet | Rows | Columns | Description |
|---|---|---|---|
| **Orders** | 9,994 | 21 | One row per order line item (order, customer, location, product, sales, quantity, discount, profit) |
| **Returns** | 296 | 2 | List of returned orders (`Order ID`, `Returned`) |
| **People** | 4 | 2 | Regional manager for each region (`Person`, `Region`) |

**Coverage:** Orders placed from 03-Jan-2014 to 30-Dec-2017 · 5,009 unique orders · 793 customers · 1,862 products · 49 states · 4 regions · 3 categories · 3 segments.

### Orders – Column Reference

| Group | Columns |
|---|---|
| Order | Row ID, Order ID, Order Date, Ship Date, Ship Mode |
| Customer | Customer ID, Customer Name, Segment |
| Location | Country, City, State, Postal Code, Region |
| Product | Product ID, Category, Sub-Category, Product Name |
| Measures | Sales, Quantity, Discount, Profit |

---

## 🛠️ Tools & Technologies Used

- Microsoft Excel (data inspection & validation)
- Power BI Desktop (Power Query, data modeling, visuals)
- DAX (Data Analysis Expressions)
- Git & GitHub

---

## 🧹 Data Cleaning & Preparation

The Excel sheets were profiled before loading. Missing values and exact duplicate rows were checked on all three sheets (none were found), and the points below were handled in Power Query.

| # | Check / Issue Found | Action Taken |
|---|---|---|
| 1 | No null values, no duplicate rows in Orders, Returns or People | Verified, no rows removed |
| 2 | `Row ID` is unique, `Order ID` follows a consistent `XX-YYYY-NNNNNN` pattern | Kept `Row ID` as line-level key; `Order ID` used for distinct order counts |
| 3 | **Postal Code** stored as a number, so 449 New England rows (CT, MA, ME, NH, NJ, RI, VT) lost their leading zero (4 digits instead of 5) | Converted to text and padded to 5 digits; marked as *Postal Code* data category |
| 4 | **Product Name** has leading/trailing spaces in 16 rows | Applied *Trim* and *Clean* |
| 5 | 32 `Product ID`s map to more than one `Product Name` | Documented; `Product ID` is kept for joins and `Product Name` is used for display |
| 6 | `Order Date` / `Ship Date` loaded as date-time | Set data type to *Date*; verified no ship date earlier than order date |
| 7 | `Returns` sheet has one column (`Returned = "Yes"`) and unique Order IDs | Used as a lookup of returned orders; kept 1 row per Order ID |
| 8 | `Region` in Orders matches `Region` in People (West, East, Central, South) | Used as the join key to map each region to its manager |
| 9 | Column types (Sales, Profit, Discount = Decimal, Quantity = Whole number) | Set explicitly to avoid wrong aggregation |
| 10 | No Calendar in source data | Created a dedicated **Calendar** table for time intelligence |

---

## 🧩 Data Model

![Data Model](05_Images/Data_Table_Relation.png)

The model contains **Orders, Orders (2), Returns, People and Calendar**.

| Table | Role |
|---|---|
| **Orders** | Main fact table (sales, profit, quantity, discount) |
| **Orders (2)** | Second copy of the fact table, used for role-playing relationships (inactive/secondary paths) |
| **Returns** | Dimension of returned orders, linked on `Order ID` |
| **People** | Dimension of regional managers, linked on `Region` |
| **Calendar** | Date dimension linked to `Order Date`, holds `Year`, `Year Month` and time-intelligence measures |

**Relationships (one-to-many, 1 → \*):**

- `Calendar[Date]` → `Orders[Order Date]`
- `Returns[Order ID]` → `Orders[Order ID]` (and a secondary/inactive path to `Orders (2)`)
- `People[Region]` → `Orders[Region]` (and a secondary/inactive path to `Orders (2)`)

> Dotted lines in the model view are inactive relationships, which can be activated in DAX with `USERELATIONSHIP()` when needed.

---

## 📊 Dashboard Preview

![Products Sales Dashboard](05_Images/Dashboard.png)

---

## 🎯 KPIs Used in the Dashboard

These are the five KPI cards at the top of the dashboard, plus the supporting measures behind the charts.

### KPI Cards

| KPI | Business Meaning | DAX |
|---|---|---|
| **TotalSales** | Total revenue from all order lines | `TotalSales = SUM(Orders[Sales])` |
| **TotalOrders** | Number of unique orders | `TotalOrders = DISTINCTCOUNT(Orders[Order ID])` |
| **Sum of Profit** | Total profit earned | `Sum of Profit = SUM(Orders[Profit])` |
| **TotalCustomers** | Number of unique customers | `TotalCustomers = DISTINCTCOUNT(Orders[Customer ID])` |
| **Total Returns** | Number of unique returned orders | `Total Returns = DISTINCTCOUNT(Returns[Order ID])` |

### Supporting Measures (Data Model)

| Measure | Purpose | DAX |
|---|---|---|
| **MarginOfProfit** | Profit as a percentage of sales | `MarginOfProfit = DIVIDE(SUM(Orders[Profit]), SUM(Orders[Sales]))` |
| **Total Sales** (Calendar) | Sales used on the time axis | `Total Sales = SUM(Orders[Sales])` |
| **PreviousYrSale** | Sales of the same period last year | `PreviousYrSale = CALCULATE([Total Sales], SAMEPERIODLASTYEAR(Calendar[Date]))` |
| **Year** | Year column for trend charts | `Year = YEAR(Calendar[Date])` |
| **Year Month** | Month level trend | `Year Month = FORMAT(Calendar[Date], "YYYY-MM")` |

### Additional KPIs You Can Derive from the Same Data

| KPI | Formula |
|---|---|
| Return Rate % | `DIVIDE([Total Returns], [TotalOrders])` |
| Average Order Value | `DIVIDE([TotalSales], [TotalOrders])` |
| YoY Sales Growth % | `DIVIDE([Total Sales] - [PreviousYrSale], [PreviousYrSale])` |
| Average Shipping Days | `AVERAGEX(Orders, DATEDIFF(Orders[Order Date], Orders[Ship Date], DAY))` |
| Quantity Sold | `SUM(Orders[Quantity])` |
| Average Discount % | `AVERAGE(Orders[Discount])` |

---

## 📈 Dashboard Visuals

| Visual | Fields Used | Question It Answers |
|---|---|---|
| KPI Cards (5) | TotalSales, TotalOrders, Sum of Profit, TotalCustomers, Total Returns | How is the business performing overall? |
| Sum of Sales and Sum of Profit by Year (line) | Calendar[Year], Sales, Profit | How are sales and profit trending year over year? |
| Sum of Sales by State (map) | Orders[State], Sales | Which states generate the most revenue? |
| Sum of Sales by Region (bar) | Orders[Region], Sales | Which region performs best? |
| TotalOrders by Ship Mode (donut) | Orders[Ship Mode], TotalOrders | How do customers prefer to get their orders? |
| Sales by Product Name (column) | Orders[Product Name], Sales | Which products sell the most? |
| **Slicers** | Segment, Category | Filter the whole dashboard by customer segment and product category |

---

## 💡 Key Business Insights

- Sales and profit dipped in **2015** and then grew steadily through **2016 and 2017**, with 2017 being the strongest year.
- **West** and **East** are the top-performing regions by sales, while Central and South contribute less.
- **Standard Class** is the dominant shipping mode, covering roughly 60% of orders; Same Day is the smallest.
- Sales are concentrated in a few large states (California and New York stand out on the map), which shows geographic concentration.
- A small number of products (mostly binding machines in the Office Supplies view) drive the top product sales.
- Returned orders are a small share of total orders but still worth monitoring, as each return reduces net revenue.

---

## 📁 Project Structure

```
Superstore-Sales-Analytics
│
├── 01_Data
│   └── Superstore.xlsx
│
├── 02_PowerBI
│   └── Superstore Sales Dashboard.pbix
│
├── 05_Images
│   ├── Dashboard.png
│   └── Data_Table_Relation.png
│
└── README.md
```

---

## 📌 Dashboard Features

✔ Interactive KPI Cards

✔ Segment & Category Slicers

✔ Yearly Sales vs Profit Trend

✔ State-wise Sales Map

✔ Region-wise Sales Analysis

✔ Order Distribution by Ship Mode

✔ Top Products by Sales

✔ Returns Tracking

---

## 🚀 How to Use

1. Download `Superstore.xlsx` and the `.pbix` file.
2. Open the `.pbix` in Power BI Desktop and point the data source to your local `Superstore.xlsx`.
3. Click **Refresh** to reload the data.
4. Use the **Segment** and **Category** slicers to explore the dashboard.

---

## 🏁 Project Outcome

This project turns three raw Excel sheets into a clean, related data model and an interactive dashboard. 
It demonstrates data cleaning, data modeling with relationships, DAX measure creation, time intelligence and dashboard design, 
helping business users track sales, profit, customers and returns in one place.

---

## 👨‍💻 Author

**Suman Kundu**

Data Analyst | Excel | Power BI | SQL | Python

GitHub: https://github.com/suman233

LinkedIn: https://www.linkedin.com/in/suman-kundu-093964153/
