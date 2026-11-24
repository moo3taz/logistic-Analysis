# 🚛 Logistics Operations & Profitability Analysis (Power BI)

An end-to-end **Power BI** data analysis project tracking **300 shipments** (**444,027 kg** total weight) across **8 carriers**, **6 warehouses**, and **25 commercial customers** in Egypt (Jan–Oct 2026). The report connects financial profitability with delivery performance and high-value insurance risk management.

---

## 📊 Dashboard Preview

### Page 1: Financial & Revenue Overview


### Page 2: Operations, Delivery & Insurance Review


---

## 🗂️ Data Model (Star Schema)

The model follows a **Star Schema** architecture with one fact table and four dimension tables:

- **`Shipments` (Fact Table — 300 rows):** Shipment ID, ship date, customer/carrier/warehouse keys, weight, packages, distance, freight cost, discount %, declared value, planned vs. actual delivery days, and status.
- **`Customers` (Dimension — 25 rows):** Customer name, city, and business segment (`Wholesale`, `Manufacturing`, `Retail`, `E-commerce`, `Pharma`).
- **`Carriers` (Dimension — 8 rows):** Carrier name, transport mode (`Air`, `Road`, `Sea`, `Rail`, `Courier`), and operating cost per km.
- **`Warehouses` (Dimension — 6 rows):** Warehouse name, city, region, and pallet capacity.
- **`Calendar` (Date Table):** Generated via DAX (`CALENDAR`) and marked as a date table for chronological month sorting.

**Relationships (`1` → `*`, Single-direction):**
- `Customers[CustomerID]` → `Shipments[CustomerID]`
- `Carriers[CarrierID]` → `Shipments[CarrierID]`
- `Warehouses[WarehouseID]` → `Shipments[WarehouseID]`
- `Calendar[Date]` → `Shipments[ShipDate]`

---

## 🧮 Key DAX Measures

```dax
Total Shipments = COUNTROWS(Shipments)

Total Weight (kg) = SUM(Shipments[Weight_kg])

Avg Freight Cost = AVERAGE(Shipments[Freight_Cost])

Active Customers = DISTINCTCOUNT(Shipments[CustomerID])

Avg Shipments per Customer = DIVIDE([Total Shipments], [Active Customers])

Net Revenue =
SUMX(
    Shipments,
    Shipments[Freight_Cost] * (1 - Shipments[Discount_Pct])
)

Operating Cost =
SUMX(
    Shipments,
    Shipments[Distance_km] * RELATED(Carriers[Cost_per_km])
)

Profit Margin = [Net Revenue] - [Operating Cost]

Net Revenue (Excl. Courier) =
CALCULATE(
    [Net Revenue],
    Carriers[Carrier_Type] <> "Courier"
)

Net Revenue (Discount > 10%) =
CALCULATE(
    [Net Revenue],
    Shipments[Discount_Pct] > 0.10
)

Delivered Net Revenue =
CALCULATE(
    [Net Revenue],
    Shipments[Status] = "Delivered"
)

Delivered Revenue % = DIVIDE([Delivered Net Revenue], [Net Revenue])
```

**Insurance Flag Column (on `Shipments`):**
```dax
High_Value_Shipments =
IF(
    Shipments[Declared_Value] > 50000,
    "Needs Review",
    "Standard"
)
```

---

## 💡 Key Business Insights

1. **Revenue vs. Margin Paradox:** **Air** freight generated the highest net revenue (**1.02M EGP**, ~55% of total) but the lowest profit margin (**17.8%**) due to high operating costs (`835.6K EGP`). **Rail (60.8%)** and **Sea (60.5%)** delivered over **3x** the profit margin.
2. **Revenue Leakage:** Out of **1.86M EGP** in net revenue, **76.14% (1.42M EGP)** was realized from delivered shipments (`234` orders), while **~319K EGP (17.2%)** was lost to `Returned` (`31` orders) and `Cancelled` (`16` orders) shipments.
3. **On-Time Delivery Gap:** Only **46.6%** of delivered shipments arrived within `Planned_Days`, averaging **4.75 actual days** vs. **3.80 planned days**.
4. **Market Concentration:** **Wholesale** leads all segments (**35.9%** of net revenue), and **Greater Cairo** accounts for **30.3%** of total net revenue.
5. **High-Value Risk:** **46% of all shipments (138 orders)** exceed **50,000 EGP** in declared value (totaling **9.55M EGP**), requiring a dedicated insurance review workflow.

---

## 🎯 Recommendations

- Shift non-urgent shipments from **Air** to **Rail** and **Sea** to improve overall profit margins.
- Audit root causes of **Returned** orders—especially among high-value shipments—to recover lost revenue.
- Review carrier SLAs to close the ~1-day average delivery delay.

---

## 📁 Repository Structure

```text
├── logistic.pbix                # Power BI Desktop report file
├── Logistics_DAX_Data.xlsx      # Source dataset
├── screenshots/                 # Dashboard page previews
│   ├── page1_financial_overview.png
│   └── page2_operations_insurance.png
└── README.md
```
