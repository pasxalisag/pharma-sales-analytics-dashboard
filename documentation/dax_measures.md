# DAX Measures Reference

Pharma Sales Analytics Dashboard | 2025

This reference documents all 35 measures exported from Power BI. Expressions are reconstructed from the multiline Excel export; no calculation logic has been intentionally changed. Descriptions are explanatory documentation, not exported model metadata.

## Measure inventory

| Area | Measures |
| --- | ---: |
| 01 - Revenue | 4 |
| 02 - Orders | 3 |
| 03 - Customers | 5 |
| 04 - Products | 4 |
| 05 - Pricing & Discounts | 4 |
| 06 - Pareto Analysis | 6 |
| 07 - Validation | 9 |
| **Total** | **35** |

## Measure definitions

### 01 - Revenue

#### Gross Revenue

**Home table:** `_Measures`  
**Model display folder:** 01 - Revenue  
**Purpose:** Revenue before discounts, calculated at sales-row level.

```dax
Gross Revenue =
SUMX(
    FactSales,
    FactSales[Quantity] * FactSales[Unit_Price]
)
```

#### MoM Growth %

**Home table:** `_Measures`  
**Model display folder:** 01 - Revenue  
**Purpose:** Relative change from previous month revenue; blank when previous revenue is blank.

```dax
MoM Growth % =
VAR CurrentRevenue = [Total Revenue]
VAR PreviousRevenue = [Previous Month Revenue]
RETURN
    IF(
        ISBLANK(PreviousRevenue),
        BLANK(),
        DIVIDE(
            CurrentRevenue - PreviousRevenue,
            PreviousRevenue
        )
    )
```

#### Previous Month Revenue

**Home table:** `_Measures`  
**Model display folder:** 01 - Revenue  
**Purpose:** Previous month revenue using DATEADD when exactly one month-year is selected.

```dax
Previous Month Revenue =
IF(
    HASONEVALUE(DimDate[Month-Year]),
    CALCULATE(
        [Total Revenue],
        DATEADD(DimDate[Date], -1, MONTH)
    ),
    BLANK()
)
```

#### Total Revenue

**Home table:** `_Measures`  
**Model display folder:** 01 - Revenue  
**Purpose:** Sum of recorded Sales_Amount (net revenue).

```dax
Total Revenue =
SUM(FactSales[Sales_Amount])
```

### 02 - Orders

#### Avg Order Value

**Home table:** `_Measures`  
**Model display folder:** 02 - Orders  
**Purpose:** Net revenue divided by distinct orders; zero fallback.

```dax
Avg Order Value =
DIVIDE([Total Revenue], [Total Orders],0)
```

#### Total Orders

**Home table:** `_Measures`  
**Model display folder:** 02 - Orders  
**Purpose:** Distinct order IDs in the current sales context.

```dax
Total Orders =
DISTINCTCOUNT(FactSales[Order_ID])
```

#### Total Orders Display

**Home table:** `_Measures`  
**Model display folder:** 02 - Orders  
**Purpose:** Text-formatted order count for display; not a numeric measure.

```dax
Total Orders Display =
FORMAT([Total Orders], "#,##0")
```

### 03 - Customers

#### Active Customers

**Home table:** `_Measures`  
**Model display folder:** 03 - Customers  
**Purpose:** Distinct customer IDs in the filtered sales records.

```dax
Active Customers =
DISTINCTCOUNT(FactSales[Customer_ID])
```

#### Orders per Customer

**Home table:** `_Measures`  
**Model display folder:** 03 - Customers  
**Purpose:** Distinct orders per active customer; zero fallback.

```dax
Orders per Customer =
DIVIDE(
    [Total Orders],
    [Active Customers],
0
)
```

#### Repeat Customer Rate

**Home table:** `_Measures`  
**Model display folder:** 03 - Customers  
**Purpose:** Share of active customers with more than one distinct order in the current context.

```dax
Repeat Customer Rate =
DIVIDE(
    [Repeat Customers],
    [Active Customers],
0
)
```

#### Repeat Customers

**Home table:** `_Measures`  
**Model display folder:** 03 - Customers  
**Purpose:** Counts customers with more than one distinct order in the current context.

```dax
Repeat Customers =
VAR CustomerTable =
    FILTER(
        VALUES(FactSales[Customer_ID]),
        CALCULATE([Total Orders]) > 1
    )
RETURN
    COUNTROWS(CustomerTable)
```

#### Revenue per Customer

**Home table:** `_Measures`  
**Model display folder:** 03 - Customers  
**Purpose:** Net revenue divided by active customers.

```dax
Revenue per Customer =
DIVIDE([Total Revenue], [Active Customers])
```

### 04 - Products

#### Active Products

**Home table:** `_Measures`  
**Model display folder:** 04 - Products  
**Purpose:** Distinct product IDs in the filtered sales records.

```dax
Active Products =
DISTINCTCOUNT(FactSales[Product_ID])
```

#### Product Price Benchmark

**Home table:** `_Measures`  
**Model display folder:** 04 - Products  
**Purpose:** Unweighted mean of product-level average selling prices for selected products with positive quantity.

```dax
Product Price Benchmark =
AVERAGEX(
    FILTER(
        ALLSELECTED(DimProduct[Product_ID]),
        [Total Quantity] > 0
    ),
    [Avg Selling Price]
)
```

#### Product Quantity Benchmark

**Home table:** `_Measures`  
**Model display folder:** 04 - Products  
**Purpose:** Total quantity divided by active product count; zero fallback.

```dax
Product Quantity Benchmark =
DIVIDE(
    [Total Quantity],
    [Active Products],
0
)
```

#### Total Quantity

**Home table:** `_Measures`  
**Model display folder:** 04 - Products  
**Purpose:** Sum of quantities sold.

```dax
Total Quantity =
SUM(FactSales[Quantity])
```

### 05 - Pricing & Discounts

#### Avg Selling Price

**Home table:** `_Measures`  
**Model display folder:** 05 - Pricing & Discounts  
**Purpose:** Net revenue per unit sold; zero fallback.

```dax
Avg Selling Price =
DIVIDE(
    [Total Revenue],
    [Total Quantity],
0
)
```

#### Discounted Revenue Share %

**Home table:** `_Measures`  
**Model display folder:** 05 - Pricing & Discounts  
**Purpose:** Share of net revenue from sales rows with a positive discount, preserving existing filters.

```dax
Discounted Revenue Share % =
DIVIDE(
    CALCULATE(
        [Total Revenue],
        KEEPFILTERS(FactSales[Discount] > 0)
    ),
    [Total Revenue],
0
)
```

#### Total Discount Amount

**Home table:** `_Measures`  
**Model display folder:** 05 - Pricing & Discounts  
**Purpose:** Gross revenue less recorded net revenue.

```dax
Total Discount Amount =
[Gross Revenue] - [Total Revenue]
```

#### Weighted Avg Discount %

**Home table:** `_Measures`  
**Model display folder:** 05 - Pricing & Discounts  
**Purpose:** Expected discount amount divided by gross revenue; zero fallback.

```dax
Weighted Avg Discount % =
DIVIDE(
    [Expected Discount Amount],
    [Gross Revenue],
0
)
```

### 06 - Pareto Analysis

#### Customer Concentration %

**Home table:** `_Measures`  
**Model display folder:** 06 - Pareto Analysis  
**Purpose:** Share of active customers required to reach the 80% revenue threshold.

```dax
Customer Concentration % =
DIVIDE(
    [Customers Generating 80% of Revenue],
    [Active Customers]
)
```

#### Customers Generating 80% of Revenue

**Home table:** `_Measures`  
**Model display folder:** 06 - Pareto Analysis  
**Purpose:** First rank position at which cumulative revenue reaches at least 80%.

```dax
Customers Generating 80% of Revenue =
MINX(
    FILTER(
        ALL('Pareto Axis'[Rank Position]),
        [Pareto Cumulative Revenue %] >= 0.8
    ),
    'Pareto Axis'[Rank Position]
)
```

#### Pareto Cumulative Revenue

**Home table:** `_Measures`  
**Model display folder:** 06 - Pareto Analysis  
**Purpose:** Cumulative revenue of the top N positive-revenue customers, with customer ID as tie-breaker.

```dax
Pareto Cumulative Revenue =
VAR CurrentPosition =
    SELECTEDVALUE('Pareto Axis'[Rank Position])

VAR CustomerTable =
    FILTER(
        ADDCOLUMNS(
            ALLSELECTED(DimCustomer[Customer_ID]),
            "@Revenue", [Total Revenue]
        ),
        [@Revenue] > 0
    )

VAR TopCustomers =
    TOPN(
        CurrentPosition,
        CustomerTable,
        [@Revenue], DESC,
        DimCustomer[Customer_ID], ASC
    )

RETURN
    IF(
        NOT ISBLANK(CurrentPosition)
            && CurrentPosition <= COUNTROWS(CustomerTable),
        SUMX(TopCustomers, [@Revenue])
    )
```

#### Pareto Cumulative Revenue %

**Home table:** `_Measures`  
**Model display folder:** 06 - Pareto Analysis  
**Purpose:** Cumulative ranked customer revenue divided by total revenue in the current context.

```dax
Pareto Cumulative Revenue % =
DIVIDE(
    [Pareto Cumulative Revenue],
    [Total Revenue]
)
```

#### Pareto Revenue

**Home table:** `_Measures`  
**Model display folder:** 06 - Pareto Analysis  
**Purpose:** Revenue at the selected rank among positive-revenue customers, with deterministic customer ID tie-breaking.

```dax
Pareto Revenue =
VAR CurrentPosition =
    SELECTEDVALUE('Pareto Axis'[Rank Position])

VAR CustomerTable =
    FILTER(
        ADDCOLUMNS(
            ALLSELECTED(DimCustomer[Customer_ID]),
            "@Revenue", [Total Revenue]
        ),
        [@Revenue] > 0
    )

VAR TopCustomers =
    TOPN(
        CurrentPosition,
        CustomerTable,
        [@Revenue], DESC,
        DimCustomer[Customer_ID], ASC
    )

RETURN
    IF(
        NOT ISBLANK(CurrentPosition)
            && CurrentPosition <= COUNTROWS(CustomerTable),
        MINX(TopCustomers, [@Revenue])
    )
```

#### Pareto Threshold

**Home table:** `_Measures`  
**Model display folder:** 06 - Pareto Analysis  
**Purpose:** Returns 80% only at rank positions with nonblank Pareto revenue.

```dax
Pareto Threshold =
IF(
    NOT ISBLANK([Pareto Revenue]),
    0.8,
    BLANK()
)
```

### 07 - Validation

#### Absolute Revenue Deviation

**Home table:** `_Measures`  
**Model display folder:** 07 - Validation  
**Purpose:** Sum of absolute row-level differences between recorded and calculated net revenue.

```dax
Absolute Revenue Deviation =
SUMX(
    FactSales,
    ABS(
        FactSales[Sales_Amount] -
        (
            FactSales[Quantity] *
            FactSales[Unit_Price] *
            (1 - FactSales[Discount])
        )
    )
)
```

#### Blank Product IDs

**Home table:** `_Measures`  
**Model display folder:** 07 - Validation  
**Purpose:** Counts blank or empty product IDs in the product dimension.

```dax
Blank Product IDs =
COALESCE(
    COUNTROWS(
        FILTER(
            DimProduct,
            ISBLANK(DimProduct[Product_ID])
                || DimProduct[Product_ID] = ""
        )
    ),
0
)
```

#### Discount Validation

**Home table:** `_Measures`  
**Model display folder:** 07 - Validation  
**Purpose:** Difference between observed and expected discount amounts.

```dax
Discount Validation =
[Total Discount Amount] -
[Expected Discount Amount]
```

#### Duplicate Product IDs

**Home table:** `_Measures`  
**Model display folder:** 07 - Validation  
**Purpose:** Product dimension row count minus distinct product ID count.

```dax
Duplicate Product IDs =
COUNTROWS(DimProduct)
    - DISTINCTCOUNT(DimProduct[Product_ID])
```

#### Expected Discount Amount

**Home table:** `_Measures`  
**Model display folder:** 07 - Validation  
**Purpose:** Sum of quantity multiplied by unit price and discount for each sales row.

```dax
Expected Discount Amount =
SUMX(
    FactSales,
    FactSales[Quantity] *
    FactSales[Unit_Price] *
    FactSales[Discount]
)
```

#### Max Discount

**Home table:** `_Measures`  
**Model display folder:** 07 - Validation  
**Purpose:** Maximum discount in the current context.

```dax
Max Discount =
MAX(FactSales[Discount])
```

#### Min Discount

**Home table:** `_Measures`  
**Model display folder:** 07 - Validation  
**Purpose:** Minimum discount in the current context.

```dax
Min Discount =
MIN(FactSales[Discount])
```

#### Revenue Mismatch Rows

**Home table:** `_Measures`  
**Model display folder:** 07 - Validation  
**Purpose:** Counts sales rows whose absolute net revenue discrepancy exceeds 0.0100001; returns zero when none.

```dax
Revenue Mismatch Rows =
COUNTROWS(
    FILTER(
        FactSales,
        ABS(
            FactSales[Sales_Amount] -
            (
                FactSales[Quantity] *
                FactSales[Unit_Price] *
                (1 - FactSales[Discount])
            )
        ) > 0.0100001
    )
) + 0
```

#### Sales Rows

**Home table:** `_Measures`  
**Model display folder:** 07 - Validation  
**Purpose:** Number of sales fact rows.

```dax
Sales Rows =
COUNTROWS(FactSales)
```

## Interpretation and validation notes

- Repeat Customer Rate measures repeat purchasing within the selected period, not longitudinal retention.
- Pareto calculations include positive-revenue customers only. The denominator uses total revenue in the current context; consistent visual interactions and nonnegative revenue should be verified.
- Product Price Benchmark is an unweighted average across products, unlike the overall unit-weighted Avg Selling Price.
- Discount Validation is an aggregate reconciliation; Revenue Mismatch Rows and Absolute Revenue Deviation provide complementary row-level checks.
- Total Orders Display returns text and should not be used for numeric aggregation.
- All 35 measures are centralized in `_Measures` and organized into seven model display folders.
- Format strings and model descriptions were not populated in the export. No formatting metadata has been inferred.
- Expressions have not been executed against the PBIX as part of this documentation review.
