# Data contract

## 1. Scope and synthetic-data notice

This contract defines the tables, metrics, aggregation rules, business rules
and comparison rules for `real-estate-supply-pulse`.

**All data is synthetic.** Regions are fictional. No real volumes, ratios,
seasonality, table names or metric definitions from any employer are used.

Machine-readable metric definitions live in
[`definitions/metric_definitions.yaml`](../definitions/metric_definitions.yaml).
Together, this document and that file are the source of truth. If they
disagree, flag it and resolve before use.

## 2. Tables

### supply_daily
- **Grain:** date x customer_type x region x transaction_type x property_type
- **Dimensions:**
  - customer_type: `pro`, `private`
  - region: 6 fictional regions
  - transaction_type: `rent`, `buy`
  - property_type: `apartment`, `house`, `commercial`, `land`

| Metric | Type | Notes |
|---|---|---|
| `total_live_listings` | stock | Listings live at end of day |
| `newly_added_listings` | flow | Listings added during the day |
| `quality_score_sum` | stock | Sum of per-listing quality scores over live listings. Per-listing score is 0-100 (synthetic). Numerator of `avg_quality_score` |

### customers_daily
- **Grain:** date x customer_type (`pro` only). No region.
- Does not depend on `supply_daily`.

| Metric | Type | Notes |
|---|---|---|
| `active_customers` | stock | See business rules (section 5) |
| `new_customers` | flow | |
| `churned_customers` | flow | Counted on the day the grace period lapses |

### revenue_monthly
- **Grain:** calendar month x customer_type (`pro` only)

| Metric | Type | Notes |
|---|---|---|
| `subscription_revenue` | flow | Billed amounts, contract fee only, summed over the month. Add-on products are out of scope |

## 3. Aggregation rules

- **Over time:** stocks roll up as the **average of daily values** (weeks and
  months). Flows **sum**.
- **Across dimensions:** all metrics sum.
- **Ratios** are computed only after roll-up, and are never stored. Store
  numerator and denominator.
- **Complete periods only.** A partial week or month never feeds a comparison.

## 4. Derived metrics (never stored)

- `avg_quality_score = SUM(quality_score_sum) / SUM(total_live_listings)`
- `revenue_per_customer` (monthly) `= subscription_revenue / average daily active_customers in the same month`
- `churn_rate = SUM(churned_customers) over the period / active_customers at end of the day before the period starts`

`churn_rate` is the one named exception to the "average of daily values" rule
for stocks: its denominator is the opening stock, not an average.

## 5. Business rules

- **Active customer:** has a live contract, or is within the grace period
  after it ends. There is no listing threshold.
- **Grace period:** 30 days (documented parameter).
- **Renewal inside the grace window:** no churn and no new customer.
- **Churn timing:** counted on the day the grace period lapses. The customer
  stays active until then.
- **No revisions:** history is never revised, and
  `opening + new - churned = closing` always reconciles.
- **Churn lag:** because of the grace period, churn lags the real contract end
  by about 30 days. **The brief must say so.**
- **Returning customer:** a customer returning after the grace period has
  lapsed counts as new.

## 6. Comparison rules

- **Weekly metrics (listings, customers):** week-on-week (WoW) and
  year-on-year (YoY) on aligned ISO weeks with comparable weekday mix.
- **Revenue:** month-on-month and same month last year. The brief shows the
  latest complete month and labels it.
- **Non-comparable periods are flagged:** definition changes, holiday
  mismatches, data gaps. A comparison spanning a definition change is flagged
  non-comparable. A derived metric is non-comparable if any input metric
  changed definition.
- **Decompositions:**
  - Listings by `customer_type` (mix shift).
  - Revenue as active customers x revenue per customer.

## 7. Planted test cases

Documented here for the later generator and detection phases. Not implemented
in this phase.

1. **Live-listing definition change** (synthetic), effective 2025-07-01.
   Recorded in `metric_definitions.yaml` (`effective_from`, `change_note`) for
   `total_live_listings`, `newly_added_listings` and `quality_score_sum`.
   Comparisons spanning it are flagged non-comparable.
2. **Subscription price change**, visible with a lag as customers reach their
   next invoice.
3. **Late renewals inside the grace window.** These must not create churn.
4. **Anomalies in listings and customers**, for the detection phase.
