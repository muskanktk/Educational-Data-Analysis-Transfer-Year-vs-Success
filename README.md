---
title: "Educational Data Anlysis: Transfer vs Success"
author: "Muskan Khattak"
date: "`r Sys.Date()`"
documentclass: article
geometry: margin=1in
fontsize: 11pt
output:
  pdf_document:
    toc: false
    df_print: kable
    fig_caption: false
    number_sections: false
    dev: pdf
    highlight: tango
  html_document:
    theme: default
    self_contained: true
    toc: false
    df_print: kable
    fig_caption: false
    number_sections: false
    smart: true
    dev: svg
---

```{r setup, include = FALSE}
# DO NOT ALTER THIS CHUNK
# Set knitr options
knitr::opts_chunk$set(
  echo = TRUE,
  eval = TRUE,
  fig.width = 5,
  fig.asp = 0.618,
  out.width = "70%",
  dpi = 120,
  fig.align = "center",
  cache = FALSE
)
# Load required packages
suppressPackageStartupMessages(library(tidyverse))
suppressPackageStartupMessages(library(infer))
suppressPackageStartupMessages(library(modelr))
suppressPackageStartupMessages(library(broom))
# Load dataset
college <- read_rds("college.rds") %>%
  type_convert(
    na=combine("", "NA", "PrivacySuppressed")
  )
# Set seed
set.seed(98261936)
```


## **Introduction**

**Research Question:**  
Is there a statistically significant difference in second-year withdrawal rates between transfer students and non-transfer (original) students at U.S. colleges?

**Variables:**  

- *Response Variables:*  
  - `WDRAW_2YR_TRANS_YR2_RT` 
    (Percentage of second-year transfer students who withdrawal from institution)  
    
  - `WDRAW_ORIG_YR2_RT` 
   (Percentage of second-year non-transfer students who withdrawal from institution)  

- *Explanatory Variable:*  
  - `Student_Type` 
    (categorical: "Transfer" vs. "Original")  
    
**Hypotheses**

  - NULL(H~0~): The mean withdraw rates are the same for both transfer and non-transfer(original) second year students.
  
  - Second(H~1~): The mean withdrawal rates differ between original and transfer students
    
**Method:**  
Two-sided hypothesis test ( $\alpha$ = 0.05.) comparing means via permutation testing.

**Data Context:**
The analysis compares institutional withdrawal rates for transfer (WDRAW_2YR_TRANS_YR2_RT) and non-transfer (WDRAW_ORIG_YR2_RT) students using college-level data from the U.S. Department of Education. Missing values (NA) in either column will be excluded during statistical testing to prevent bias, though retained for exploratory visualizations.

**Why this question matters?**  
This analysis challenges stereotypes about transfer students' academic success. By comparing withdrawal rates, we can evaluate whether institutional structures hinder transfer students' ability to thrive. Second-year students are the focus because they most commonly transfer after completing general requirements but before entering their major coursework. The results may identify systemic barriers that institutions need to address to ensure all students have equitable opportunities for success.

# Preprocessing

```{r}

# Reprocessing Steps:
# 1. Select relevant variables for analysis
#  - withdrawal rates for both types of second-year students
#  - CONTROL variable focused on institution type
# 2. Rename columns for clarity and readability
# 3. Reshape data for group-based comparisons (One-sided t-test)
# NOTE:data containing N\A is reserved for initial analysis 

  
college_reduced <- college %>%
  # 1. Select relevant variables
  select(
    WDRAW_2YR_TRANS_YR2_RT,   # Transfer student withdrawal rate
    WDRAW_ORIG_YR2_RT,        # Original student withdrawal rate 
    CONTROL                   # Institution type (Public,Private,For-Profit)
  ) %>%
  
  # 2. Rename columns for readability 
  rename(
    transfer_withdrawal_rate = WDRAW_2YR_TRANS_YR2_RT,
    original_withdrawal_rate = WDRAW_ORIG_YR2_RT
  ) %>%
  
  # 3. Reshape to long format
  pivot_longer(
    cols = c(transfer_withdrawal_rate, original_withdrawal_rate),
    names_to = "student_type",
    values_to = "withdrawal_rate",
    values_drop_na = FALSE  # Explicitly keep NAs
  ) %>%
  
  # 4. Re-code categorical variables
  mutate(
    student_type = recode_factor(
      student_type,
      "transfer_withdrawal_rate" = "Transfer",
      "original_withdrawal_rate" = "Original"
    ),
    CONTROL = recode_factor(
      as.character(CONTROL),
      "1" = "Public",
      "2" = "Private",
      "3" = "For-Profit"
    )
  )

# Verification check-ins
college_reduced %>% count(student_type)
college_reduced %>% count(CONTROL)

# Display processed data structure
glimpse(college_reduced)
```

## Visualization

*Purpose:* This box plot compares the withdrawal rates of transfer and original second-year students. It allows us to examine the distribution of withdrawal rates within each group, identify potential outliers, and compare mean, median and variability between groups.

```{r}
# Box Plot comparing withdrawal rates for transfer vs. original students
college_reduced %>%
  #remove N/A data
  filter(!is.na(withdrawal_rate)) %>%
  
  ggplot(aes(x = student_type, y = withdrawal_rate, fill = student_type)) +
  geom_boxplot() +
  
# title, x-axis, and y-axis labels for graph
  labs(
    title = "Withdrawal Rates: Transfer vs. Original Students",
    x = "Student Type",
    y = "Withdrawal Rate"
  )  +
  
  theme_minimal()

```

The box plot shows that original students tend to have a higher median withdrawal rate (around 0.28) compared to transfer students (around 0.05). The interquartile range (spread of the middle 50% of the data) is wider for original students, suggesting more variability in their withdrawal rates. Transfer students have many outliers, indicating that while most have low withdrawal rates, some institutions have higher values.

*Purpose:* This histogram shows the distribution of withdrawal rates for both transfer and original students. It allows us to examine the overall shape of each group's data, identify skewness, and observe how much the two distributions overlap. This helps us visually compare the variability between the two student types.

```{r}
# Histogram of withdraw rate for both groups among institutions

college_reduced %>%
  
  #remove N/A data
  filter(!is.na(withdrawal_rate)) %>%
  ggplot(aes(x = withdrawal_rate, fill = student_type)) +
  
  geom_histogram(alpha = 0.5, bins = 25, position = "identity") +
  
  # title, x-axis, and y-axis labels for graph
  labs(
    title = "Withdrawal Rates Distribution: Transfer vs. Original Students",
    x = "Withdrawal Rate",
    y = "Count"
  ) +
  
  theme_minimal()

```
The histogram shows that original students have a roughly bi-modal distribution, with noticeable peaks around 0.2 and 0.5. This suggests variability among institutions in how often original students withdraw. In contrast, transfer students show a right-skewed distribution, with most institutions having low withdrawal rates and a long tail toward higher values. The two groups overlap between 0.0 and 0.2, indicating that some institutions have similar withdrawal rates for both groups, but overall, the distributions differ.

*Purpose:* This scatter plot visualizes the relationship between original student withdrawal rates and transfer student withdrawal rates across various institutions. By examining these rates, we explore whether institutional factors influence withdrawal patterns for each group. The plot is faceted by institution type—public, private nonprofit, and private for-profit—to investigate how different educational environments may affect withdrawal rates for both original and transfer students.


```{r}
# Scatter plot that displays the withdrawal rate for transfer and original students for the same institution
college_wide <- college %>%
  select(
    transfer_withdrawal = WDRAW_2YR_TRANS_YR2_RT,
    original_withdrawal = WDRAW_ORIG_YR2_RT,
    CONTROL #control group implementation
    
  ) %>%
  filter(!is.na(transfer_withdrawal) & !is.na(original_withdrawal)) %>%
  distinct()

ggplot(college_wide, aes(x = original_withdrawal, y = transfer_withdrawal)) +
  geom_point(alpha = 0.5) +
  geom_abline(slope = 1, intercept = 0, color = "blue", linetype = "dashed") +
  
  #Different types of institutional environments 
  facet_wrap(~ CONTROL, labeller = labeller(CONTROL = 
        c("1" = "Public", 
          "2" = "Private Nonprofit", 
          "3" = "Private For-Profit"))) +
  labs(
    title = "Withdrawal Rates: Original vs Transfer Students",
    x = "Original Student Withdrawal Rate (%)",
    y = "Transfer Student Withdrawal Rate (%)",
    caption = "Point=institution"
  )  +
  
  theme_minimal()

```
The scatter plot displays withdrawal rates for each institution, with each point representing one school. Most points fall below the blue reference line, indicating that, at the majority of institutions, original students have higher withdrawal rates than transfer students. Points above the line represent institutions where transfer students withdraw at higher rates.

The data reveals a weak positive trend: as original student withdrawal rates increase, transfer student withdrawal rates tend to rise slightly as well. This suggests that institutional factors may play a role in overall student withdrawal patterns.

When examining the faceted plots by institution type, we observe that original students are more likely to drop out across both private nonprofit and for-profit institutions. However, in public institutions, the trend is not as neat, with some institutions having higher transfer student.

## Summary Statistics
```{r}

# Statistics for withdrawal rates 
college_reduced %>%
  filter(!is.na(withdrawal_rate)) %>%
  
  group_by(student_type) %>%
  summarize(
    count = n(),
    mean = mean(withdrawal_rate),
    median = median(withdrawal_rate),
    sd = sd(withdrawal_rate),
    range = max(withdrawal_rate) - min(withdrawal_rate),
    IQR = IQR(withdrawal_rate)
  )
```

The mean withdrawal rate for original students is 0.31, compared to 0.03 for transfer students, indicating original students are eight times more likely to withdraw. The median values (0.28 vs. 0.035) reinforce this consistent pattern, regardless of outliers. Original students also show greater variability, with a higher standard deviation (0.155 vs. 0.0121) and a wider range (0.723 vs. 0.287).

The high withdrawal rates among original students may reflect academic challenges or institutional barriers, while transfer students—who, due to their prior experience or selective enrollment, might adapt more effectively—show lower rates. The wide IQR for original students suggests inconsistent challenges across institutions, whereas the narrow IQR for transfer students points to more uniform institutional experiences.

## Data Analysis

 NULL(H~0~): The mean withdraw rates are the same for both transfer and non-transfer(original) second year students.
  
 Second(H~1~): The mean withdrawal rates differ between original and transfer students

*I am conducting a 2-sided hypothesis test.*

Test statistic: Difference in means of original and transfer students.

```{r}
# remove data with N/A values
college_testing <- college_reduced %>%
  filter(!is.na(withdrawal_rate))

# calculating the difference in mean for transfer and original students
obs_stats <- college_testing %>%
  specify(withdrawal_rate ~ student_type) %>%
  calculate(stat = "diff in means", order = c("Original", "Transfer"))

# to test null use permutation of 10000
null <- college_testing %>%
  specify(withdrawal_rate ~ student_type) %>%
  hypothesize(null = "independence") %>%
  generate(reps = 10000, type = "permute") %>%
  calculate(stat = "diff in means", order = c("Original", "Transfer"))

# calculate the p value
#NOTE: The p value is not 0, but instead it is very small
p_value <- null %>% 
  get_p_value(obs_stat = obs_stats, direction = "two-sided")

# visualize the null distribution and p-value
null %>%
  visualize() +
  shade_p_value(obs_stat = obs_stats, direction = "two-sided")

# display the statistics and p value
print(obs_stats)
print(p_value)


```
*Results:*

  - Observed difference in means: 0.273295	

  - p-value: 0.0001 < $\alpha$: 0.05

Since the p-value (< 0.0001) is much smaller than the significance level ($\alpha$ = 0.05), we reject the null hypothesis. There is statistical evidence that withdrawal rates differ between original and transfer students.

The difference (0.273) suggests that original students withdraw at a higher rate than transfer students in their second year.

The null distribution plot displays simulation under the assumption that student type (original vs. transfer) has no effect (null hypothesis is true). The distribution is clustered around 0.0, indicating that random permutations produce almost zero differences. The observed difference (0.273, marked by red line) at the the right of this distribution, where no permuted differences occur. This visual supports the rejection of null hypothesis.

## Conclusion

This study examined whether second-year withdrawal rates differ between original and transfer students. Visualizations and summary statistics revealed clear trends: original students exhibited higher withdrawal rates (median = 30%, wide IQR of 20–45%) with variability across institutions, while transfer students had consistently lower rates (median = 5%, tight IQR of 2–5%), suggesting consistency. The hypothesis test confirmed these observations (p < 0.0001), rejecting the null hypothesis that withdrawal rates are equal. This statistical significance, combined with the histograms and box plots, highlights that original students withdraw more, evidently in public institutions.

Potential explanations for this disparity include original students facing adjustment challenges (e.g., "sophomore slump," financial stress), whereas transfer students may benefit from prior college experience or more careful institution selection. However, the data cannot assess individual student experiences (only institutional averages), and unmeasured factors (e.g., age, socioeconomic status) may influence results. For institutions, targeted support (e.g., academic advising, peer mentoring) for first-time students could decrease withdrawals, especially in public schools. This analysis highlights systemic differences in student continuation of their studies, urging research into underlying causes and potential solutions.
