# Predicting-Dropout-Risk-among-JHS-and-SHS-Students-in-Ghana-with-R

## Project Overview

This repository contains an R Markdown analysis of the **Ghana Education and School Survey (GESS)** dataset. The project investigates student, household, and school characteristics associated with dropout risk and core-subject performance among Junior High School (JHS) and Senior High School (SHS) students in Ghana.

The analysis progresses from data preparation and exploratory analysis to inferential testing, multiple linear regression, and supervised machine-learning models.

> **Primary outcome:** `Dropout_Risk_Score` — a continuous dropout-risk score.  
> **Secondary outcome:** `Passed_Core` — whether a student passed the core subjects.

The analysis deliberately avoids target leakage. In particular, `Passed_Core` is defined from `Overall_Average >= 50`, so `Overall_Average`, the individual subject scores, and related derived performance variables are not used as predictors of `Passed_Core`.

---

## Objectives

The project aims to:

1. Import and inspect the GESS student and school datasets.
2. Join student and school records using `School_ID`.
3. Audit and treat missing values using documented rules.
4. Prepare a clean analysis dataset through filtering, selecting, mutating, reshaping, and factor ordering.
5. Explore dropout risk through descriptive statistics and visualisations.
6. Perform inferential tests to examine differences and associations.
7. Build and interpret a multiple linear regression model for dropout risk.
8. Develop and compare supervised classification models for `Passed_Core`.
9. Evaluate model performance using held-out test data.
10. Identify important predictors while considering the limitations of observational data.

---

## Data

The analysis uses two CSV files:

- `gess_student.csv` — student-level records.
- `gess_school.csv` — school-level records.

The files are joined using:

```text
School_ID
```

The student file is used as the left-hand table because the unit of analysis is the student. The analysis checks that `School_ID` is unique in the school dataset and that every student has a corresponding school record.

The analysis also checks conflicting `Region` fields after the join. The school-level region is treated as the authoritative geographic field for regional summaries and mapping.

---

## Analytical Workflow

### 1. Data Import and Inspection

The project begins by importing the two CSV files with `read_csv()` and inspecting their structure using `glimpse()`.

The analysis verifies:

- Number of observations and variables.
- Variable types.
- Student and school identifiers.
- The relationship between `Passed_Core` and `Overall_Average`.
- Uniqueness of `School_ID`.
- Whether all students have matching school records.

### 2. Missing-Value Treatment

Missing values are audited before modelling.

The project uses different treatment rules depending on the variable:

| Variable type | Treatment |
|---|---|
| `Dropout_Risk_Score` | Listwise deletion |
| Continuous student predictors | Median imputation within `Form_Level` |
| Categorical student predictors | Mode imputation |
| School-level continuous variables | Median imputation within `School_Level` |
| Subject scores used for visualisation | Retained as `NA` |
| School examination pass rates | Retained as `NA` |

The primary outcome is not imputed because creating artificial dropout-risk scores could distort the regression analysis.

### 3. Data Wrangling

The final analysis dataset, `dat_clean`, is created from the cleaned student and school data.

Key transformations include:

- Extracting a numeric identifier from `Student_ID`.
- Simplifying household-income labels.
- Removing students with missing `Dropout_Risk_Score`.
- Selecting variables relevant to the analytical questions.
- Creating `Attendance_Gap`.
- Creating the categorical `Risk_Level` variable.
- Ordering categorical variables and defining regression reference categories.
- Reshaping subject scores into the long-format `subject_scores_long` dataset.

---

## Exploratory Data Analysis

Five main visual analyses are included.

### Plot 1 — Dropout-Risk Distribution

A histogram and density curve are used to examine the distribution of `Dropout_Risk_Score`, including its mean and median.

### Plot 2 — Dropout Risk by Household Income

Boxplots compare dropout-risk scores across Low, Mid, and High household-income groups.

### Plot 3 — Attendance and Teacher Support

A scatterplot examines the relationship between `Attendance_Rate_Pct` and `Teacher_Support_Score`, with separate fitted lines by `School_Level`.

### Plot 4 — Subject Score Distributions

A ridge plot compares English, Mathematics, Science, and Social Studies score distributions by `Passed_Core`.

### Plot 5 — Geographic Analysis

Two geographic views are provided:

- A static regional choropleth showing mean dropout risk across Ghana.
- An interactive `leaflet` map showing individual students by dropout-risk level.

---

## Inferential Statistics

Three inferential tests are performed.

### Independent-Samples t-Test

The t-test compares mean `Dropout_Risk_Score` between:

- Day students
- Boarding students

Assumption checks include:

- Shapiro-Wilk normality testing.
- Q-Q plots.
- Levene's test.
- Wilcoxon rank-sum robustness testing.

Cohen's *d* is reported as the effect size.

### One-Way ANOVA

A one-way ANOVA compares `Dropout_Risk_Score` across:

- Low household income
- Mid household income
- High household income

The analysis includes:

- Bartlett's test.
- Levene's test.
- Residual Q-Q plot.
- Tukey HSD pairwise comparisons.
- Kruskal-Wallis robustness check.
- Eta-squared effect size.

### Chi-Square Test

A Pearson chi-square test evaluates the association between:

- `Passed_Core`
- `HH_Income_Band`

The analysis reports:

- Observed counts.
- Expected counts.
- Chi-square statistic.
- Cramér's V.
- Minimum expected count.

The findings show statistically significant associations, but the reported effect sizes are small. Statistical significance is therefore interpreted separately from practical effect size.

---

## Regression Analysis

Because `Dropout_Risk_Score` is continuous, the project uses **multiple linear regression**.

The initial model includes:

- `Attendance_Rate_Pct`
- `Study_Hours_Day`
- `Teacher_Support_Score`
- `Resource_Index`
- `Pupil_Teacher_Ratio`
- `HH_Income_Band`
- `Residence_Type`
- `School_Level`
- `Sex`

Stepwise model selection is performed using `MASS::stepAIC()`.

### Regression Diagnostics

The final model is assessed using:

- Residuals vs fitted values.
- Normal Q-Q plot.
- Scale-location plot.
- Residuals vs leverage.
- Variance Inflation Factors (VIF).
- Formal residual tests.

The analysis reports that the LINE assumptions are sufficiently supported for the assignment's OLS model, with no material multicollinearity detected.

### Main Regression Findings

The final model explains approximately **12.7% of the variance** in dropout risk.

The strongest reported associations include:

- Higher attendance is associated with lower dropout risk.
- Students from Mid- and High-income households have lower predicted dropout risk than students from Low-income households.
- Boarding students have lower predicted dropout risk than Day students.

The analysis also notes that most variation in dropout risk remains unexplained and that students are nested within schools. A multilevel model is identified as a possible future extension.

---

## Predictive Analytics

The predictive component treats `Passed_Core` as the classification target.

Two supervised machine-learning models are compared:

1. **Decision Tree**
2. **Random Forest**

The models are implemented using the `tidymodels` framework.

### Leakage Prevention

The predictive models exclude:

- `Overall_Average`
- `Perf_Band`
- `Dropout_Risk_Score`
- `Risk_Level`
- `Attendance_Gap`
- Individual subject scores as predictors

These exclusions prevent the models from using information that directly or indirectly defines the target.

### Train/Test Strategy

The data is divided into:

- **70% training**
- **30% testing**

The split is stratified by `Passed_Core`.

The training data is evaluated using **10-fold stratified cross-validation**.

The held-out test set is evaluated only after model tuning is complete.

### Model Tuning

The decision tree tunes:

- `cost_complexity`
- `tree_depth`

The random forest tunes:

- `mtry`
- `min_n`

The random forest uses 500 trees.

Model selection is based primarily on **AUC-ROC**, rather than accuracy alone, because accuracy can be misleading when the target classes are imbalanced.

### Evaluation Metrics

The models are compared using:

- Accuracy
- AUC-ROC
- Sensitivity
- Specificity
- Confusion matrices

A majority-class baseline is also reported for comparison.

### Predictive Findings

The held-out results indicate limited predictive signal in the available pre-results variables. The models perform better at identifying students who passed than students who did not pass, and their AUC values are only modestly above 0.50.

The project therefore treats the predictive results as a useful negative finding: the available variables appear more useful for describing group differences and associations than for accurately classifying individual failures.

The random forest is preferred between the two models based on ranking performance, but the analysis does **not** recommend using either model for high-stakes individual decisions in its current form.

---

## Key Findings

The analysis identifies several consistent patterns:

- Dropout risk varies across household-income groups.
- Day students have higher average dropout risk than boarding students.
- Lower household income is associated with higher dropout risk and a higher share of students who do not pass core subjects.
- Attendance is one of the clearest predictors of lower dropout risk in the regression model.
- Higher household income is associated with lower predicted dropout risk after controlling for other variables.
- School-level resource variables contribute relatively little after other predictors are considered.
- The predictive models have limited ability to identify individual students who will not pass using only the available pre-results variables.
- Statistical significance should not be interpreted as evidence of a large practical effect.
- Because students are clustered within schools, a multilevel modelling approach could provide a stronger treatment of school-level dependence.

---

## Technologies and R Packages

The analysis uses R and the following packages:

### Core analysis

- `tidyverse`
- `knitr`
- `scales`

### Visualisation

- `ggplot2`
- `ggridges`
- `sf`
- `leaflet`
- `geodata`

### Tables and reporting

- `flextable`
- `gtsummary`
- `broom`

### Statistical analysis

- `effectsize`
- `car`
- `MASS`

### Predictive modelling

- `tidymodels`
- `themis`
- `vip`

### Plot composition

- `patchwork`

---

## Reproducibility

The analysis sets a random seed:

```r
set.seed(602)
```

The same seed is used for sampling and model evaluation where required.

The R Markdown document is designed to produce PDF and Word outputs and contains executable R code, tables, plots, statistical tests, regression results, and machine-learning evaluation.

---

## How to Run the Project

### 1. Install R

Install R from the official R Project website.

### 2. Install RStudio

RStudio or another compatible R development environment can be used to open and run the `.Rmd` file.

### 3. Place the data files in the project directory

The following files should be available in the same working directory as the R Markdown file:

```text
gess_student.csv
gess_school.csv
Inferential Tests_Regression Analysis_Prediction_Analytics.Rmd
```

### 4. Install required packages

You can install the main packages with:

```r
install.packages(c(
  "tidyverse",
  "knitr",
  "scales",
  "ggridges",
  "sf",
  "flextable",
  "leaflet",
  "geodata",
  "gtsummary",
  "broom",
  "effectsize",
  "car",
  "MASS",
  "patchwork",
  "tidymodels",
  "themis",
  "vip"
))
```

### 5. Open the R Markdown file

Open:

```text
Inferential Tests_Regression Analysis_Prediction_Analytics.Rmd
```

### 6. Run or Knit the document

The document can be executed interactively using RStudio or knitted into the configured output formats.

The geospatial component requires access to GADM geographic data through the `geodata` package.

---

## Repository Structure

A recommended GitHub repository structure is:

```text
gess-dropout-risk-analysis/
│
├── README.md
├── Inferential Tests_Regression Analysis_Prediction_Analytics.Rmd
│
├── data/
│   ├── gess_student.csv
│   └── gess_school.csv
│
├── output/
│   ├── figures/
│   └── tables/
│
└── .gitignore
```

If the GESS data is restricted or contains sensitive information, it should **not** be committed to a public GitHub repository. Instead, document how authorized users can obtain the data and reproduce the analysis.

---

## Reproducibility and Responsible Use

This project is an analytical and academic exercise based on observational survey data.

The results describe associations within the GESS sample and should not be interpreted as causal effects.

The predictive models should not be used for high-stakes decisions about individual students without further validation, stronger longitudinal data, appropriate fairness assessment, and a more rigorous modelling framework.

Students are nested within schools, which creates a potential dependence structure that ordinary linear regression does not fully address. A multilevel model would be an appropriate extension.

---

## AI Assistance Disclosure

The original analysis reports that portions of the R code were produced with assistance from an AI coding tool. The code was run against the GESS files, checked against the resulting output, and edited by the project group. The analytical decisions, including the join-key resolution and missingness treatment rules, were made by the group.

---

## Project Scope

This repository demonstrates an end-to-end data-science workflow covering:

```text
Raw Data
   │
   ▼
Data Import & Inspection
   │
   ▼
Data Joining
   │
   ▼
Missing-Value Audit & Treatment
   │
   ▼
Data Wrangling
   │
   ▼
Exploratory Data Analysis
   │
   ├── Descriptive Statistics
   ├── Visualisation
   └── Geographic Analysis
   │
   ▼
Inferential Testing
   │
   ├── t-Test
   ├── ANOVA
   └── Chi-Square
   │
   ▼
Multiple Linear Regression
   │
   ▼
Predictive Analytics
   │
   ├── Decision Tree
   └── Random Forest
   │
   ▼
Model Evaluation & Interpretation
```

---

## Conclusion

The project provides a complete analytical workflow for investigating dropout risk and core-subject outcomes among JHS and SHS students in Ghana.

The results consistently point to attendance, household socioeconomic background, and residence type as important correlates of dropout risk. However, the relatively modest regression explanatory power and weak classification performance show that the available variables are not sufficient for highly accurate individual-level prediction.

Future work could incorporate longitudinal student records, term-by-term attendance and academic performance, teacher referrals, and multilevel modelling to better account for school-level clustering and improve predictive performance.
