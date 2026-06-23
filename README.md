# Predictive Customer Lifetime Value (CLV) Model for Acquisition Targeting

## Business Problem
Marketing teams often optimize acquisition campaigns around short-term conversion metrics (clicks, sign-ups) without accounting for how much a customer is actually worth over time. This project predicts each customer's **future value** from their early purchase behavior, then uses that prediction to evaluate which acquisition channels are genuinely worth investing in — not just which ones convert the most people.

## Dataset
A synthetic dataset of 4,000 customers and 42,241 transactions was generated to simulate realistic acquisition and purchase behavior across five channels (Paid Search, Email, Referral, Social, Organic), with deliberately designed behavioral differences by channel (e.g. referral-acquired customers purchase more frequently and spend more per order). Real Amex-style transaction data isn't publicly available, so this approach validates the methodology end-to-end on data with a known, designed signal — a standard technique before applying a method to messier real-world data.

## Methodology

**1. Temporal split** — each customer's history was split into a 90-day calibration window (used to build features) and a 180-day holdout window (used to calculate their actual future spend, the prediction target). This avoids the common mistake of "predicting" CLV using data that already includes the outcome.

**2. Feature engineering** — RFM features (Recency, Frequency, Monetary) were computed strictly from calibration-window data, combined with acquisition channel and demographics (age band, region).

**3. Modeling** — Linear Regression and XGBoost were trained and compared on an 80/20 train/test split:

| Model | RMSE | R² |
|---|---|---|
| Linear Regression | 2,772.31 | 0.524 |
| **XGBoost** | **2,461.60** | **0.625** |

XGBoost was selected as the final model. Feature importance showed `monetary_total` (calibration-window spend) and acquisition channel as the strongest predictors of future value; demographics (age band, region) contributed comparatively little.

**4. Segmentation** — the final model's predictions were used to cluster all customers into three value tiers (Low / Medium / High) via K-means, using predicted CLV plus supporting behavioral features rather than arbitrary percentile cutoffs.

| Tier | Customers | Avg Predicted CLV |
|---|---|---|
| Low | 1,749 | ₹2,797.65 |
| Medium | 1,925 | ₹4,101.44 |
| High | 326 | ₹14,072.77 |

**5. Statistical validation** — chi-square tests of independence checked whether tier membership was significantly associated with channel, region, and age band:

| Variable | Chi² | p-value | Significant? |
|---|---|---|---|
| Acquisition Channel | 3,623.85 | < 0.000001 | Yes — strong effect |
| Region | 14.30 | 0.026 | Yes — but weak; likely a sample-size artifact, not a meaningful effect |
| Age Band | 7.04 | 0.533 | No |

**6. Dashboard** — a Power BI dashboard ties predicted CLV and tier membership to an assumed acquisition cost (CAC) per channel, producing a CLV:CAC ratio that directly informs budget allocation:

| Channel | Avg Predicted CLV | Assumed CAC | CLV:CAC Ratio |
|---|---|---|---|
| Referral | ₹11,951.78 | ₹100 | **119.5x** |
| Organic | ₹4,458.27 | ₹150 | 29.7x |
| Email | ₹4,082.53 | ₹350 | 11.7x |
| Paid Search | ₹3,451.90 | ₹950 | 3.6x |
| Social | ₹2,325.46 | ₹700 | 3.3x |

*(CAC figures are illustrative assumptions used to demonstrate the methodology, not real Amex data.)*

## Key Finding
Referral-acquired customers show dramatically higher predicted lifetime value relative to acquisition cost than any other channel (119.5x vs 3.3-29.7x for others), and this relationship is statistically robust (chi² = 3,623.85, p < 0.000001) — while demographic factors (age, region) show no meaningful relationship to customer value. This suggests acquisition budget would be better allocated toward referral and organic growth than toward higher-cost paid channels, at least directionally, pending validation on real transaction data.

## Tech Stack
Python (pandas, numpy, scikit-learn, XGBoost, scipy), Power BI, Git/GitHub

## Repository Structure
```
clv-prediction/
├── data/
│   ├── raw/            # generated customer + transaction data
│   └── processed/       # split, feature-engineered, and prediction outputs
├── notebooks/           # one notebook per pipeline phase
├── dashboard/           # Power BI .pbix file
└── README.md
```

## Limitations
- Dataset is synthetic; real-world transaction data would likely show noisier, less cleanly separated patterns than seen here.
- CAC figures are assumptions, not actual acquisition costs.
- Model validated via temporal holdout rather than live deployment monitoring.