# Pricing-Optimization-Revenue-Impact-Analysis-Using-SQL

![12-Ways-to-Optimize-SQL-Queries_-scaled](https://github.com/user-attachments/assets/5992db78-9685-4751-93a8-1a7c438a227d)

## Table Of Contents
[Project Overview](#project-overview)

[Data Description](#data-description)

[Key Metrics And KPIs](#key-metrics-and-kpis)

[Analysis And Insights](#analysis-and-insights)

[Recommendations](#recommendations)

 ## Project Overview

As a Data Analyst for a mid-sized manufacturing company, I conducted a deep-dive analysis into our pricing architecture. The goal was to identify why revenue was growing slowly despite high sales volumes and to provide a data-driven roadmap for margin recovery.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## Business Problem

The leadership team identified three critical "leaks" in the business:

Revenue Stagnation: Slow growth despite consistent sales activity.

Margin Erosion: Profit margins varied wildly across products and regions.

Uncontrolled Discounting: Sales teams were applying discounts (up to 20%) without clear guidelines, leading to "giving away the farm" to close deals.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 ## Data Description

The dataset contains transaction-level data for corporate clients with the following key attributes:

Products: Models like Aero, Volt, Flash, Urban, and Energy.

Sales Metrics: Quantity (1–49 units per order), Base Price, Cost, and Revenue.

Pricing Levers: Discount % (stored as a decimal ranging from 0.10 to 0.20) and Actual Price (Final price after discount).

Scope: Multiple regions (North, South, East, West) and diverse corporate customers.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## Key Metrics And KPIs

To measure success, I defined and tracked four primary KPIs:

Gross Margin %: (Total Margin / Total Revenue) - Our baseline for efficiency.

Average Discount Rate: Tracking how close we are to our 20% ceiling.

Price Realization: The difference between our Base Price and Actual Price.

Customer Lifetime Volume: Identifying our "Whales" (Quantity > 40).


```sql
SELECT 
    CAST((SUM(Margin) / SUM(Revenue)) * 100 AS DECIMAL(5,2)) AS Total_Gross_Margin_Pct,
    CAST(AVG([Discount %]) * 100 AS DECIMAL(5,2)) AS Avg_Discount_Rate,
    CAST((SUM(Revenue) / SUM([Base Price] * Quantity)) * 100 AS DECIMAL(5,2)) AS Price_Realization_Pct,
    SUM(Quantity) AS Total_Units_Sold,
    CAST(SUM(Revenue) / SUM(Quantity) AS DECIMAL(18,2)) AS Avg_Selling_Price    
FROM SalesData; 
```

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## Analysis And Insights

### A. Product Efficiency Analysis

The Question: Which products generate the most profit, and which are underperforming?

The Logic: I used SUM and GROUP BY to calculate the margin percentage for each model to identify our "Stars."
```sql
SELECT 
    Model,
    SUM(Quantity) AS Total_Units,
    SUM(Revenue) AS Total_Revenue,
    CAST((SUM(Margin) / NULLIF(SUM(Revenue), 0)) * 100 AS DECIMAL(5,2)) AS Margin_Percent
FROM SalesData
GROUP BY Model
ORDER BY Margin_Percent DESC;
```
This query identified that Flash and Volt are high-efficiency products, maintaining margins above 32.9% even when discounted.

### B. Regional Discount Auditing

The Question: Are certain regions over-discounting?

The Logic: I calculated the average discount rate per region to see if sales teams in specific areas were hitting the 20% max too often.

```sql
SELECT 
    Region,
    AVG([Discount %]) * 100 AS Avg_Discount_Rate,
    AVG(Margin / Revenue) * 100 AS Regional_Profit_Margin
FROM SalesData
GROUP BY Region
ORDER BY Avg_Discount_Rate DESC;
```
This revealed "Discount Leakage" in regions where high discounts did not result in a proportional increase in sales volume.

### C. Strategic Customer Segmentation

The Question: Who are our high-value vs. high-risk customers?

The Logic: Using a CASE statement, I tiered customers based on their volume (max 49) and the discounts they demand.

```sql
SELECT 
    Customer,
    SUM(Quantity) AS Total_Volume,
    CASE 
        WHEN SUM(Quantity) >= 40 AND AVG([Discount %]) <= 0.13 THEN 'Tier 1: High Value'
        WHEN AVG([Discount %]) >= 0.18 THEN 'Tier 3: Margin Risk'
        ELSE 'Tier 2: Standard'
    END AS Customer_Segment
FROM SalesData
GROUP BY Customer;
```

### D. Identifying Pricing Power

The Question: Which models can sustain a price increase?

The Logic: I used a Subquery to filter for models that outperform the company-wide average margin of 32.90%.

```sql
SELECT 
    Model,
    CAST(AVG(Margin / Revenue) * 100 AS DECIMAL(5,2)) AS Product_Margin_Percent
FROM SalesData
GROUP BY Model
HAVING AVG(Margin / Revenue) > (SELECT AVG(Margin / Revenue) FROM SalesData);
```

## Recommendations

Based on the SQL analysis of the provided dataset, I proposed the following:

  Targeted Price Increase: Increase the Base Price of the Flash and Volt models by 2%. Their high margins suggest the market values them enough to absorb a price hike.

  Implement a Discount Floor: For "Tier 3" customers, I recommended a mandatory approval process for any discount exceeding 15% (0.15).

  Regional Standardization: Align the discount strategy of the South and West regions with the more profitable East region to plug a 3% margin leak.

  Volume Incentives: Shift from "flat discounts" to "volume-based discounts," where the 20% max discount is only available for orders exceeding 45 units.
