# Loan-Default-Dashboard 

**Dashboard Pages:** Loan Default & Overview · Applicant Demographics & Financial Profile · Financial Risk Matrix

---

## 1) Problem Statement

Banks and NBFCs need a clear picture of which borrower segments, products, and time periods are driving loan defaults. This Power BI report brings those signals together - default rates, income/employment cohorts, credit-score bins, and YoY/YTD trends so risk teams can focus on the highest-impact levers.

**What this really means:**

* See where default concentrates by employment type and purpose.
* Compare loan sizes (avg/median) across credit bands and age groups.
* Track change over time: YoY loan amount and YoY default-rate movement.
* Drill into YTD loan flows by credit-score bins, marital status, and employment type.

---

## 2) Dataset

**Name:** Loan Default Dataset
**Description:** Borrower applications with financial attributes, loan characteristics, repayment behavior, and outcomes.
---

## 3) Model & Data Preparation

### Power Query (Excel/Power BI)

* Profiled columns using **column profile/quality/distribution**.
* Cleaned nulls in numeric fields used in averages/medians.
* Validated ranges (e.g., credit score bins) and mapped text categories.

---

## 4) Key DAX Measures

> Adjust table/column names to your model. These mirror the measures used for visuals in this report.

**Base Measures**

```DAX
Total Loan Amount = SUM(Loans[LoanAmount])
Applicant Count = COUNTROWS(Loans)
Default Count = COUNTROWS(FILTER(Loans, Loans[Defaulted] = "Yes"))
Default Rate = DIVIDE([Default Count], [Applicant Count])
Average Loan = AVERAGE(Loans[LoanAmount])
Average Income = AVERAGE(Loans[AnnualIncome])
```

**Non-blank aggregation used in “Loan Amount by Purpose”**

```DAX
Total Loan Amount (NonBlank) =
SUMX(
    FILTER(Loans, NOT ISBLANK(Loans[LoanAmount])),
    Loans[LoanAmount]
)
```

**Average Income by Employment Type**

```DAX
Avg Income by Emp =
CALCULATE(
    [Average Income],
    ALLEXCEPT(Loans, Loans[EmploymentType])
)
```

**Default Rate by Employment Type**

```DAX
Default Rate by Emp =
CALCULATE(
    [Default Rate],
    ALLEXCEPT(Loans, Loans[EmploymentType])
)
```

**Average Loan by Age Group**

```DAX
Avg Loan by AgeGroup =
AVERAGEX(
    VALUES(Loans[AgeGroup]),
    CALCULATE([Average Loan])
)
```

**Median Loan Amount (with validation using MEDIANX)**

```DAX
Median Loan Amount =
MEDIANX(
    FILTER(Loans, NOT ISBLANK(Loans[LoanAmount])),
    Loans[LoanAmount]
)

Median Loan by Credit Category =
CALCULATE(
    [Median Loan Amount],
    ALLEXCEPT(Loans, Loans[CreditScoreCategory])
)
```

**Year-over-Year (YoY) Measures**

```DAX
YOY Loan Amount Change = 
DIVIDE(
    CALCULATE(SUM('Loan Default'[LoanAmount]),'Loan Default'[Year]=YEAR(MAX('Loan Default'[Loan_Date_DD_MM_YYYY])))-
    CALCULATE(SUM('Loan Default'[LoanAmount]),'Loan Default'[Year]=YEAR(MAX('Loan Default'[Loan_Date_DD_MM_YYYY]))-1)

    , CALCULATE(SUM('Loan Default'[LoanAmount]),'Loan Default'[Year]=YEAR(MAX('Loan Default'[Loan_Date_DD_MM_YYYY]))-1),0) *100
```

**YTD Loan Amount**

```DAX
YTD Loan Amount = 
CALCULATE(SUM('Loan Default'[LoanAmount]),DATESYTD('Loan Default'[Loan_Date_DD_MM_YYYY])
,ALLEXCEPT('Loan Default','Loan Default'[Credit Score Bins],'Loan Default'[MaritalStatus]))
```

**Decomposition Tree Helper (example using SWITCH  and IF)**

```DAX
Risk Lens =
SWITCH(
    TRUE(),
    [Default Rate] > 0.15, "High Risk",
    [Default Rate] > 0.08, "Medium Risk",
    "Low Risk"
)
```

**Bucketing Columns (Power Query or Calculated Columns)**

```DAX
Age Groups = 
IF('Loan Default'[Age]<19,"Teen",
        IF( 'Loan Default'[Age]<=39,"Adults",
        IF('Loan Default'[Age]<=59,"Middle Aged Adults",
        "Senior Citizens")))
)

CreditScoreCategory =
SWITCH(TRUE(),
    Loans[CreditScore] < 580, "Very Low",
    Loans[CreditScore] < 670, "Low",
    Loans[CreditScore] < 740, "Medium",
    "High"
)

IncomeBracket =
SWITCH(TRUE(),
    Loans[AnnualIncome] < 40000, "Low Income",
    Loans[AnnualIncome] < 90000, "Medium Income",
    "High Income"
)
```

---

## 5) Report Pages & Visuals

### **Page 1 — Loan Default & Overview**

**Visuals**

* **Loan Amount by Purpose** (Column + line marker) → uses *Total Loan Amount (NonBlank)*.
* **Average Income by Employment Type** → *Avg Income by Emp*.
* **Default Rate by Employment Type** → *Default Rate by Emp*.
* **Default Rate by Year (2013–2018)** → *Default Rate* with Year on axis; tooltip shows *Default Rate YoY (pp)*.

**Snapshot placeholders:**

*<img width="1366" height="780" alt="Image" src="https://github.com/user-attachments/assets/1a2d7ff6-65a7-416c-bd64-b1d0c1622c42" />

### **Page 2 — Applicant Demographics & Financial Profile**

**Visuals**

* **Median Loan Amount by Credit Score Category** → *Median Loan by Credit Category*.
* **Average Loan by Age Group & Marital Status** (Donut) → *Avg Loan by AgeGroup* with legend *MaritalStatus*.
* **Total Loan (Adults) by Credit Categories** → *Total Loan Amount* filtered to `AgeGroup = Adults` with `CreditScoreCategory` on axis.
* **Total Loan (Middle-Aged) by Mortgage/Dependents** → clustered columns with `HasMortgage`/`HasDependents`.
* **Number of Loans by Education Type** (if available) → count of `ApplicantID`.

**Snapshot placeholders:**

* <img width="1381" height="788" alt="Image" src="https://github.com/user-attachments/assets/5066981d-c747-4619-8aa0-b6c0ba3564a2" />

### **Page 3 — Financial Risk Matrix**

**Visuals**

* **YoY Loan Amount Change by Year** → *Loan Amount YoY %* (line chart).
* **YoY Default Loans Change by Year** → *Default Rate YoY (pp)* (line chart).
* **YTD Loan Amount by Credit Score Bins & Marital Status** → *Total Loan Amount YTD* with `CreditScoreCategory` as category and `MaritalStatus` as legend.
* **Decomposition Tree** slicing **IncomeBracket → EmploymentType → CreditScoreCategory** with *Default Rate* as the analyzed measure and *Risk Lens* providing a qualitative drill.

**Snapshot placeholders:**

* <img width="1396" height="792" alt="Image" src="https://github.com/user-attachments/assets/32c5a5da-3e10-41bb-8efc-5c642823c92a" />

---

## 6) Steps Followed (Build Log)

1. **Loaded** CSV files into Power BI Desktop.
2. Opened **Power Query** → turned on **Column quality/column distribution/column profile** and set profiling to **Entire dataset**.
3. **Validated data**: handled blanks in `LoanAmount`, `AnnualIncome`, and ensured `Defaulted` is strictly Yes/No.
4.. Added **calculated columns** for `Age Group`, `CreditScoreCategory`, and `IncomeBracket`.
5. Built **base measures** and then specific measures for **Default Rate**, **YoY**, **YTD**, and **Median**.
6. Applied a **dark theme** and consistent typography/icons; added tooltips with measure notes.
7. **Validated** DAX outputs against spot-checks in Power Query and Excel.
8. Published to **Power BI Service**; pinned key visuals to a dashboard.

---

## 7) Insights

This document summarizes the **14 key insights** derived from the Power BI loan default analysis project, combining both **visual interpretation and numeric insights**.

---

### 1. Loan Amount by Purpose

* **Highest loan purpose**: Home Improvement (**6545M**), Business (**6522M**).
* The lowest loan category is Others (**649M**).

### 2. Loan Amount by Employment Type

* **Highest Average Income**: Full-time employment (**82890**).
* Least Average Income in case of Unemployed (**82272**).

### 3. Default Rate by Employment Type

* **The highest loan default rate is observed among the unemployed (**3.39%**), making employment a key factor in credit risk.**.


### 4. Default Rate by Age Group

* **Defaults are highest among Adults (**127871**), moderate among Middle-Aged Adults, and lowest in the Teen group(**126052**)**.
.

### 5. Default Rate by Year

* Loan defaults peaked in 2016, while 2015, 2014, and 2017 recorded lower levels**.
* **A dip followed, with a slight rebound in 2018**.

### 6. Median Loan Amount by Credit Score Category

* **Borrowers with a Medium credit score received the highest median loan amounts, followed by those with Low scores.**
* **Borrowers with High credit scores typically took smaller loans.**.

### 7. Average Loan Amount by Age Group and Marital Status

* **Loan size varies across age and marital status, with married applicants in older age groups tending to borrow more than their single counterparts.**.

---

### 8. Total Loan by Credit Score (Adults)

* ** Among adults, both the Low and High credit score groups account for about **$4.8 billion** in loans, while the Medium group holds a smaller share**.


### 9. Loans for Middle-Aged Adults by Mortgage/Dependents

* **Loan volumes are higher among middle-aged adults with mortgages and multiple dependents, reflecting financial responsibilities as a driver of borrowing.**.

### 10. Number of Loans by Education Type

* Individuals with a Bachelor’s degree account for the largest share of loans (64,366 loans), while PhD holders account for the fewest (63,537 loans).**.


### 11. YOY Loan Amount Change by Year

* **2015**: biggest increase at **+1.30%**.
* **2018**: also strong with **+1.72%**.
* **2014**: sharpest decline at **-1.53%**.

### 12. YOY Loan Default Change by Year

* **2015**: defaults jumped **+2.7%**.
* **2018**: again rose by **+1.9%**.
* **2017**: steepest decline at **-2.8%**.

### 13. YTD Loan Amount by Credit Score Bins and Marital Status

* **High credit**: **0.7B**.
* **Low credit**: also **0.7B**.
* **Medium credit**: **0.3B**.
* **Very low credit**: **0.2B**.
* Spread is consistent across Single, Married, and Divorced.

### 14. Decomposition Tree (Loan Amount by Income Bracket and Employment Type)

* **High income** drives most loans (**21.7B**).
* **Medium income**: **7.2B**, **Low income**: **3.6B**.
* Within High income:

  * Full-time: **5.44B**
  * Part-time: **5.43B**
  * Self-employed: **5.42B**
  * Unemployed: **5.42B**
* Distribution is almost even across employment types once income is high.

---



