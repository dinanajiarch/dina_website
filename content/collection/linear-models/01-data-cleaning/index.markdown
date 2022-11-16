---
title: "Lab 1: Data Screening and Cleaning"
subtitle: "Information on how to clean and screen your data and write up the results."
date: 2021-03-30
author: "TA: Dina Arch"
weight: 1
draft: false
images:
series:
tags:
- r markdown
- data cleaning
- data screening
- tidy data
- regression
categories:
- data analysis
editor_options: 
  markdown: 
    wrap: sentence
---




```r
#Load packages here
library(haven) #for read_sav() function
library(psych) # describe()
library(ggpubr) # ggdensity() and ggqqplot()
library(apaTables) # apa.cor.table()
library(tidyverse)
```

## Data Screening & Cleaning

The data sets used in this lab are used in Chapter 4 of Tabachnick & Fidell (2012; I believe there is a new version from 2019 now).
The chapter is provided in GauchoSpace and is a useful resource if you need more information on how to clean and screen your data and write up the results.

Here is a table of the variables and their description:

|              |                                |
|--------------|--------------------------------|
| **Variable** | **Description**                |
| subno        | Subject number                 |
| timedrs      | Visits to health professionals |
| attdrug      | Attitudes toward medication    |
| atthouse     | Attitudes toward housework     |
| income       | Ordinal income variable        |
| emplmnt      | Whether currently employed     |
| mstatus      | Whether currently married      |
| race         | Binary race variable           |

The style of code and package we will be using for this class is called [`tidyverse`](https://www.tidyverse.org/) .
This shouldn't be anything too different than what you've seen in the previous two courses in this series.
Most functions are within the `tidyverse` package and if not, I've indicated the packages used in the code chunk above.

As a reminder, if a function does not work and you receive an error like this: `could not find function "random_function"`; or if you try to load a package and you receive an error like this: `` there is no package called `random_package` `` , then you will need to install the package using `install.packages("random_package")` in the console (the bottom-left window in R studio).
Once you have installed the package you will *never* need to install it again, however you must *always* load in the packages at the beginning of your R markdown using `library(random_package)`, as shown in this document.

### Read in Data

Let's read in a data set called `screen1.sav`:


```r
# Notice this is an SPSS dataset
screen1 <- read_sav("screen1.sav")
```

Usually, we want to work with .csv files, so let's convert this data set from *.sav* to *.csv* and save it to our computers.


```r
# write_csv saves a .csv version of your dataset to your working directory.
# Enter the name of the object that contains your data set (in this case, "screen1"), then enter the name you want to save your dataset as. We can call it the same thing: "screen1.csv"

write_csv(screen1, "screen1.csv")
```

Let's read in the new *.csv* data set:


```r
# If you get an error, you might need to wait a few seconds for the .csv to save to your folder before running this. 

screen1 <- read_csv("screen1.csv")
```

Use `names()` to view the variable names in the data set:


```r
names(screen1)
```

```
## [1] "subno"    "timedrs"  "atthouse"
```

Let's read in another data set `screen2.sav` and convert it to a *.csv* like we did before:


```r
# Read in .sav
screen2 <- read_sav("screen2.sav")

# Convert and save a .csv version
write.csv(screen2, "screen2.csv")

# Read in the new .csv version
screen2 <- read_csv("screen2.csv")

# Look at variable names
names(screen2)
```

```
## [1] "...1"    "income"  "mstatus" "race"
```

### Merge Data

What if we want to merge the two data sets?
As with many thing in R, there are multiple ways to do this.
Here is probably the most efficient way to merge data sets using `tidyverse::full_join()`.
This will join two data sets based on a "join" variable and return all rows and columns from both data sets.
Because we are joining two data sets that have different variable names to join them by (i.e., the ID or Subject number) , we use `by` shown below to let R know that there are different names.
As a reminder, use the pipe operator `%>%` to create a sequence of functions.


```r
# Here we created a new dataset called `merged_screen`
merged_screen <- screen1 %>% 
  full_join(screen2, by = c("subno" = "...1")) 
```

### Examine Data

Let's examine the data set:


```r
# View the first 5 rows
head(merged_screen)
```

```
## # A tibble: 6 × 6
##   subno timedrs atthouse income mstatus  race
##   <dbl>   <dbl>    <dbl>  <dbl>   <dbl> <dbl>
## 1     1       1       27      5       2     1
## 2     2       3       20      6       2     1
## 3     3       0       23      3       2     1
## 4     4      13       28      8       2     1
## 5     5      15       24      1       2     1
## 6     6       3       25      4       2     1
```

Let's look at descriptive statistics for each variable.
Because looking at the ID variable's (`subno`) descriptives is unnecessary, we use `select()` to remove the variable by using the minus (`-`) sign:


```r
merged_screen %>% 
  select(-subno) %>% 
  summary() 
```

```
##     timedrs          atthouse         income         mstatus     
##  Min.   : 0.000   Min.   : 2.00   Min.   : 1.00   Min.   :1.000  
##  1st Qu.: 2.000   1st Qu.:21.00   1st Qu.: 2.50   1st Qu.:2.000  
##  Median : 4.000   Median :24.00   Median : 4.00   Median :2.000  
##  Mean   : 7.901   Mean   :23.54   Mean   : 4.21   Mean   :1.778  
##  3rd Qu.:10.000   3rd Qu.:27.00   3rd Qu.: 6.00   3rd Qu.:2.000  
##  Max.   :81.000   Max.   :35.00   Max.   :10.00   Max.   :2.000  
##  NA's   :128      NA's   :128     NA's   :154     NA's   :128    
##       race      
##  Min.   :1.000  
##  1st Qu.:1.000  
##  Median :1.000  
##  Mean   :1.088  
##  3rd Qu.:1.000  
##  Max.   :2.000  
##  NA's   :128
```

Alternatively, we can use the `psych::describe()` function to give more information:


```r
merged_screen %>% 
  select(-subno) %>% 
  describe()
```

```
##          vars   n  mean    sd median trimmed  mad min max range  skew kurtosis
## timedrs     1 465  7.90 10.95      4    5.61 4.45   0  81    81  3.23    12.88
## atthouse    2 465 23.54  4.48     24   23.62 4.45   2  35    33 -0.45     1.52
## income      3 439  4.21  2.42      4    4.01 2.97   1  10     9  0.58    -0.38
## mstatus     4 465  1.78  0.42      2    1.85 0.00   1   2     1 -1.34    -0.21
## race        5 465  1.09  0.28      1    1.00 0.00   1   2     1  2.90     6.40
##            se
## timedrs  0.51
## atthouse 0.21
## income   0.12
## mstatus  0.02
## race     0.01
```

What if we want to look at a subset of the data?
For example, what if we want to subset the data to see whose been to the doctor more than 10 times?
We can use `tidyverse::filter()` to subset the data using certain criteria.


```r
merged_screen %>% 
  filter(timedrs > 10) %>% 
  describe() 
```

```
##          vars   n   mean     sd median trimmed    mad min max range  skew
## subno       1 107 324.25 199.63    292  320.41 269.83   4 756   752  0.14
## timedrs     2 107  22.49  14.90     16   19.40   5.93  11  81    70  1.88
## atthouse    3 107  24.33   4.14     24   24.36   4.45  14  35    21 -0.02
## income      4  68   4.19   2.32      4    4.05   1.48   1  10     9  0.52
## mstatus     5  72   1.81   0.40      2    1.88   0.00   1   2     1 -1.51
## race        6  72   1.11   0.32      1    1.02   0.00   1   2     1  2.42
##          kurtosis    se
## subno       -1.14 19.30
## timedrs      3.06  1.44
## atthouse    -0.10  0.40
## income      -0.42  0.28
## mstatus      0.29  0.05
## race         3.93  0.04
```

```r
#You can use any operator to filter: >, <, ==, >=, etc.
```

### Recode Missing Values

Let's check for missing values.
First, how are missing values identified?
They could be `-999`, `NA`, or literally anything else.
The simplest way to do this is to look back at the `summary()` function.
We can see that they are identified as `NA`.
Because, for example, `timedrs` has 128 `NA`'s.
We also know that it's not `-999` because the *min* and *max* values look normal.


```r
merged_screen %>% 
  select(-subno) %>% 
  summary() 
```

```
##     timedrs          atthouse         income         mstatus     
##  Min.   : 0.000   Min.   : 2.00   Min.   : 1.00   Min.   :1.000  
##  1st Qu.: 2.000   1st Qu.:21.00   1st Qu.: 2.50   1st Qu.:2.000  
##  Median : 4.000   Median :24.00   Median : 4.00   Median :2.000  
##  Mean   : 7.901   Mean   :23.54   Mean   : 4.21   Mean   :1.778  
##  3rd Qu.:10.000   3rd Qu.:27.00   3rd Qu.: 6.00   3rd Qu.:2.000  
##  Max.   :81.000   Max.   :35.00   Max.   :10.00   Max.   :2.000  
##  NA's   :128      NA's   :128     NA's   :154     NA's   :128    
##       race      
##  Min.   :1.000  
##  1st Qu.:1.000  
##  Median :1.000  
##  Mean   :1.088  
##  3rd Qu.:1.000  
##  Max.   :2.000  
##  NA's   :128
```

Additionally, we can look at the unique values (using `unique()`) for each variable to see if there are any unusual values or multiple missing values were coded.
For example, for `timedrs` (Visits to health professionals), we can see that the unique values don't look unusual, and there is `NA`.


```r
unique(merged_screen$timedrs)
```

```
##  [1]  1  3  0 13 15  2  7  4  5 12  6 14 60 20  8  9 10 23 39 16 33 38 22 11 34
## [26] 27 30 17 25 19 49 52 18 24 57 21 58 43 37 75 29 56 81 NA
```

Usually, we want our missing values to be identified as `NA` since that's what R understands as missing by default.
What if your dataset doesn't have missing as `NA`?
What if the missing values are `-999`?
Or vise versa?
Here is how you would recode the values from `NA` to `-999` (but usually, you would never do this since you want the missing to be labeled as `NA`):


```r
# Use tidyverse::mutate() adjust an existing variable
# Then use tidyverse::replace_na() to replace -999 with NA for the timedrs variable
new_data_with_NA <- merged_screen %>% 
  mutate(timedrs = replace_na(timedrs, -999))

# Recode back to `NA` using tidyverse::na_if()
new_data_with_NA <- merged_screen %>% 
  mutate(timedrs = na_if(timedrs, "-999"))
```

### Recode Continuous Variable into Factor

What if you want to recode a continuous variable into different levels (e.g., high, medium, and low)?
Let's use the variable `income` as an example.
First, let's recall the descriptives:


```r
merged_screen %>% 
  select(income) %>% 
  summary() 
```

```
##      income     
##  Min.   : 1.00  
##  1st Qu.: 2.50  
##  Median : 4.00  
##  Mean   : 4.21  
##  3rd Qu.: 6.00  
##  Max.   :10.00  
##  NA's   :154
```

Here, we can see that the values range from 1 - 10.
Lets recode the variable to look like this:

|        |        |
|--------|--------|
| Low    | 1 - 3  |
| Medium | 4 - 6  |
| High   | 7 - 10 |

We use `cut()` to divide continuous variables into intervals:


```r
income_factor <- merged_screen %>% 
  mutate(income_factor = cut(income,
                      breaks = c(0, 3, 6, 10), #Use 0 at the beginning to ensure that 1 is included in the first break, then break by last number in each section (3, 6, 10) 
                      labels = c("low", "medium", "high")))
# View summary
income_factor %>% 
  select(income, income_factor) %>% 
  summary() 
```

```
##      income      income_factor
##  Min.   : 1.00   low   :189   
##  1st Qu.: 2.50   medium:166   
##  Median : 4.00   high  : 84   
##  Mean   : 4.21   NA's  :154   
##  3rd Qu.: 6.00                
##  Max.   :10.00                
##  NA's   :154
```

### Normality and Transformation

It's important to inspect our data to see if it is normally distributed.
Many analyses are sensitive to violations of normality so in order to make sure we are confident that our data are normal, there are several things we can look at: density plots, histograms, QQ plots, the Shapiro-Wilk's normality test, and the descriptives such as skewness and kurtosis.
Normally, we would want to inspect every variable, but for demonstration purposes, lets focus on the `timedrs` variable.

#### Density Plots

A density plot is a visualization of the data over a continuous interval.
As we can see by this density plot, the variable `timedrs` is positively skewed.
There are more people that make less doctor visits than those who make more doctor visits.


```r
ggdensity(merged_screen$timedrs, fill = "lightgray")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-17-1.png" width="672" />

#### Histogram

A histogram provides the same information as the density plot but provides a count instead of density on the x-axis.


```r
gghistogram(merged_screen$timedrs, fill = "lightgray")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-18-1.png" width="672" />

#### QQ Plots

QQ plot, or quantile-quantile plot, is a plot of the correlation between a sample and the normal distribution.
In a QQ plot, each observation is plotted as a single dot.
If the data are normal, the dots should form a straight line.
If the data are skewed, you will see either a downward curve (negatively skewed) or upward curve (positively skewed).


```r
ggqqplot(merged_screen$timedrs)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-19-1.png" width="672" />

As you can see in this QQ plot, there is an upward curve, which further tells us that we have a positively skewed variable.

#### Shapiro-Wilk's Test of Normality

Sometimes it's difficult to tell whether the variable is normal based on the plots alone (not in this case, however).
The Shapiro-Wilk's normality test is a significance test that compares the sample distribution to a normal distribution.
If the test is not significant, then the distribution is normal.
As in there is no significant difference between the sample and a normal distribution.
If the test is significant, then the distribution is *not* normal.
As in, there is a significant difference between the sample and a normal distribution.
We can use `shapiro_test` to test this.


```r
#library(rstatix) 
#merged_screen %>% 
#  shapiro_test(timedrs)
```

To no one's surprise, the Shapiro-Wilk's Test of Normality was significant for the `timedrs` variable.

#### Skewness and Kurtosis

One final thing to look are the skewness and kurtosis values in the descriptive statistics provided earlier.
There are many different sources that provide different cut-off values, but as a general rule of thumb, skewness and kurtosis values greater than +3/-3 indicate a non-normal distribution.
Positive skew values indicate positively skewed variables and negative skew values indicate negatively skewed variables.
Positive values of kurtosis indicate leptokurtic distributions, or higher peaks with taller tails than a normal distribution.
Negative values of kurtosis indicate platykurtic distributions, or flat peaks with thin tails.


```r
describe(merged_screen$timedrs)
```

```
##    vars   n mean    sd median trimmed  mad min max range skew kurtosis   se
## X1    1 465  7.9 10.95      4    5.61 4.45   0  81    81 3.23    12.88 0.51
```

Here we can see that the *skew* value is greater than 3 and positive, indicating a positive distribution.
The *kurtosis* value is greater than 3 and positive, probably because, as we saw in the density plot, it has a high peak and tall tails.

#### Transformation

So, you've violated normality, now what?
Well, we need to consider the analysis were about to run.
Is it robust to minor violations of normality (such as regression or ANOVA)?
If we decide that the violation of normality is a big deal, we can use a mathematical function to transform the each point in the data set.

Most of the time, we don't like to transform a variable unless it's absolutely necessary.
I've found that researchers in other departments (other than social science) transform data all the time, but since we're working with responses from real people it's a bit trickier.
There are many different ways to do this, the one we will use is logarithmic transformation.
[Here](https://en.wikipedia.org/wiki/Data_transformation_%28statistics%29) is an in-depth look at how transformations work.

Log transform `timedrs` using `log()`:


```r
log <- merged_screen %>% 
  drop_na() %>% #droping NA values
  mutate(log_timedrs = log(timedrs + 1)) # Adding a constant (1) here because there were zeros's. Log(0) is undefined. You do not need to add 1 if there are no zeros.

#Before transformation
ggdensity(merged_screen$timedrs, fill = "lightgray")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-22-1.png" width="672" />

```r
#After transformation
ggdensity(log$log_timedrs, fill = "lightgray")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-22-2.png" width="672" />

```r
#Let's also look at changes in descriptives
describe(log$log_timedrs)
```

```
##    vars   n mean   sd median trimmed  mad min  max range skew kurtosis   se
## X1    1 321 1.66 0.97   1.61    1.65 0.76   0 4.33  4.33 0.26    -0.21 0.05
```

Here, we can see a big improvement in the distribution compared to the previous, untransformed variable.

## Regression Overview

Let's start this section using applied examples:

**Simple Linear Regression**: Predict **physics** scores from **math SAT scores**.
$$\hat Y = \beta_0 + \beta_1x $$ $$ physics = \beta_0 + \beta_1 (msat) $$ **Multiple Regression**: Predict **physics** scores from: **emotional intelligence, IQ, verbal SAT, math SAT, creativity, and abstract thinking.** $$ \hat Y = \beta_0 + \beta_1x + \beta_2x ... \beta_ix_i  $$ $$physics = \beta_0 + \beta_1(ei) + \beta_2(iq) + \beta_3(vsat) + \beta_4(msat) + \beta_5(creativity) + \beta_6(abstract)  $$

Read in data:


```r
physics <- read_sav("physics.sav")

write.csv(physics, "physics.csv")

physics <- read_csv("physics.csv") %>% 
  select(-...1, -idno) 


# Descriptive Statistics
describe(physics) 
```

```
##          vars   n   mean     sd median trimmed    mad   min max range  skew
## ei          1 199 107.59  12.17  108.0  107.82  10.38  75.0 139  64.0 -0.15
## iq          2 199  99.99  14.59  100.0   99.89  14.83  57.0 134  77.0  0.01
## vsat        3 199 534.46 105.37  526.0  532.13 108.23 314.0 801 487.0  0.21
## msat        4 199 493.38  42.98  487.0  490.93  43.00 364.0 601 237.0  0.40
## create      5 200   9.57   3.78    9.0    9.53   4.45   1.0  19  18.0  0.14
## abstract    6 199   9.46   2.98   10.0    9.64   2.97   1.0  15  14.0 -0.55
## physics     7 195  79.57  17.58   79.5   79.34  16.90  29.5 119  89.5  0.04
##          kurtosis   se
## ei          -0.20 0.86
## iq          -0.23 1.03
## vsat        -0.51 7.47
## msat        -0.25 3.05
## create      -0.57 0.27
## abstract     0.15 0.21
## physics     -0.10 1.26
```

### Assumptions of Regression

1.  Linearity - Relationship between the outcome variable and each predictor

2.  Normality - Are the data normally distributed?

3.  Homoscedasticity - Residuals are assumed to have a constant variance (*This can be checked after running the model*)

4.  Multicollinearity - Bivariate correlations are less than .90.

#### Linearity & Multicollinearity

The easiest way to check **linearity** of relationship between the outcome variable and each predictor is to check the bivariate correlations and scatterplots.
For the most part, most variables are (positively) correlated with each other, implying linear relationships.

To assess **multicollinearity**, we want to see if there are predictors that are too highly correlated (*r* \> .90)

Use `cor()` to look at bivariate correlations for the entire data set (Note: Missing values are not allowed in correlation analyses, use `drop_na()` to do listwise deletion):


```r
physics %>% 
  drop_na() %>% #remove missing data
  cor(method = "pearson")
```

```
##                   ei         iq        vsat       msat      create   abstract
## ei        1.00000000 0.03400695  0.53927222 0.04823131 -0.02356759 0.02311522
## iq        0.03400695 1.00000000  0.64810303 0.70635708  0.64281265 0.57259750
## vsat      0.53927222 0.64810303  1.00000000 0.50502038  0.45670255 0.35357159
## msat      0.04823131 0.70635708  0.50502038 1.00000000  0.57382957 0.50705131
## create   -0.02356759 0.64281265  0.45670255 0.57382957  1.00000000 0.42365795
## abstract  0.02311522 0.57259750  0.35357159 0.50705131  0.42365795 1.00000000
## physics  -0.38171959 0.35304329 -0.09995838 0.69101577  0.35328045 0.25249211
##              physics
## ei       -0.38171959
## iq        0.35304329
## vsat     -0.09995838
## msat      0.69101577
## create    0.35328045
## abstract  0.25249211
## physics   1.00000000
```

```r
#Fun tip: `apa.cor.table()` creates an APA formated correlation matrix and saves it to your computer
#apa.cor.table(physics, filename = "cor_table.doc")
```

You can also subset the data set to include only the variables of interest:


```r
physics %>% 
  select(ei, iq, physics) %>% 
  drop_na() %>% #remove missing data
  cor() 
```

```
##                 ei        iq    physics
## ei       1.0000000 0.0360554 -0.3941978
## iq       0.0360554 1.0000000  0.3440316
## physics -0.3941978 0.3440316  1.0000000
```

\[Correlation Interpretation:\]{.ul}

Among the students in the sample, IQ and physics test scores were positively correlated, *r* = 0.34, *p* \< .001.

We can also look at partial-correlations using `ppcor::pcor()` to see how the predictors relate to the outcome after controlling for the other correlations and semi-partial correlations using `ppcor::spcor()` to see how predictors relate to the outcome after controlling for *X* but not *Y*.


```r
library(ppcor) 
# Partial Correlation
physics %>% 
  drop_na() %>% 
  pcor() 
```

```
## $estimate
##                   ei          iq       vsat       msat      create    abstract
## ei        1.00000000 -0.32568951  0.4650756 0.16441386 -0.12115000  0.01488448
## iq       -0.32568951  1.00000000  0.4746335 0.17078572  0.23857783  0.29849210
## vsat      0.46507555  0.47463350  1.0000000 0.44653013  0.16077919 -0.13045988
## msat      0.16441386  0.17078572  0.4465301 1.00000000  0.05092278  0.21300341
## create   -0.12115000  0.23857783  0.1607792 0.05092278  1.00000000  0.07149435
## abstract  0.01488448  0.29849210 -0.1304599 0.21300341  0.07149435  1.00000000
## physics  -0.21342029  0.04833899 -0.5147686 0.80097974  0.08561041 -0.14365807
##              physics
## ei       -0.21342029
## iq        0.04833899
## vsat     -0.51476858
## msat      0.80097974
## create    0.08561041
## abstract -0.14365807
## physics   1.00000000
## 
## $p.value
##                    ei           iq         vsat         msat       create
## ei       0.000000e+00 4.805721e-06 1.560295e-11 2.377604e-02 0.0967917262
## iq       4.805721e-06 0.000000e+00 5.208355e-12 1.879216e-02 0.0009463296
## vsat     1.560295e-11 5.208355e-12 0.000000e+00 1.193188e-10 0.0270993829
## msat     2.377604e-02 1.879216e-02 1.193188e-10 0.000000e+00 0.4865040527
## create   9.679173e-02 9.463296e-04 2.709938e-02 4.865041e-01 0.0000000000
## abstract 8.389149e-01 3.021094e-05 7.356918e-02 3.253293e-03 0.3282642697
## physics  3.191877e-03 5.089145e-01 3.517794e-14 1.575852e-43 0.2414812417
##              abstract      physics
## ei       8.389149e-01 3.191877e-03
## iq       3.021094e-05 5.089145e-01
## vsat     7.356918e-02 3.517794e-14
## msat     3.253293e-03 1.575852e-43
## create   3.282643e-01 2.414812e-01
## abstract 0.000000e+00 4.859688e-02
## physics  4.859688e-02 0.000000e+00
## 
## $statistic
##                  ei         iq      vsat       msat     create   abstract
## ei        0.0000000 -4.7105727  7.184029  2.2793444 -1.6689948  0.2035648
## iq       -4.7105727  0.0000000  7.374048  2.3702833  3.3595141  4.2767879
## vsat      7.1840291  7.3740478  0.000000  6.8243429  2.2276026 -1.7993904
## msat      2.2793444  2.3702833  6.824343  0.0000000  0.6972632  2.9811919
## create   -1.6689948  3.3595141  2.227603  0.6972632  0.0000000  0.9801788
## abstract  0.2035648  4.2767879 -1.799390  2.9811919  0.9801788  0.0000000
## physics  -2.9873047  0.6617994 -8.210795 18.2952888  1.1750187 -1.9850851
##             physics
## ei       -2.9873047
## iq        0.6617994
## vsat     -8.2107955
## msat     18.2952888
## create    1.1750187
## abstract -1.9850851
## physics   0.0000000
## 
## $n
## [1] 194
## 
## $gp
## [1] 5
## 
## $method
## [1] "pearson"
```

```r
# Semi-Parial Correlation
physics %>% 
  drop_na() %>% 
  spcor()
```

```
## $estimate
##                   ei          iq       vsat       msat      create    abstract
## ei        1.00000000 -0.24386511  0.3719153 0.11800106 -0.08640342  0.01053850
## iq       -0.17991579  1.00000000  0.2816447 0.09053069  0.12831341  0.16334780
## vsat      0.24787217  0.25442843  1.0000000 0.23546184  0.07685947 -0.06208477
## msat      0.06775628  0.07045955  0.2028619 1.00000000  0.02072700  0.08861955
## create   -0.08966473  0.18048584  0.1196753 0.03745962  1.00000000  0.05265892
## abstract  0.01184521  0.24886148 -0.1047045 0.17347221  0.05703550  1.00000000
## physics  -0.10442322  0.02313364 -0.2870138 0.63952396  0.04107356 -0.06938997
##              physics
## ei       -0.15465198
## iq        0.02527679
## vsat     -0.28329892
## msat      0.54384971
## create    0.06312646
## abstract -0.11550987
## physics   1.00000000
## 
## $p.value
##                    ei           iq         vsat         msat     create
## ei       0.0000000000 0.0007207838 1.369299e-07 1.058482e-01 0.23713321
## iq       0.0132390483 0.0000000000 8.634775e-05 2.153898e-01 0.07847628
## vsat     0.0005840884 0.0004109848 0.000000e+00 1.107964e-03 0.29317174
## msat     0.3542482772 0.3353314498 5.117270e-03 0.000000e+00 0.77710849
## create   0.2198294618 0.0129459172 1.009535e-01 6.088274e-01 0.00000000
## abstract 0.8714869384 0.0005542444 1.516168e-01 1.697934e-02 0.43566074
## physics  0.1527254403 0.7520272679 6.223148e-05 3.984530e-23 0.57468919
##            abstract      physics
## ei       0.88556108 3.360256e-02
## iq       0.02471236 7.299077e-01
## vsat     0.39605865 7.811472e-05
## msat     0.22527418 6.106787e-16
## create   0.47174829 3.881654e-01
## abstract 0.00000000 1.134748e-01
## physics  0.34273760 0.000000e+00
## 
## $statistic
##                  ei         iq      vsat       msat     create   abstract
## ei        0.0000000 -3.4386200  5.478884  1.6249933 -1.1859843  0.1441198
## iq       -2.5011248  0.0000000  4.013921  1.2430931  1.7692850  2.2641585
## vsat      3.4987885  3.5976493  0.000000  3.3130430  1.0541557 -0.8506374
## msat      0.9286874  0.9659206  2.833000  0.0000000  0.2834983  1.2166410
## create   -1.2311057  2.5093158  1.648381  0.5126124  0.0000000  0.7211004
## abstract  0.1619921  3.5136730 -1.439726  2.4087159  0.7812205  0.0000000
## physics  -1.4358157  0.3164324 -4.097240 11.3757630  0.5621468 -0.9511863
##             physics
## ei       -2.1405874
## iq        0.3457654
## vsat     -4.0395481
## msat      8.8622340
## create    0.8649665
## abstract -1.5902181
## physics   0.0000000
## 
## $n
## [1] 194
## 
## $gp
## [1] 5
## 
## $method
## [1] "pearson"
```

```r
detach("package:ppcor", unload = TRUE)
```

Finally, use `pairs()` to look at bivariate scatterplots.
Do the relationships look linear?:


```r
pairs(physics)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-27-1.png" width="672" />

```r
# Or we can look at individual scatterplots:
physics %>% 
  ggplot(aes(physics, ei)) +
  geom_point() +
  geom_smooth(method = "lm", se =F) +
  theme_minimal()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-27-2.png" width="672" />

#### Normality

Density Plot:


```r
lapply(physics, ggdensity, fill = "lightgray")
```

```
## $ei
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-28-1.png" width="672" />

```
## 
## $iq
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-28-2.png" width="672" />

```
## 
## $vsat
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-28-3.png" width="672" />

```
## 
## $msat
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-28-4.png" width="672" />

```
## 
## $create
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-28-5.png" width="672" />

```
## 
## $abstract
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-28-6.png" width="672" />

```
## 
## $physics
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-28-7.png" width="672" />

```r
# alternatively, you may look at each one individually like this:
# ggdensity(physics$ei, fill = "lightgray")
```

QQ Plots for individual predictors (Later, we will want to look at the QQ plot of the entire model, which looks at the normality of residuals):


```r
lapply(physics, ggqqplot, fill = "lightgray")
```

```
## $ei
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-29-1.png" width="672" />

```
## 
## $iq
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-29-2.png" width="672" />

```
## 
## $vsat
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-29-3.png" width="672" />

```
## 
## $msat
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-29-4.png" width="672" />

```
## 
## $create
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-29-5.png" width="672" />

```
## 
## $abstract
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-29-6.png" width="672" />

```
## 
## $physics
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-29-7.png" width="672" />

Shapiro-Wilk's Test of Normality:


```r
#physics %>% 
#  shapiro_test(ei, iq, vsat, msat, create, abstract, physics)
#Here, abstract and msat is significant. However, Shapiro-Wilk's test is very sensitive to minor deviations from normality. Our density plot suggests normality.
```

Skewness and Kurtosis:


```r
describe(physics)
```

```
##          vars   n   mean     sd median trimmed    mad   min max range  skew
## ei          1 199 107.59  12.17  108.0  107.82  10.38  75.0 139  64.0 -0.15
## iq          2 199  99.99  14.59  100.0   99.89  14.83  57.0 134  77.0  0.01
## vsat        3 199 534.46 105.37  526.0  532.13 108.23 314.0 801 487.0  0.21
## msat        4 199 493.38  42.98  487.0  490.93  43.00 364.0 601 237.0  0.40
## create      5 200   9.57   3.78    9.0    9.53   4.45   1.0  19  18.0  0.14
## abstract    6 199   9.46   2.98   10.0    9.64   2.97   1.0  15  14.0 -0.55
## physics     7 195  79.57  17.58   79.5   79.34  16.90  29.5 119  89.5  0.04
##          kurtosis   se
## ei          -0.20 0.86
## iq          -0.23 1.03
## vsat        -0.51 7.47
## msat        -0.25 3.05
## create      -0.57 0.27
## abstract     0.15 0.21
## physics     -0.10 1.26
```

### Simple Linear Regression

Finally, we have arrived at our main goal.
It's good to reflect on how much work it takes to prepare the data for analyses.
It's important to clean and screen the data before analyses.

Recall our research hypothesis:

**Research Example**: Predict **physics** scores from **math SAT** scores.
Run a simple linear regression model using `lm()`:


```r
#`msat` predicting `physics`
slr_model <- lm(physics~msat,data=physics)
summary(slr_model)
```

```
## 
## Call:
## lm(formula = physics ~ msat, data = physics)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -36.735  -7.484  -0.243   8.980  30.098 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept) -58.54334   10.46659  -5.593 7.54e-08 ***
## msat          0.27977    0.02112  13.246  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.76 on 193 degrees of freedom
##   (5 observations deleted due to missingness)
## Multiple R-squared:  0.4762,	Adjusted R-squared:  0.4735 
## F-statistic: 175.5 on 1 and 193 DF,  p-value: < 2.2e-16
```

\[Simple Linear Regression Interpretation:\]{.ul}

For every one unit increase in `msat` scores, `physics` scores increase 0.28 points, *p* \< 0.001.

$$ physics = -58.54 + 0.28 (msat) $$

Looking at the **homoscedasticity** assumption, we use the `plot()` function.
To assess for constant variance, we want to see a straight, horizontal red line across this plot.


```r
# 3 is for the Scale-Location plot. This shows if residuals are spread equally along the ranges of preditors. A straight line means constant variance.
plot(slr_model, 3)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-33-1.png" width="672" />

Looking at the **normality of residuals** assumption (whereas before, we were looking at normality of each predictor), we again use `plot()` function.:


```r
# 2 is to assess the normality of residuals (QQ plot for the model) We want to see the points on the diagonal. 
plot(slr_model, 2)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-34-1.png" width="672" />

### Multiple Regression

Recall our research hypothesis:

**Research Example**: Predict **physics** scores from: emotional intelligence, IQ, verbal SAT, math SAT, creativity, and abstract thinking.

Run a multiple regression model using `lm()`:


```r
# using the period (.) tells R you want ALL the variables except the outcome (`physics`)
# alternatively, you can type out all the variables by doing this:
# lm(physics~ei+iq+vsat+msat+create+abstract, data=physics)
mr_model <- lm(physics~.,data=physics)
summary(mr_model)
```

```
## 
## Call:
## lm(formula = physics ~ ., data = physics)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -18.4419  -6.4058  -0.6015   6.2519  21.1205 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept) -40.80967   10.06104  -4.056 7.32e-05 ***
## ei           -0.20493    0.06860  -2.987  0.00319 ** 
## iq            0.05214    0.07878   0.662  0.50891    
## vsat         -0.08664    0.01055  -8.211 3.52e-14 ***
## msat          0.37663    0.02059  18.295  < 2e-16 ***
## create        0.25462    0.21669   1.175  0.24148    
## abstract     -0.50442    0.25411  -1.985  0.04860 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 8.425 on 187 degrees of freedom
##   (6 observations deleted due to missingness)
## Multiple R-squared:  0.7715,	Adjusted R-squared:  0.7642 
## F-statistic: 105.2 on 6 and 187 DF,  p-value: < 2.2e-16
```

\[Multiple Regression Interpretation:\]{.ul}

Controlling for all other variables, for every one unit increase in `ei` scores, `physics` scores decrease 0.20 points, *p* \< 0.001.

Rinse and repeat for remaining predictors.
For non-significant predictors, "IQ was not found to be a significant predictor of the model" is sufficient.

**Homoscedasticity**

Scale-Location plot:


```r
# 3 is for the Scale-Location plot. This shows if residuals are spread equally along the ranges of preditors. A straight line means constant variance.
plot(mr_model, 3)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-36-1.png" width="672" />

**Normality of Residuals**

QQ plot:


```r
# 2 is to assess the normality of residuals (QQ plot for the model) We want to see the points on the diagonal. 
plot(mr_model, 2)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-37-1.png" width="672" />
