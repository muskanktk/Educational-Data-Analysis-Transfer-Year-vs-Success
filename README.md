# College Scorecard Analysis: Transfer vs. Original Students

## 📌 Project Overview

This project analyzes the **U.S. Department of Education’s College Scorecard dataset** to investigate withdrawal patterns among U.S. colleges.
The key research question is:

> **Do second-year withdrawal rates differ between transfer students and original (non-transfer) students?**

By applying data preprocessing, exploratory visualization, summary statistics, and hypothesis testing, this study evaluates whether institutional structures impact transfer students differently than their non-transfer peers.

---

## 🎯 Key Insights

* **Original students** withdraw at significantly higher rates than transfer students.
* Median withdrawal rate: \~30% for original students vs. \~5% for transfer students.
* Statistical testing (permutation test, α = 0.05) confirmed a **significant difference** (p < 0.0001).
* Patterns vary by institution type: private for-profit and nonprofit institutions show stronger disparities, while public institutions are more mixed.

---

## 🛠 Methods & Workflow

1. **Preprocessing**

   * Selected relevant variables:

     * `WDRAW_2YR_TRANS_YR2_RT` (transfer withdrawal rate)
     * `WDRAW_ORIG_YR2_RT` (original withdrawal rate)
     * `CONTROL` (institution type: public, private nonprofit, private for-profit)
   * Cleaned column names and recoded categorical variables.
   * Reshaped data to long format for comparison.

2. **Exploratory Visualization**

   * Boxplots comparing distributions.
   * Histograms of withdrawal rate distributions.
   * Scatter plots (faceted by institution type).

3. **Summary Statistics**

   * Mean withdrawal rate: **31% (original)** vs. **3.9% (transfer)**.
   * Variability is much greater among original students (wide range, high SD).

4. **Hypothesis Testing**

   * Null hypothesis: No difference between groups.
   * Test: Permutation test with 10,000 simulations.
   * Result: p < 0.0001 → reject null.

---

## 📊 Visual Examples

Boxplot comparing withdrawal rates:

![Boxplot Example](./images/boxplot_withdrawal.png)

Scatter plot by institution type:

![Scatter Example](./images/scatter_institution.png)

---

## 📂 Repository Structure

```
├── DataAnalysis.pdf      # Full final report
├── final_project.Rmd     # R Markdown source
├── README.md             # Project documentation
├── /images               # Exported graphs
```

---

## 🚀 Skills Demonstrated

* **R & Tidyverse**: `dplyr`, `ggplot2`, `tidyr`, `infer`
* **Statistical inference**: hypothesis testing, permutation methods
* **Data visualization**: boxplots, histograms, scatter plots, faceting
* **Data wrangling**: recoding categorical variables, reshaping data

---

## 📌 Conclusion

This analysis reveals that **transfer students withdraw at lower rates than original students**, challenging stereotypes about transfer student persistence. Institutions may need to provide more targeted support for first-time students, especially in public schools, to address these disparities.
