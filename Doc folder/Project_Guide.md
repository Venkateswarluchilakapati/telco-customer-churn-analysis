# Telco Customer Churn Analysis: Step-by-Step Project Guide

A complete walkthrough of the project, written as study notes. Each stage explains **What** it is, **Why** it matters, **Where** it is used in real jobs, and **How** to do it.

---

## Workflow Overview

| Stage | Task | Tool |
|---|---|---|
| 1 | Define business questions | Notes |
| 2 | Clean the data | Python (pandas) |
| 3 | Explore the data (EDA) | Python (pandas, matplotlib) |
| 4 | Build the dashboard | Power BI |
| 5 | Write findings, recommendations, README | Markdown |
| 6 | Publish | GitHub, LinkedIn |

---

## Project Folder Structure

```
churn-analysis-project/
├── data/            raw CSV (never edited) + cleaned CSV
├── notebooks/       Python notebooks (cleaning + EDA)
├── dashboard/       Power BI .pbix file + screenshot
└── README.md        project summary (the "front page")
```

**Why:** A tidy structure shows professional habits. Recruiters usually open the README first, then the dashboard, and only then the code.

When a notebook is inside `notebooks/` and the data is inside `data/`, use the path `../data/file.csv` (`../` means "go up one folder").

---

## Stage 1: Business Questions

**What:** The questions the project will answer.
**Why:** Analysis without a question produces charts without meaning.
**Where:** Every real analytics project starts with a business problem.

1. What is the overall churn rate, and how much revenue does it cost?
2. Does contract type affect churn?
3. Do customers with certain services (internet type, Tech Support) churn more?
4. Are new customers more likely to leave?
5. Does payment method matter?
6. What actions can reduce churn?

---

## Stage 2: Data Cleaning (Python)

**What:** Fixing data so it is correct and usable (types, missing values, duplicates).
**Why:** Wrong data gives wrong conclusions. Cleaning is usually 60-70% of an analyst's work.
**Where:** Every dataset in a real company needs cleaning before analysis.

### Dataset facts

- 7,043 rows, 21 columns, one row per customer
- No duplicate customer IDs
- `TotalCharges` is stored as text, and 11 of its values are blank

### Code

```python
import pandas as pd

df = pd.read_csv("../data/Telco-Customer-Churn.csv")
print(df.shape)          # (7043, 21)
df.info()

# 1. Convert TotalCharges from text to number
df["TotalCharges"] = pd.to_numeric(df["TotalCharges"], errors="coerce")
df.info()                # TotalCharges: 7032 non-null, float64

# 2. Inspect the 11 empty rows
df[df["TotalCharges"].isna()][["customerID", "tenure", "MonthlyCharges", "TotalCharges"]]

# 3. Decision: all 11 have tenure = 0 (brand-new customers, not billed yet)
df["TotalCharges"] = df["TotalCharges"].fillna(0)
print(df["TotalCharges"].isna().sum())     # 0

# 4. Make SeniorCitizen consistent with the other Yes/No columns
df["SeniorCitizen"] = df["SeniorCitizen"].map({0: "No", 1: "Yes"})

# 5. Numeric churn flag (1 = churned, 0 = stayed)
df["ChurnFlag"] = (df["Churn"] == "Yes").astype(int)
print("Churn rate:", df["ChurnFlag"].mean())   # 0.2653...

# 6. Tenure groups
df["TenureGroup"] = pd.cut(df["tenure"],
                           bins=[-1, 12, 24, 48, 72],
                           labels=["0-12 months", "13-24 months", "25-48 months", "49-72 months"])

# 7. Save the cleaned file
df.to_csv("../data/telco_churn_clean.csv", index=False)
```

### Key points

- `errors="coerce"` turns values that cannot be converted into NaN instead of crashing.
- Missing values are a **decision**, not just code. Here the 11 blanks are logical (new customers), so filling with 0 is better than deleting. Write the reason in the README.
- `ChurnFlag` makes rates easy: the mean of a 1/0 column is the percentage.

---

## Stage 3: Exploratory Data Analysis (EDA)

**What:** Exploring the data to find patterns that answer the business questions.
**Why:** You must know what the data says before building a dashboard.
**Where:** Done in every analysis project after cleaning.

### Code

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv("../data/telco_churn_clean.csv")

# Q1: Overall churn
print(df["Churn"].value_counts())                         # No 5174, Yes 1869
print(df["Churn"].value_counts(normalize=True) * 100)     # No 73.5%, Yes 26.5%

# Q1: Revenue impact
total_revenue = df["MonthlyCharges"].sum()
lost_revenue = df.loc[df["ChurnFlag"] == 1, "MonthlyCharges"].sum()
print(round(total_revenue, 2), round(lost_revenue, 2),
      round(lost_revenue / total_revenue * 100, 1))       # 456116.6, 139130.85, 30.5

# Q2: Contract
contract_churn = df.groupby("Contract")["ChurnFlag"].mean() * 100
print(contract_churn.round(1))      # Month-to-month 42.7, One year 11.3, Two year 2.8
contract_churn.plot(kind="bar", color="steelblue")
plt.title("Churn Rate by Contract Type")
plt.ylabel("Churn Rate (%)")
plt.xticks(rotation=0)
plt.show()

# Q3: Internet service
internet_churn = df.groupby("InternetService")["ChurnFlag"].mean() * 100
print(internet_churn.round(1))      # DSL 19.0, Fiber optic 41.9, No 7.4

# Q3: Tech Support
support_churn = df.groupby("TechSupport")["ChurnFlag"].mean() * 100
print(support_churn.round(1))       # No 41.6, No internet service 7.4, Yes 15.2

# Q4: Tenure
tenure_churn = df.groupby("TenureGroup", observed=True)["ChurnFlag"].mean() * 100
print(tenure_churn.round(1))        # 0-12: 47.4, 13-24: 28.7, 25-48: 20.4, 49-72: 9.5

# Q5: Payment method
payment_churn = df.groupby("PaymentMethod")["ChurnFlag"].mean() * 100
print(payment_churn.round(1).sort_values(ascending=False))
# Electronic check 45.3, Mailed check 19.1, Bank transfer 16.7, Credit card 15.2
```

### How `groupby` works

`df.groupby("Contract")["ChurnFlag"].mean()` splits customers into groups by contract type and takes the average of the 1/0 churn flag in each group. That average is the **churn rate**. It is the same idea as a pivot table in Excel.

### Optional deeper check (confounding)

```python
m2m = df[df["Contract"] == "Month-to-month"]
print((m2m.groupby("PaymentMethod")["ChurnFlag"].mean() * 100).round(1))
```

This compares payment methods only among month-to-month customers, which removes the contract effect.

---

## Stage 4: Power BI Dashboard

**What:** An interactive page with KPI cards, charts, and filters (slicers).
**Why:** Managers read dashboards, not code. It also makes a strong portfolio piece.
**Where:** Analysts at most companies build weekly or monthly dashboards.

### Steps

1. **Load data:** Get data, Text/CSV, select `telco_churn_clean.csv`, then Transform Data.
2. **Check types:** `MonthlyCharges` and `TotalCharges` = Decimal number; `tenure` and `ChurnFlag` = Whole number; the rest = Text. Then Close & Apply.
3. **Create measures** (Report view, right-click the table, New measure, one measure at a time):

```
Total Customers = COUNTROWS(telco_churn_clean)

Churned Customers = SUM(telco_churn_clean[ChurnFlag])

Churn Rate = DIVIDE([Churned Customers], [Total Customers])

Lost Monthly Revenue = CALCULATE(SUM(telco_churn_clean[MonthlyCharges]), telco_churn_clean[Churn] = "Yes")
```

4. **Format measures:** Select a measure, then Measure tools, then Format. `Churn Rate` = Percentage (1 decimal). `Lost Monthly Revenue` = Decimal number (2 decimals), **not** percentage.
5. **Build visuals:**
   - 4 KPI cards: Total Customers, Churned Customers, Churn Rate, Lost Monthly Revenue
   - 4 column charts with `Churn Rate` on the Y-axis: by Contract, InternetService, TenureGroup, PaymentMethod
   - 2 slicers: gender and SeniorCitizen
6. **Sort:** For Contract, Internet, and Payment charts use the chart's "..." menu, Sort axis, Churn Rate, Sort descending. Keep Tenure in natural order (0-12 to 49-72).
7. **Polish:** title text box, data labels on, align and distribute visuals, consistent theme.
8. **Save:** `dashboard/telco_churn_dashboard.pbix` and a screenshot `dashboard/dashboard_preview.png` (with all slicers cleared).

### Validation (always do this)

| Metric | Expected |
|---|---|
| Total Customers | 7,043 |
| Churned Customers | 1,869 |
| Churn Rate | 26.5% |
| Lost Monthly Revenue | 139,130.85 |
| With Female filter: customers / churned / rate | 3,488 / 939 / 26.9% |

If the dashboard matches Python, the dashboard is correct.

### Troubleshooting

| Problem | Reason | Fix |
|---|---|---|
| Churn Rate shows 0.27 | No percentage format | Measure tools, Format, Percentage |
| Lost Revenue shows "13913.1K%" | Percentage format applied to a revenue measure | Change to Decimal number |
| Cards show "7K" and "2K" | Display units set to Auto | Format visual, Callout values, Display units = None. Or use `FORMAT([Total Customers], "#,##0")` as a text measure in cards |
| "Syntax error" when pasting a measure | Pasted in DAX query view | Use Report view, then New measure, and paste one line only |
| Display units option not found | A chart (not a card) is selected | Click the number inside the card and check the icons in the Visualizations pane |

---

## Stage 5: Findings and Recommendations

**What:** Turning numbers into sentences with meaning.
**Why:** People rarely have time to read code or study charts. Clear sentences carry your message.
**Where:** README, LinkedIn post, interviews.

### Sentence pattern

> [Group] has the [highest/lowest] churn rate ([x]%), compared to [other group] ([y]%).

Always use **group + rate + number**.

### Key findings

1. Month-to-month customers have the highest churn rate (42.7%), compared to 11.3% for one-year and 2.8% for two-year contracts.
2. Fiber optic customers have a churn rate of 41.9%, compared to 19.0% for DSL customers.
3. Among customers with internet service, those without Tech Support have a churn rate of 41.6%, while those with Tech Support have 15.2%. Customers with no internet service (7.4%) are a different group.
4. Customers in their first 12 months have the highest churn rate (47.4%), while those with 49-72 months have the lowest (9.5%).
5. Electronic check customers have the highest churn rate (45.3%), while credit card (automatic) customers have the lowest (15.2%).
6. Churned customers are 26.5% of customers but 30.5% of monthly revenue (139,130.85 lost per month).

### Recommendations

1. Offer incentives to move month-to-month customers to longer contracts.
2. Offer a free trial or bundle of Tech Support to internet customers without it.
3. Build an onboarding and early-retention program for the first 12 months.
4. Encourage automatic payments instead of electronic check.

### Rules for writing findings

- **Count vs rate vs total:** Before writing any number, ask "is this a count, a rate, or a total?" "Month-to-month has more customers" and "month-to-month has a higher churn rate" are different claims.
- **Association is not cause:** Write "customers with X churn more", not "X causes churn".
- **Treat "No internet service" separately:** They have no internet, so Tech Support does not apply to them.

---

## Stage 6: Publishing

1. Create a GitHub repository named `telco-customer-churn-analysis`.
2. Upload the folders: `data`, `notebooks`, `dashboard`, and `README.md`.
3. Make sure `dashboard/dashboard_preview.png` exists so the README image displays.
4. Post on LinkedIn: the dashboard screenshot, 3 key findings, and the repository link.

---

## Interview Questions You Should Be Able to Answer

1. **Why did you fill the 11 missing `TotalCharges` values with 0?**
   All 11 had tenure 0. They were new customers who had not been billed, so 0 is logically correct, and keeping the rows preserves data.

2. **What does "30.5% of revenue" mean?**
   Revenue from churned customers (139,130.85) divided by total monthly revenue (456,116.60). Churned customers are 26.5% of customers but 30.5% of revenue, so they pay more than average.

3. **Why is the churn rate of "No internet service" customers so low?**
   They only have phone service, which is a different customer type. It should not be compared with Tech Support effects.

4. **Why do Fiber optic customers churn more?**
   The data does not say. Possible reasons (price, service quality, competition) need further investigation.

5. **Does electronic check cause churn?**
   Not necessarily. The data shows an association. That group may also overlap with month-to-month contracts, so I would check payment method within a single contract type.

6. **How did you validate your dashboard?**
   I compared the dashboard cards (total, churned, rate, revenue) and a Female-filtered view with Python results, and all matched.

7. **What is the difference between a measure and a column in Power BI?**
   A column is calculated per row and stored. A measure is calculated on demand and recalculates when filters change.

8. **What would you do next?**
   Build a churn prediction model (logistic regression) to score customers by churn risk.

---

## Author

**Venkateswarlu Chilakapati**
LinkedIn: https://www.linkedin.com/in/venkateswarlu-chilakapati/
GitHub: https://github.com/Venkateswarluchilakapati
