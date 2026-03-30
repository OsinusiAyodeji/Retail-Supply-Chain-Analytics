# Retail Supply Chain Analytics
> Retail operations generate millions in revenue while quietly absorbing hundreds of thousands in avoidable losses. This project diagnoses where — mapping delivery failures, margin erosion, and return risk across 5,009 orders into one operational intelligence system.

![Tool](https://img.shields.io/badge/Tool-Power%20BI-2E4057?style=flat-square)
![Dataset](https://img.shields.io/badge/Dataset-FP20%20Analytics-1D9E75?style=flat-square)
![Category](https://img.shields.io/badge/Category-Supply%20Chain-534AB7?style=flat-square)
![Pages](https://img.shields.io/badge/Report%20Pages-3-BA7517?style=flat-square)

## Abstract Summary

![Retail-Supply-Chain-Analytics](Supply%20Chain.png)



## Introduction

Most retail businesses can tell you what shipped. Very few can tell you what it actually cost them — in time, margin, and customer trust.

This project was built to close that gap. Using a retail supply chain dataset provided by FP20 Analytics, I took on the role of a supply chain analyst embedded in the business — not just building a dashboard, but constructing a diagnostic system that answers three operational questions that most reports leave unanswered:

> Are we delivering on time? Are we making money doing it? And where are returns costing us the most?

The dataset covers 5,009 orders across multiple product categories, regions, and ship modes. On the surface, the numbers look reasonable — $2.30M in total revenue, a functioning logistics operation, and products moving across every region. But beneath those headline figures, the data tells a different story.

| Metric | Value |
|---|---|
| Total orders | 5,009 |
| Total revenue | $2.30M |
| Late delivery rate | 30.07% — 1 in 3 orders |
| Revenue lost to returns | $180.50K |
| Revenue loss recorded | $156.13K |
| Profit margin | 12.47% |

To make sense of these numbers — and to make them actionable — the report is structured across three analytical pages, each targeting a distinct layer of supply chain performance:

**Page 1 — Delivery Performance & Lead Time Analysis**
Tracks SLA compliance by ship mode, identifies which product categories and regions are driving the most late deliveries, and surfaces where the gap between promised and actual delivery times is widest.

**Page 2 — Profitability & Supply Chain Health Scorecard**
Evaluates financial performance at the category, segment, and customer level. At its core is a custom Efficiency Score — a composite metric that ranks every state and city by a weighted combination of delivery speed, return rate, and profit contribution.

**Page 3 — Return Rate & Order Fulfillment Analysis**
Examines return patterns by product category, ship mode, and month. Crucially, it assigns accountability at the salesperson level — surfacing whose book of business carries the highest return exposure.

The goal was not to build a reporting tool. It was to build something that forces a decision — one that a logistics manager, a sales director, or a CFO could open on a Monday morning and immediately know where to focus.



## Problem Statement

Running a retail supply chain without the right diagnostic lens is expensive. Not because the data isn't there — it is. But because most reporting tools are built to summarise the past, not interrogate it.

This business had four visibility gaps that no standard report was addressing:



### Problem 01 — No accountability for late deliveries

30.07% of orders — nearly 1 in 3 — arrived late. Yet there was no structured view of which ship modes were violating SLA thresholds, which product categories were most affected, or which regions were consistently underperforming. Late delivery was treated as a fact of operations rather than a measurable, attributable failure.

> 1,506 late deliveries — Office Supplies alone accounted for 1,122


### Problem 02 — Profit leakage with no clear source

The business recorded $156.13K in revenue loss against $2.30M in total revenue — a margin of 12.47% that looks acceptable until you ask where the leakage is coming from. Without segment-level, region-level, and customer-level profit visibility, there was no way to distinguish between a pricing problem, a cost problem, or a logistics problem.

> $156.13K revenue loss — source unattributed across categories and regions



### Problem 03 — Returns treated as noise, not signal

A 5.91% return rate and $180.50K in revenue lost to returns were sitting in the data with no ownership. No breakdown by category, no trend analysis by month, and critically — no salesperson-level accountability. Returns were being absorbed as a cost of doing business when they were, in fact, a measurable and addressable risk.

> Anna Andreadi: 11.73% return rate — more than 3x the next salesperson



### Problem 04 — No standard for "good" supply chain performance

Without a composite benchmark, every region and city was evaluated in isolation. A state could have fast delivery but high returns and still appear healthy in a one-dimensional view. There was no weighted scoring model that brought delivery speed, return rate, and profitability together into a single, comparable metric.

> No efficiency benchmark — states like Arizona and California scoring 70% went undetected


These four gaps meant the business was making operational decisions — routing, stocking, staffing — without a shared definition of what good performance actually looked like. This project was built to provide that definition.



## Solution

The response to each problem was not a separate dashboard — it was a single connected system, built across three report pages, with a shared data model and a set of custom DAX measures designed to make every metric traceable back to a business decision.



### Report architecture

**Page 01 — Delivery Performance & Lead Time Analysis**
- SLA classification matrix mapping each ship mode to an industry-standard delivery window — Same Day (≤1 day), First Class (≤3), Second Class (≤5), Standard Class (≤10)
- Actual vs. expected delivery day comparison per order, surfacing where each ship mode is breaching its threshold
- Late delivery trend by month, category, and region — isolating whether the problem is seasonal, structural, or geographic
- On-time delivery rate KPI (69.93%) tracked with YoY and MoM variance to monitor directional change

**Page 02 — Profitability & Supply Chain Health Scorecard**
- Profit breakdown by product category, customer segment, and top-5 customers — separating revenue volume from genuine margin contribution
- Custom Efficiency Score: a weighted composite ranking every state and city across three determinant factors — delivery speed, return rate, and profit
- Product category distribution view to identify concentration risk and volume-profit mismatches
- Revenue loss quantified as a standalone KPI, not buried inside a profit waterfall

**Page 03 — Return Rate & Order Fulfillment Analysis**
- Return rate segmented by product category and ship mode — identifying whether returns are a product quality issue or a logistics one
- Monthly return trend to detect seasonality and recurring spikes
- Correlation scatter plot between ship mode and return rate — testing the hypothesis that faster shipping reduces return likelihood
- Salesperson accountability table: total orders, total returns, and return rate per rep — making individual exposure visible and comparable


### The Efficiency Score — methodology

The centrepiece of the scorecard page is a custom Efficiency Score, built to rank supply chain performance at the state and city level without relying on any single metric. The logic mirrors how a consulting engagement would define operational health: not just speed, not just profit, but the balance between all three.
![Retail-Supply-Chain-Analytics](Supply%20Chain.png)

| Determinant Factor | Weight | Threshold |
|---|---|---|
| Average Delivery Days | 50% | 34.61 days (dataset average) |
| Return Rate | 30% | 0 returns |
| Profit | 20% | Calculated average profit |

A location scores 100% when it meets or beats all three thresholds simultaneously. Anything below signals where operational investment is most needed before the business considers expanding into new markets.

---

### DAX engineering highlights

All measures were built with defensive coding conventions — ISBLANK checks to handle sparse data, underscore-prefixed VAR naming for readability, and SLA logic encoded directly in DAX rather than imported as static labels.

```dax
-- Delivery days calculation
Delivery Days =
VAR _OrderDate = MIN(Orders[Order Date])
VAR _ShipDate  = MIN(Orders[Ship Date])
RETURN
    IF(ISBLANK(_ShipDate), BLANK(), DATEDIFF(_OrderDate, _ShipDate, DAY))

-- SLA classification
SLA Status =
VAR _Days = [Delivery Days]
VAR _Mode = SELECTEDVALUE(Orders[Ship Mode])
RETURN
    SWITCH(TRUE(),
        _Mode = "Same Day"       && _Days <= 1,  "Within SLA",
        _Mode = "First Class"    && _Days <= 3,  "Within SLA",
        _Mode = "Second Class"   && _Days <= 5,  "Within SLA",
        _Mode = "Standard Class" && _Days <= 10, "Within SLA",
        "SLA Breached"
    )
```

---

### What this system makes possible

Individually, each page answers a question. Together, they form a closed diagnostic loop — delivery performance informs the efficiency score, the efficiency score contextualises the profitability view, and the return analysis completes the picture by surfacing the human and product factors that neither delivery nor profit metrics capture alone.

> This is not a dashboard that shows you what happened. It is a system that tells you why — and where to act next.

---

## Key Insights

The data surfaced five findings that move beyond summary statistics — each one actionable, each one pointing to a specific part of the operation.

---

**Delivery is broken at the mode level, not just in aggregate.**
Standard Class accounts for 918 SLA breaches — orders that took more than 10 business days despite being booked on a tier that promises delivery within that window. This is not a volume problem; it is a contract and routing problem. Same Day shipping, by contrast, shows only 7 breaches across the entire dataset.

**Office Supplies is the late delivery category, not a late delivery category.**
With 1,122 late deliveries — more than Furniture (529) and Technology (473) combined — Office Supplies has a fulfilment problem that is category-specific. The West region (474 late deliveries) and East region (424) are where this concentration is highest.

**Technology leads profit but carries the highest return rate.**
Technology generates the most profit at $184K, yet it also holds the highest return rate at 7.97%. This is the most fragile margin in the portfolio — high contribution, high exposure. A sustained increase in Technology returns could erode the category's profit leadership quickly.

**The efficiency scorecard separates real performance from headline numbers.**
Multiple states achieve a 100% efficiency score — meeting all three thresholds simultaneously. Arizona, Alabama, and California score 70%, flagging underperformance that would be invisible in a revenue-only view. The scorecard makes that gap impossible to overlook.

**One salesperson is carrying disproportionate return risk.**
Anna Andreadi's 11.73% return rate across 1,611 orders is not a rounding error — it is a signal. Chuck Magee (3.14%) and Kelly Williams (3.32%) operate at roughly a quarter of that rate. Without salesperson-level visibility, this exposure would have remained hidden inside the aggregate 5.91% figure.

---

## Recommendations

Five actions, prioritised by impact and feasibility.

---

**01 — Audit Standard Class logistics contracts**
918 SLA breaches on a single ship mode is not acceptable at scale. The business should review carrier performance data, renegotiate SLA terms, or route Standard Class orders through alternative fulfilment channels in high-breach regions. This is the single highest-volume failure in the dataset.

**02 — Investigate Office Supplies fulfilment as a standalone problem**
Office Supplies late deliveries (1,122) exceed the next two categories combined. This pattern suggests a warehouse, supplier, or picking process issue specific to this category — not a general logistics failure. A targeted root cause analysis, separate from broader delivery improvement efforts, is warranted.

**03 — Assign return rate KPIs to the sales team**
Anna Andreadi's 11.73% return rate requires immediate review — whether the cause is customer targeting, product misrepresentation, or order quality issues. More importantly, the absence of return rate accountability across the sales team means this kind of exposure can accumulate undetected. Return rate should be a standard performance metric alongside revenue and margin.

**04 — Use the efficiency scorecard to direct operational investment**
Before expanding into new markets or increasing volume in existing ones, states scoring below 100% should be prioritised for operational improvement. Investing in growth in a 70%-scoring state without first addressing the underlying delivery or return issues compounds the problem at scale.

**05 — Monitor Technology returns as a leading indicator**
Technology's combination of highest profit and highest return rate (7.97%) makes it the most important category to watch. A return rate monitoring cadence — monthly at minimum — should be established for this category specifically, with a defined threshold that triggers review before margin impact becomes material.

---

## Conclusion

This project set out to answer a question that most supply chain reports never ask: not what happened, but where the operation is failing — and who or what is responsible.

By building a diagnostic system across three connected pages, with a custom Efficiency Score at its core, the analysis moves beyond totals and trends into accountability. Every metric in this report is attributable — to a ship mode, a product category, a region, or a person. That is what turns data into a tool for decisions.

The technical foundation — DAX-driven SLA logic, defensive measure construction, weighted composite scoring — was built to be extensible. A version two of this system could layer in supplier lead time data, inventory stock levels, or forward-looking demand signals. The architecture supports it.

For now, the system answers the three questions it was built to answer. With clarity, and without ambiguity.

> Built with Power BI · Dataset by FP20 Analytics · Connect with me on [LinkedIn](https://www.linkedin.com/in/osinusi-ayodeji)

---
