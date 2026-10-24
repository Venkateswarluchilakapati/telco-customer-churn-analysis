# Telco Customer Churn Analysis

An end-to-end data analysis project that finds out **which customers are leaving a telecom company, and what they have in common**, using Python for cleaning and analysis and Power BI for an interactive dashboard.

![Dashboard Preview](dashboard/dashboard_preview.png)

---

## Business Problem

A telecom company is losing customers every month. Management wants to know:

1. What is the overall churn rate, and how much revenue does it cost?
2. Does contract type affect churn?
3. Do customers with certain services (internet type, Tech Support) churn more?
4. Are new customers more likely to leave?
5. Does the payment method matter?
6. What actions can reduce churn?

## Dataset

- **Source:** Telco Customer Churn dataset (Kaggle)
- **Size:** 7,043 customers, 21 columns (one row per customer)
- **Target column:** `Churn` (Yes / No)
- **Includes:** demographics, services subscribed, contract and billing details, tenure, monthly and total charges

## Tools Used

| Tool | Purpose |
|---|---|
| Python (pandas, matplotlib) | Data cleaning and exploratory data analysis |
| VS Code + Jupyter | Development environment |
| Power BI Desktop | Interactive dashboard (DAX measures, slicers, charts) |

## Project Structure

```
churn-analysis-project/
├── data/
│   ├── Telco-Customer-Churn.csv     (raw data, never edited)
│   └── telco_churn_clean.csv        (cleaned data)
├── notebooks/
│   └── 01_cleaning.ipynb            (cleaning + EDA)
├── dashboard/
│   ├── telco_churn_dashboard.pbix
│   └── dashboard_preview.png
└── README.md
```

## Data Cleaning

| Issue | What I found | What I did |
|---|---|---|
| `TotalCharges` stored as text | Python read it as a string column | Converted to numeric with `pd.to_numeric(errors="coerce")` |
| 11 blank `TotalCharges` values | All 11 customers have `tenure = 0` (brand-new, not billed yet) | Filled with 0 instead of deleting the rows, because the missing value is logical, not an error |
| `SeniorCitizen` used 0/1 | Every other Yes/No column uses text | Mapped to No/Yes for consistent dashboard labels |
| No numeric churn column | Needed for averages and rates | Created `ChurnFlag` (1 = churned, 0 = stayed) |
| `tenure` was a raw number | Hard to compare groups | Created `TenureGroup` (0-12, 13-24, 25-48, 49-72 months) |

No duplicate customer IDs were found.

## Key Findings

**Overall:** 1,869 of 7,043 customers churned, a **26.5% churn rate**.

1. **Contract:** Month-to-month customers have the highest churn rate (42.7%), compared to 11.3% for one-year and 2.8% for two-year contracts.
2. **Internet service:** Fiber optic customers have a churn rate of 41.9%, compared to 19.0% for DSL customers.
3. **Tech Support:** Among customers with internet service, those without Tech Support have a churn rate of 41.6%, while those with Tech Support have 15.2%. Customers with no internet service have the lowest churn rate (7.4%), but they are a different group because they only have phone service.
4. **Tenure:** Customers in their first 12 months have the highest churn rate (47.4%), while those with 49-72 months have the lowest (9.5%).
5. **Payment method:** Electronic check customers have the highest churn rate (45.3%), while credit card (automatic) customers have the lowest (15.2%).
6. **Revenue impact:** Churned customers are 26.5% of customers but account for **30.5% of monthly revenue** (139,130.85 lost per month out of 456,116.60). Customers who leave pay more than average.

## Recommendations

1. **Move customers to longer contracts.** Offer discounts or perks to month-to-month customers for switching to one-year or two-year plans, because their churn rate (42.7%) is far higher than two-year customers (2.8%).
2. **Promote Tech Support.** Offer a free trial or bundle of Tech Support to internet customers who do not have it, because they churn at 41.6% compared to 15.2% for those who do.
3. **Focus retention on the first year.** Create an onboarding and early-retention program for new customers, because 47.4% of customers in their first 12 months churn.
4. **Encourage automatic payments.** Offer a small incentive for switching from electronic check to automatic payment, because electronic check customers churn at 45.3% versus 15.2% to 17% for automatic methods.

## Dashboard

The Power BI dashboard includes:

- **4 KPI cards:** Total Customers, Churned Customers, Churn Rate, Lost Revenue
- **4 charts:** Churn rate by Contract, Internet Service, Tenure Group, and Payment Method
- **2 slicers:** Gender and Senior Citizen, which filter every card and chart

**DAX measures used:**

```
Total Customers = COUNTROWS(telco_churn_clean)
Churned Customers = SUM(telco_churn_clean[ChurnFlag])
Churn Rate = DIVIDE([Churned Customers], [Total Customers])
Lost Monthly Revenue = CALCULATE(SUM(telco_churn_clean[MonthlyCharges]), telco_churn_clean[Churn] = "Yes")
```

### Validation

Dashboard numbers were cross-checked against Python results:

| Metric | Python | Power BI |
|---|---|---|
| Total customers | 7,043 | 7,043 |
| Churned customers | 1,869 | 1,869 |
| Churn rate | 26.5% | 26.5% |
| Lost monthly revenue | 139,130.85 | 139,130.85 |
| Female filter: customers / churned / rate | 3,488 / 939 / 26.9% | 3,488 / 939 / 26.9% |

## Limitations and Next Steps

- This analysis shows **association, not cause**. For example, electronic check customers churn the most even within month-to-month contracts (53.7% vs 31.6% to 34.1% for other payment methods), so contract type alone does not explain the pattern, but the reason is still unknown.
- The reason Fiber optic customers churn about twice as much as DSL customers is not in the data. Price, service quality, or competitors would need to be investigated.
- The dataset is a single snapshot, so trends over time cannot be analysed.
- **Next steps:** build a churn prediction model (logistic regression) to score customers by churn risk.

## How to Run

1. Clone this repository.
2. Open `notebooks/01_cleaning.ipynb` in VS Code or Jupyter and run all cells (needs `pandas` and `matplotlib`).
3. Open `dashboard/telco_churn_dashboard.pbix` in Power BI Desktop.

## Author

**Venkateswarlu Chilakapati**
LinkedIn: [https://www.linkedin.com/in/venkateswarlu-chilakapati/](https://www.linkedin.com/in/venkateswarlu-chilakapati/)
GitHub: [https://github.com/Venkateswarluchilakapati](https://github.com/Venkateswarluchilakapati)
