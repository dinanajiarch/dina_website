---
title: 'Lab 3: Moderation'
author: 'TA: Dina Arch'
date: 2021-04-16
subtitle: Overview of regression using categorical and continuous moderators with interpretation.
weight: 3
draft: false
images: 
series: 
tags:
- r markdown
- tidy data
- regression
- moderation
categories:
- data analysis
editor_options:
  markdown:
    wrap: sentence
---

``` r
library(knitr) #include_graphics()
library(equatiomatic) # extract_eq()
library(psych) #describe()
library(gtsummary) #tbl_summary()
library(summarytools) #descr()
library(stargazer) #stargazer()
library(sjPlot) #tab_model()
library(interactions) #interact_plot(), sim_slopes()
library(jtools) #summ()
library(tidyverse)
```

# Moderation

When the research hypotheses state that different categories, or levels of another variable, may have differing responses to other independent variables, we need to use interaction terms

\- Also called **moderation**

![](images/moderation.png)

**Example**: The relationship between *discrimination* and *grades* depends on prog.

Graph drawn using [draw.io](draw.io).

------------------------------------------------------------------------

## Moderation Example

Suppose you are doing a simple study on weight loss and notice that people who spend more time exercising lose more weight.
Upon further analysis you notice that those who spend the same amount of time exercising lose more weight if they are more effortful.
The more effort people put into their workouts, the less time they need to spend exercising.
This is popular in workouts like high intensity interval training (HIIT).

Example adapted from UCLA’s [seminar](https://stats.idre.ucla.edu/r/seminars/interactions-r/#s4) on decomposing, probing, and plotting interactions in R.

Read in data:

``` r
exercise <- read.csv("https://stats.idre.ucla.edu/wp-content/uploads/2019/03/exercise.csv") %>% 
  select(-gender) %>% 
  mutate(prog = factor(prog, labels = c("Jogging", "Swimming", "Reading")),
         hours_scaled = as.numeric(scale(hours)))
```

------------------------------------------------------------------------

### Variable Descriptions

|          |                                                                                          |
|----------|------------------------------------------------------------------------------------------|
| **Name** | **Labels**                                                                               |
| prog     |                                                                                          |
| loss     | Weight loss (continuous), positive = weight loss, negative scores = weight gain          |
| hours    | Hours spent exercising (continuous)                                                      |
| effort   | Effort during exercise (continuous), 0 = minimal physical effort and 50 = maximum effort |

------------------------------------------------------------------------

### Descriptive Statistics

``` r
#Three ways of presenting descriptives:

# psych::describe()
round(describe(exercise),2) 
```

    ##              vars   n   mean     sd median trimmed    mad    min    max  range
    ## id              1 900 450.50 259.95 450.50  450.50 333.58   1.00 900.00 899.00
    ## loss            2 900  10.02  14.10   7.88    9.06  15.42 -17.14  54.15  71.29
    ## hours           3 900   2.00   0.49   2.01    2.00   0.49   0.18   4.07   3.90
    ## effort          4 900  29.66   5.14  29.63   29.64   5.08  12.95  44.08  31.13
    ## prog*           5 900   2.00   0.82   2.00    2.00   1.48   1.00   3.00   2.00
    ## hours_scaled    6 900   0.00   1.00   0.01    0.00   0.99  -3.70   4.19   7.88
    ##              skew kurtosis   se
    ## id           0.00    -1.20 8.67
    ## loss         0.50    -0.64 0.47
    ## hours        0.07     0.47 0.02
    ## effort       0.00     0.01 0.17
    ## prog*        0.00    -1.50 0.03
    ## hours_scaled 0.07     0.47 0.03

``` r
# gtsummary::gtsummary()
tbl_summary(exercise,
            statistic = list(all_continuous() ~ "{mean} ({sd})"),
                             missing = "no")
```

<div id="sghigmuysv" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#sghigmuysv .gt_table {
  display: table;
  border-collapse: collapse;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 16px;
  font-weight: normal;
  font-style: normal;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#sghigmuysv .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#sghigmuysv .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#sghigmuysv .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 0;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#sghigmuysv .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#sghigmuysv .gt_col_headings {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#sghigmuysv .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#sghigmuysv .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#sghigmuysv .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#sghigmuysv .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#sghigmuysv .gt_column_spanner {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 5px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#sghigmuysv .gt_group_heading {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
}

#sghigmuysv .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#sghigmuysv .gt_from_md > :first-child {
  margin-top: 0;
}

#sghigmuysv .gt_from_md > :last-child {
  margin-bottom: 0;
}

#sghigmuysv .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: solid;
  border-top-width: 1px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#sghigmuysv .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
}

#sghigmuysv .gt_stub_row_group {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
  vertical-align: top;
}

#sghigmuysv .gt_row_group_first td {
  border-top-width: 2px;
}

#sghigmuysv .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#sghigmuysv .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#sghigmuysv .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#sghigmuysv .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#sghigmuysv .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#sghigmuysv .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#sghigmuysv .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#sghigmuysv .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#sghigmuysv .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#sghigmuysv .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-left: 4px;
  padding-right: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#sghigmuysv .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#sghigmuysv .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#sghigmuysv .gt_left {
  text-align: left;
}

#sghigmuysv .gt_center {
  text-align: center;
}

#sghigmuysv .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#sghigmuysv .gt_font_normal {
  font-weight: normal;
}

#sghigmuysv .gt_font_bold {
  font-weight: bold;
}

#sghigmuysv .gt_font_italic {
  font-style: italic;
}

#sghigmuysv .gt_super {
  font-size: 65%;
}

#sghigmuysv .gt_two_val_uncert {
  display: inline-block;
  line-height: 1em;
  text-align: right;
  font-size: 60%;
  vertical-align: -0.25em;
  margin-left: 0.1em;
}

#sghigmuysv .gt_footnote_marks {
  font-style: italic;
  font-weight: normal;
  font-size: 75%;
  vertical-align: 0.4em;
}

#sghigmuysv .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#sghigmuysv .gt_slash_mark {
  font-size: 0.7em;
  line-height: 0.7em;
  vertical-align: 0.15em;
}

#sghigmuysv .gt_fraction_numerator {
  font-size: 0.6em;
  line-height: 0.6em;
  vertical-align: 0.45em;
}

#sghigmuysv .gt_fraction_denominator {
  font-size: 0.6em;
  line-height: 0.6em;
  vertical-align: -0.05em;
}
</style>
<table class="gt_table">
  
  <thead class="gt_col_headings">
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1"><strong>Characteristic</strong></th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1"><strong>N = 900</strong><sup class="gt_footnote_marks">1</sup></th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr><td class="gt_row gt_left">id</td>
<td class="gt_row gt_center">450 (260)</td></tr>
    <tr><td class="gt_row gt_left">loss</td>
<td class="gt_row gt_center">10 (14)</td></tr>
    <tr><td class="gt_row gt_left">hours</td>
<td class="gt_row gt_center">2.00 (0.49)</td></tr>
    <tr><td class="gt_row gt_left">effort</td>
<td class="gt_row gt_center">29.7 (5.1)</td></tr>
    <tr><td class="gt_row gt_left">prog</td>
<td class="gt_row gt_center"></td></tr>
    <tr><td class="gt_row gt_left" style="text-align: left; text-indent: 10px;">Jogging</td>
<td class="gt_row gt_center">300 (33%)</td></tr>
    <tr><td class="gt_row gt_left" style="text-align: left; text-indent: 10px;">Swimming</td>
<td class="gt_row gt_center">300 (33%)</td></tr>
    <tr><td class="gt_row gt_left" style="text-align: left; text-indent: 10px;">Reading</td>
<td class="gt_row gt_center">300 (33%)</td></tr>
    <tr><td class="gt_row gt_left">hours_scaled</td>
<td class="gt_row gt_center">0.00 (1.00)</td></tr>
  </tbody>
  
  <tfoot class="gt_footnotes">
    <tr>
      <td class="gt_footnote" colspan="2"><sup class="gt_footnote_marks">1</sup> Mean (SD); n (%)</td>
    </tr>
  </tfoot>
</table>
</div>

``` r
# summarytools::descr()
#print(descr(exercise), method = 'render')
```

------------------------------------------------------------------------

## Regression with a Categorical Moderator

**Research Question:** Does *exercise program* moderate the relationship between *hours spent exercising* and *weight loss*?
Or, said differently, Does the relationship between the *hours spent exercising* and *weight loss* differ based on the type of exercise you do (e.g., *exercise program*)?

``` r
reg <- lm(loss ~ scale(hours)+prog, 
                  data=exercise)
mod <- lm(loss ~ scale(hours)*prog, 
                  data=exercise)

#Compare regression and moderation models
tab_model(reg, mod,show.std = TRUE, show.stat = TRUE, show.zeroinf = TRUE )
```

<table style="border-collapse:collapse; border:none;">
<tr>
<th style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm;  text-align:left; ">
 
</th>
<th colspan="6" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; ">
loss
</th>
<th colspan="6" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; ">
loss
</th>
</tr>
<tr>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  text-align:left; ">
Predictors
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">
Estimates
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">
std. Beta
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">
CI
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">
standardized CI
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">
Statistic
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  col7">
p
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  col8">
Estimates
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  col9">
std. Beta
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  0">
CI
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  1">
standardized CI
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  2">
Statistic
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  3">
p
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
(Intercept)
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
8.08
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.14
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
7.29 – 8.87
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.19 – -0.08
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
20.17
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col7">
<strong>\<0.001</strong>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col8">
8.14
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col9">
-0.13
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  0">
7.41 – 8.88
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  1">
-0.19 – -0.08
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  2">
21.68
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  3">
<strong>\<0.001</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
hours
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
1.62
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.11
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
1.16 – 2.07
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.08 – 0.15
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
6.98
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col7">
<strong>\<0.001</strong>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col8">
3.69
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col9">
0.26
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  0">
2.90 – 4.47
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  1">
0.21 – 0.32
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  2">
9.25
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  3">
<strong>\<0.001</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
prog \[Swimming\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
17.78
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
1.26
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
16.67 – 18.90
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
1.18 – 1.34
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
31.40
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col7">
<strong>\<0.001</strong>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col8">
17.77
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col9">
1.26
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  0">
16.73 – 18.81
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  1">
1.19 – 1.33
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  2">
33.46
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  3">
<strong>\<0.001</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
prog \[Reading\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-11.96
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.85
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-13.07 – -10.85
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.93 – -0.77
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-21.10
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col7">
<strong>\<0.001</strong>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col8">
-11.85
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col9">
-0.84
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  0">
-12.89 – -10.80
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  1">
-0.91 – -0.77
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  2">
-22.29
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  3">
<strong>\<0.001</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
hours \* prog \[Swimming\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col7">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col8">
-0.29
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col9">
-0.02
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  0">
-1.37 – 0.80
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  1">
-0.10 – 0.06
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  2">
-0.52
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  3">
0.605
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
hours \* prog \[Reading\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col7">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col8">
-5.15
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col9">
-0.37
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  0">
-6.19 – -4.11
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  1">
-0.44 – -0.29
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  2">
-9.71
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  3">
<strong>\<0.001</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm; border-top:1px solid;">
Observations
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="6">
900
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="6">
900
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">
R<sup>2</sup> / R<sup>2</sup> adjusted
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="6">
0.759 / 0.758
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="6">
0.789 / 0.787
</td>
</tr>
</table>

``` r
#Can also use stargazer::stargazer()
stargazer(reg, mod, type = "text")
```

    ## 
    ## ===========================================================================
    ##                                          Dependent variable:               
    ##                           -------------------------------------------------
    ##                                                 loss                       
    ##                                     (1)                      (2)           
    ## ---------------------------------------------------------------------------
    ## scale(hours)                      1.616***                 3.686***        
    ##                                   (0.232)                  (0.398)         
    ##                                                                            
    ## progSwimming                     17.785***                17.771***        
    ##                                   (0.566)                  (0.531)         
    ##                                                                            
    ## progReading                      -11.961***               -11.846***       
    ##                                   (0.567)                  (0.531)         
    ##                                                                            
    ## scale(hours):progSwimming                                   -0.286         
    ##                                                            (0.554)         
    ##                                                                            
    ## scale(hours):progReading                                  -5.148***        
    ##                                                            (0.530)         
    ##                                                                            
    ## Constant                          8.080***                 8.143***        
    ##                                   (0.401)                  (0.376)         
    ##                                                                            
    ## ---------------------------------------------------------------------------
    ## Observations                        900                      900           
    ## R2                                 0.759                    0.789          
    ## Adjusted R2                        0.758                    0.787          
    ## Residual Std. Error           6.937 (df = 896)         6.502 (df = 894)    
    ## F Statistic               939.491*** (df = 3; 896) 666.752*** (df = 5; 894)
    ## ===========================================================================
    ## Note:                                           *p<0.1; **p<0.05; ***p<0.01

``` r
#Only looks at moderation 
# and jtools:summ()
summ(mod)
```

<table class="table table-striped table-hover table-condensed table-responsive" style="width: auto !important; margin-left: auto; margin-right: auto;">
<tbody>
<tr>
<td style="text-align:left;font-weight: bold;">
Observations
</td>
<td style="text-align:right;">
900
</td>
</tr>
<tr>
<td style="text-align:left;font-weight: bold;">
Dependent variable
</td>
<td style="text-align:right;">
loss
</td>
</tr>
<tr>
<td style="text-align:left;font-weight: bold;">
Type
</td>
<td style="text-align:right;">
OLS linear regression
</td>
</tr>
</tbody>
</table>
<table class="table table-striped table-hover table-condensed table-responsive" style="width: auto !important; margin-left: auto; margin-right: auto;">
<tbody>
<tr>
<td style="text-align:left;font-weight: bold;">
F(5,894)
</td>
<td style="text-align:right;">
666.75
</td>
</tr>
<tr>
<td style="text-align:left;font-weight: bold;">
R²
</td>
<td style="text-align:right;">
0.79
</td>
</tr>
<tr>
<td style="text-align:left;font-weight: bold;">
Adj. R²
</td>
<td style="text-align:right;">
0.79
</td>
</tr>
</tbody>
</table>
<table class="table table-striped table-hover table-condensed table-responsive" style="width: auto !important; margin-left: auto; margin-right: auto;border-bottom: 0;">
<thead>
<tr>
<th style="text-align:left;">
</th>
<th style="text-align:right;">
Est.
</th>
<th style="text-align:right;">
S.E.
</th>
<th style="text-align:right;">
t val.
</th>
<th style="text-align:right;">
p
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;font-weight: bold;">
(Intercept)
</td>
<td style="text-align:right;">
8.14
</td>
<td style="text-align:right;">
0.38
</td>
<td style="text-align:right;">
21.68
</td>
<td style="text-align:right;">
0.00
</td>
</tr>
<tr>
<td style="text-align:left;font-weight: bold;">
scale(hours)
</td>
<td style="text-align:right;">
3.69
</td>
<td style="text-align:right;">
0.40
</td>
<td style="text-align:right;">
9.25
</td>
<td style="text-align:right;">
0.00
</td>
</tr>
<tr>
<td style="text-align:left;font-weight: bold;">
progSwimming
</td>
<td style="text-align:right;">
17.77
</td>
<td style="text-align:right;">
0.53
</td>
<td style="text-align:right;">
33.46
</td>
<td style="text-align:right;">
0.00
</td>
</tr>
<tr>
<td style="text-align:left;font-weight: bold;">
progReading
</td>
<td style="text-align:right;">
-11.85
</td>
<td style="text-align:right;">
0.53
</td>
<td style="text-align:right;">
-22.29
</td>
<td style="text-align:right;">
0.00
</td>
</tr>
<tr>
<td style="text-align:left;font-weight: bold;">
scale(hours):progSwimming
</td>
<td style="text-align:right;">
-0.29
</td>
<td style="text-align:right;">
0.55
</td>
<td style="text-align:right;">
-0.52
</td>
<td style="text-align:right;">
0.61
</td>
</tr>
<tr>
<td style="text-align:left;font-weight: bold;">
scale(hours):progReading
</td>
<td style="text-align:right;">
-5.15
</td>
<td style="text-align:right;">
0.53
</td>
<td style="text-align:right;">
-9.71
</td>
<td style="text-align:right;">
0.00
</td>
</tr>
</tbody>
<tfoot>
<tr>
<td style="padding: 0; " colspan="100%">
<sup></sup> Standard errors: OLS
</td>
</tr>
</tfoot>
</table>

### Simple Slope Equations

The results above tell us that there is a moderation effect because the p-value for scale(hours)\*program(reading) is significant.
The estimate of -5.15 is not that useful on it’s own.
The best way to understand is to use the estimates above and create a simple slope equation for each of the exercise programs (reading, swimming, and jogging).
We do this below.

``` r
extract_eq(mod)
```

$$
\operatorname{loss} = \alpha + \beta_{1}(\operatorname{scale(hours)}) + \beta_{2}(\operatorname{prog}_{\operatorname{Swimming}}) + \beta_{3}(\operatorname{prog}_{\operatorname{Reading}}) + \beta_{4}(\operatorname{scale(hours)} \times \operatorname{prog}_{\operatorname{Swimming}}) + \beta_{5}(\operatorname{scale(hours)} \times \operatorname{prog}_{\operatorname{Reading}}) + \epsilon
$$

``` r
extract_eq(mod, use_coefs = TRUE)
```

$$
\operatorname{\widehat{loss}} = 8.14 + 3.69(\operatorname{scale(hours)}) + 17.77(\operatorname{prog}_{\operatorname{Swimming}}) - 11.85(\operatorname{prog}_{\operatorname{Reading}}) - 0.29(\operatorname{scale(hours)} \times \operatorname{prog}_{\operatorname{Swimming}}) - 5.15(\operatorname{scale(hours)} \times \operatorname{prog}_{\operatorname{Reading}})
$$

Reading:

$$
y_{reading} = 8.14 + 3.69(hours) + 17.77(0) - 11.85(1) - 0.29(hours * 0) - 5.15 (hours * 1)
$$ By dropping the terms that have a zero in it and adding the constant terms (e.g., those that have a 1), the equation simplifies to the equation below.
This is the simple slope for the reading program.
Another way to say it, this is the relationship between hours spent exercising and the outcome of weight loss for those who were in the reading program.

$$
y_{reading} = -3.71 - 1.46 (hours) 
$$

Swimming: $$
y_{swimming} = 8.14 + 3.69(hours) + 17.77(1) - 11.85(0) - 0.29(hours * 1) - 5.15 (hours * 0)
$$ The simple slope for those who were in the swimming program is below:

$$
y_{swimming} = 25.91 + 3.40(hours) 
$$

Jogging: $$
y_{jogging} = 8.14 + 3.69(hours) + 17.77(0) - 11.85(0) - 0.29(hours * 0) - 5.15 (hours * 0)
$$ The simple slope for those who were in the jogging program is below:

$$
y_{jogging} = 8.14 + 3.69(hours) 
$$

Looking at those three equations for each of the program time, we can see they vary both in their intercepts and their slopes.
It is not directly evident just by looking at the point estimates in the table above how the combination of parameter estimates leads to the difference across the exercise programs– but simple slopes helps us!

------------------------------------------------------------------------

### Plotting Interactions

Plot code adapted from [here](https://cran.r-project.org/web/packages/interactions/vignettes/interactions.html#simple_slopes_analysis_and_johnson-neyman_intervals) (Long, 2020).

Here, we plot the interaction or simple slope plots between `hours` and `loss` and levels of `prog`.
The `interact_plot()` also plots the confidence intervals at 90%.

``` r
interact_plot(mod, pred = hours, modx = prog, interval = TRUE, int.width = 0.9, data = exercise) + theme_apa()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-1.png" width="672" />

``` r
mod_adj <- lm(loss ~ hours_scaled*prog, data=exercise)
interact_plot(mod_adj, pred = hours_scaled, modx = prog, interval = TRUE, int.width = 0.9, data = exercise) + theme_apa()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-7-1.png" width="672" />

Note that the only difference between the two interaction plots above is the scale of the x variable, *hours*.
In the top plot, the hours variable is not scaled and in the bottom one it is.
This does not change the slope, but it does change the intercept since we are moving the “zero” value in the scaled versoin.

### Simple Slopes

Here, we can test the simple slopes at each level of *hours spend exercising*.
The results show us that for all levels of `prog`, there a significant association between *hours spent exercising* and *weight loss*.
Note that the slope is different for each program.

``` r
# The moderation needs to be adjusted a bit for this function to work (it doesn't like when we scale the hours variable directly in the lm function)
mod_adj <- lm(loss ~ hours_scaled*prog, data=exercise)

sim_slopes(mod_adj, pred = hours_scaled, modx = prog, johnson_neyman = FALSE)
```

    ## SIMPLE SLOPES ANALYSIS 
    ## 
    ## Slope of hours_scaled when prog = Reading: 
    ## 
    ##    Est.   S.E.   t val.      p
    ## ------- ------ -------- ------
    ##   -1.46   0.35    -4.18   0.00
    ## 
    ## Slope of hours_scaled when prog = Swimming: 
    ## 
    ##   Est.   S.E.   t val.      p
    ## ------ ------ -------- ------
    ##   3.40   0.38     8.84   0.00
    ## 
    ## Slope of hours_scaled when prog = Jogging: 
    ## 
    ##   Est.   S.E.   t val.      p
    ## ------ ------ -------- ------
    ##   3.69   0.40     9.25   0.00

------------------------------------------------------------------------

## Regression with a Continuous Moderator

**Research Question:** Does *effort during exercise* moderate the relationship between *hours spent exercising* and *weight loss*?
Said differently (and equivalently), Does the relationship between the *hours spent* exercising and weight loss vary by the *effort during exercise*?

``` r
reg2 <- lm(loss ~ scale(hours)+effort, 
                  data=exercise)
mod2 <- lm(loss ~ scale(hours)*effort, 
                  data=exercise)

tab_model(reg2, mod2,show.std = TRUE, show.stat = TRUE, show.zeroinf = TRUE )
```

<table style="border-collapse:collapse; border:none;">
<tr>
<th style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm;  text-align:left; ">
 
</th>
<th colspan="6" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; ">
loss
</th>
<th colspan="8" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; ">
loss
</th>
</tr>
<tr>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  text-align:left; ">
Predictors
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">
Estimates
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">
std. Beta
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">
CI
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">
standardized CI
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">
Statistic
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  col7">
p
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  col8">
Estimates
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  col9">
std. Beta
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  0">
CI
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  1">
standardized CI
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  2">
Statistic
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  3">
std. Statistic
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  4">
p
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  5">
std. p
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
(Intercept)
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-10.90
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.00
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-16.10 – -5.69
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.06 – 0.06
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-4.11
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col7">
<strong>\<0.001</strong>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col8">
-10.98
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col9">
-0.00
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  0">
-16.17 – -5.78
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  1">
-0.06 – 0.06
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  2">
-4.15
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  3">
-0.04
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  4">
<strong>\<0.001</strong>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  5">
0.971
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
hours
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
1.16
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.08
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.27 – 2.05
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.02 – 0.15
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
2.56
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col7">
<strong>0.010</strong>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col8">
-4.64
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col9">
0.08
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  0">
-10.13 – 0.86
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  1">
0.02 – 0.14
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  2">
-1.66
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  3">
2.50
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  4">
0.098
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  5">
<strong>0.012</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
effort
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.71
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.26
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.53 – 0.88
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.19 – 0.32
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
8.00
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col7">
<strong>\<0.001</strong>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col8">
0.71
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col9">
0.26
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  0">
0.53 – 0.88
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  1">
0.20 – 0.32
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  2">
8.04
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  3">
8.04
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  4">
<strong>\<0.001</strong>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  5">
<strong>\<0.001</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
hours \* effort
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col7">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col8">
0.19
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col9">
0.07
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  0">
0.01 – 0.38
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  1">
0.00 – 0.14
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  2">
2.10
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  3">
2.10
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  4">
<strong>0.036</strong>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  5">
<strong>0.036</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm; border-top:1px solid;">
Observations
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="6">
900
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="8">
900
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">
R<sup>2</sup> / R<sup>2</sup> adjusted
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="6">
0.074 / 0.072
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="8">
0.078 / 0.075
</td>
</tr>
</table>

``` r
#Can also use stargazer::stargazer()
stargazer(reg2, mod2, type = "text")
```

    ## 
    ## ===================================================================
    ##                                   Dependent variable:              
    ##                     -----------------------------------------------
    ##                                          loss                      
    ##                               (1)                     (2)          
    ## -------------------------------------------------------------------
    ## scale(hours)                1.162**                 -4.637*        
    ##                             (0.453)                 (2.801)        
    ##                                                                    
    ## effort                     0.705***                0.707***        
    ##                             (0.088)                 (0.088)        
    ##                                                                    
    ## scale(hours):effort                                 0.195**        
    ##                                                     (0.093)        
    ##                                                                    
    ## Constant                  -10.897***              -10.975***       
    ##                             (2.653)                 (2.648)        
    ##                                                                    
    ## -------------------------------------------------------------------
    ## Observations                  900                     900          
    ## R2                           0.074                   0.078         
    ## Adjusted R2                  0.072                   0.075         
    ## Residual Std. Error    13.586 (df = 897)       13.560 (df = 896)   
    ## F Statistic         35.659*** (df = 2; 897) 25.330*** (df = 3; 896)
    ## ===================================================================
    ## Note:                                   *p<0.1; **p<0.05; ***p<0.01

``` r
# and jtools:summ()
summ(mod2)
```

<table class="table table-striped table-hover table-condensed table-responsive" style="width: auto !important; margin-left: auto; margin-right: auto;">
<tbody>
<tr>
<td style="text-align:left;font-weight: bold;">
Observations
</td>
<td style="text-align:right;">
900
</td>
</tr>
<tr>
<td style="text-align:left;font-weight: bold;">
Dependent variable
</td>
<td style="text-align:right;">
loss
</td>
</tr>
<tr>
<td style="text-align:left;font-weight: bold;">
Type
</td>
<td style="text-align:right;">
OLS linear regression
</td>
</tr>
</tbody>
</table>
<table class="table table-striped table-hover table-condensed table-responsive" style="width: auto !important; margin-left: auto; margin-right: auto;">
<tbody>
<tr>
<td style="text-align:left;font-weight: bold;">
F(3,896)
</td>
<td style="text-align:right;">
25.33
</td>
</tr>
<tr>
<td style="text-align:left;font-weight: bold;">
R²
</td>
<td style="text-align:right;">
0.08
</td>
</tr>
<tr>
<td style="text-align:left;font-weight: bold;">
Adj. R²
</td>
<td style="text-align:right;">
0.08
</td>
</tr>
</tbody>
</table>
<table class="table table-striped table-hover table-condensed table-responsive" style="width: auto !important; margin-left: auto; margin-right: auto;border-bottom: 0;">
<thead>
<tr>
<th style="text-align:left;">
</th>
<th style="text-align:right;">
Est.
</th>
<th style="text-align:right;">
S.E.
</th>
<th style="text-align:right;">
t val.
</th>
<th style="text-align:right;">
p
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;font-weight: bold;">
(Intercept)
</td>
<td style="text-align:right;">
-10.98
</td>
<td style="text-align:right;">
2.65
</td>
<td style="text-align:right;">
-4.15
</td>
<td style="text-align:right;">
0.00
</td>
</tr>
<tr>
<td style="text-align:left;font-weight: bold;">
scale(hours)
</td>
<td style="text-align:right;">
-4.64
</td>
<td style="text-align:right;">
2.80
</td>
<td style="text-align:right;">
-1.66
</td>
<td style="text-align:right;">
0.10
</td>
</tr>
<tr>
<td style="text-align:left;font-weight: bold;">
effort
</td>
<td style="text-align:right;">
0.71
</td>
<td style="text-align:right;">
0.09
</td>
<td style="text-align:right;">
8.04
</td>
<td style="text-align:right;">
0.00
</td>
</tr>
<tr>
<td style="text-align:left;font-weight: bold;">
scale(hours):effort
</td>
<td style="text-align:right;">
0.19
</td>
<td style="text-align:right;">
0.09
</td>
<td style="text-align:right;">
2.10
</td>
<td style="text-align:right;">
0.04
</td>
</tr>
</tbody>
<tfoot>
<tr>
<td style="padding: 0; " colspan="100%">
<sup></sup> Standard errors: OLS
</td>
</tr>
</tfoot>
</table>

### Model Equations

``` r
extract_eq(mod2)
```

$$
\operatorname{loss} = \alpha + \beta_{1}(\operatorname{scale(hours)}) + \beta_{2}(\operatorname{effort}) + \beta_{3}(\operatorname{scale(hours)} \times \operatorname{effort}) + \epsilon
$$

``` r
extract_eq(mod2, use_coefs = TRUE)
```

$$
\operatorname{\widehat{loss}} = -10.98 - 4.64(\operatorname{scale(hours)}) + 0.71(\operatorname{effort}) + 0.19(\operatorname{scale(hours)} \times \operatorname{effort})
$$

------------------------------------------------------------------------

### Plotting Interactions

Plot code adapted from [here](https://cran.r-project.org/web/packages/interactions/vignettes/interactions.html#simple_slopes_analysis_and_johnson-neyman_intervals) (Long, 2020).

Here, we plot the interaction between `hours` and `loss` for +1 SD, mean, and -2 SD of `effort`.

``` r
interact_plot(mod2, pred = hours, modx = effort, interval = TRUE, int.width = 0.9, data = exercise) + theme_apa()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-11-1.png" width="672" />

------------------------------------------------------------------------

### Simple Slopes

Here, we can test the simple slopes at each level (in this case +1 SD, mean, -1SD) of *effort during exercise.* The results show us that for mean and +1 SD levels of *effort*, the slope is significantly different than zero.

*NOTE*: There is a difference in the way we interpret continuous by continuous interactions.
We use the Johnson-Neyman approach (`johnson_neyman=TRUE`) will tell us the exact values of the moderator for which the slope of the predictor will be statistically significant.
We can also plot the Johnson Neyman test by using `jnplot=TRUE`.

``` r
# The moderation needs to be adjusted a bit for this function to work (it doesn't like when we scale the hours variable directly in the lm function)
mod_adj <- lm(loss ~ hours_scaled*effort, data=exercise)

sim_slopes(mod_adj, pred = hours_scaled, modx = effort, johnson_neyman = TRUE, jnplot = TRUE)
```

    ## JOHNSON-NEYMAN INTERVAL 
    ## 
    ## When effort is OUTSIDE the interval [-64.73, 28.55], the slope of
    ## hours_scaled is p < .05.
    ## 
    ## Note: The range of observed values of effort is [12.95, 44.08]

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-12-1.png" width="672" />

    ## SIMPLE SLOPES ANALYSIS 
    ## 
    ## Slope of hours_scaled when effort = 24.51646 (- 1 SD): 
    ## 
    ##   Est.   S.E.   t val.      p
    ## ------ ------ -------- ------
    ##   0.13   0.67     0.20   0.84
    ## 
    ## Slope of hours_scaled when effort = 29.65922 (Mean): 
    ## 
    ##   Est.   S.E.   t val.      p
    ## ------ ------ -------- ------
    ##   1.13   0.45     2.50   0.01
    ## 
    ## Slope of hours_scaled when effort = 34.80198 (+ 1 SD): 
    ## 
    ##   Est.   S.E.   t val.      p
    ## ------ ------ -------- ------
    ##   2.13   0.65     3.30   0.00

Johnson-Neyman interpretation: When effort is OUTSIDE the interval \[-64.73, 28.55\], the slope of hours_scaled is p \< .05.
In other words, the Johnson-Neymen tells us that when effort is less than 28.55, there is no significant relationship between hours and weight loss.
However, for those who put in and effort level of 29 (rounding 28.55) and greater, there is a significant weight loss for the hours they put in.

Bottom line: It doesn’t matter how much time you put in to working out, you won’t see weight loss if the effort is low (less than 29).

------------------------------------------------------------------------

Write up (note the formatting is not APA here– just the text)

A moderated regression model was used to explore the impact of the effort of an athlete on the relationship between hours spent exercising and weight loss.
Specifically, we estimated a regression model with the hours spent exercising (standardized), effort, and the interaction of hours and effort to predict weight loss.
There was evidence that there is a relationship with at least one predictor, *F*(3, 896) = 25.33, *p* \< .01, and collectively these variables explained 8% of the variability in weight loss.
Results indicated that there was a significant interaction (*B* = .19, *p* = .04).
Figure 1 presents the simple slope at the plot which we used to understand the interaction.
Simple slopes are plotted for those who put in high (+1 SD above the mean), moderate effort (mean), and low effort (-1 SD below the mean) effort.
The test of the simple slopes at each of these different levels indicated that for those who put in low effort (-1 SD below the mean), the slope is not significant (*B* = .13, *p* \>.05), indicating that for low effort, there is no relationship between the time spent exercising and weight loss.
However, for higher values of effort, there was a relation.
Specifically, for mean effort and high effort, the slope was significant, specifically *B* = 1.13 (*p* \< .05) and *B* = 2.13 (*p* \< .01).
This implies that the slope increases the more effort put in, that is there is greater weight loss for time put in for those who have higher effort.
The Johnson-Neyman interval (Johnson & Neyman, 1936) specifically indicates that for effort levels greater than 29, there is a positive and significant relationship between the time exercising and weight loss.

------------------------------------------------------------------------

**References**

Esarey, J., & Sumner, J. L.
(2017).
Marginal effects in interaction models: Determining and controlling the false positive rate.
Comparative Political Studies, 1–33.
Advance online publication.
<https://doi.org/10.1177/0010414017730080>

Johnson, P. O.
& Neyman, J.
(1936).
Tests of certain linear hypotheses and their applications to some educational problems.
Statistical Research Memoirs, 1, 57–93.
R Core Team (2021).
R: A language and environment for statistical computing.
R Foundation for Statistical Computing, Vienna, Austria.
URL <https://www.R-project.org/>.
