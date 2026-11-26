# Data Dictionary — Pharma Sales Analytics

## Dataset overview

Synthetic pharmaceutical and dietary supplement sales data for calendar year 2025. The source CSV contains 3,629 sales line records, 1,000 distinct orders, 180 customers, 250 products and 8 categories. Each row represents a sales line, not an entire order. Order_ID can repeat across lines.

## Source fields

| Column | Logical data type | Description | Example |
|---|---|---|---|
| Order_ID | Whole number | Order identifier; repeated across sales lines | 1 |
| Date | Date | Transaction date (YYYY-MM-DD) | 2025-09-21 |
| Month | Whole number | Calendar month number (1–12) | 9 |
| Year | Whole number | Calendar year | 2025 |
| Customer_ID | Text | Customer identifier | C093 |
| Product_ID | Text | Product identifier | P209 |
| Category | Text | Product category | Herbs |
| Quantity | Whole number | Units sold on the sales line | 1 |
| Unit_Price | Decimal number | Unit price before discount, in EUR | 9.37 |
| Discount | Decimal number | Discount rate stored as a fraction | 0.15 (15%) |
| Sales_Amount | Decimal number | Net sales amount after discount, in EUR | 7.96 |

## Business rules and validation

- Gross line revenue = Quantity × Unit_Price.
- Expected net line revenue = Quantity × Unit_Price × (1 − Discount).
- Observed discount rates: 0%, 5%, 10%, 12%, 15%, 20%.
- Distinct orders must be counted using Order_ID, not the number of sales lines.
- Each order has one customer and one discount rate in this dataset.
- No missing values or exact duplicate rows were found.
- All line revenue values agree with the expected calculation within EUR 0.005 (rounding tolerance).

## Reconciliation totals

| Metric | Value |
|---|---:|
| Gross revenue | EUR 193,872.34 |
| Net revenue | EUR 173,458.15 |
| Gross less net revenue | EUR 20,414.19 |
| Units sold | 9,702 |
| Distinct orders | 1,000 |

## Import notes

The original Power BI source uses comma-delimited CSV and Windows Greek (code page 1253). Decimal values use a period. When refreshing the PBIX on another computer, update the local source file path in Power Query and ensure numeric parsing uses a compatible locale.

## Limitations

The dataset is synthetic and covers only one calendar year. It contains no product cost or margin information; profitability and causal effects of discounts cannot be established from these fields alone.
