---
title: "Data Wrangling in R"
date: 2021-12-14
author: 'Dina Arch'
subtitle: A series of functions that could be useful when preparing your dataset for analysis.
weight: 3
draft: false
images: 
series: 
tags:
- r markdown
- machine learning
- visualization
- classification
- principal components
categories:
- r project
editor_options:
  markdown:
    wrap: sentence
links:
- icon: github
  icon_pack: fab
  name: code
  url: https://github.com/dinanajiarch/dina_website/tree/main/content/collection/tips-and-tricks-in-r
---





```r
library(stringr) # str_to_lower()
library(janitor) # clean_names()
library(palmerpenguins) # penguins
library(sjmisc) # move_columns
library(tidyverse)
```


# Data Wrangling Tips in R


Let's use a dataset called `penguins` in the `palmerpenguins` package (Reference: [Horst AM, Hill AP, & Gorman KB, 2020](https://allisonhorst.github.io/palmerpenguins/))

---

## Read in and view data


```r
data("penguins")
head(penguins_raw) # reads first 6 rows
```

```
## # A tibble: 6 × 17
##   studyName `Sample Number` Species          Region Island Stage `Individual ID`
##   <chr>               <dbl> <chr>            <chr>  <chr>  <chr> <chr>          
## 1 PAL0708                 1 Adelie Penguin … Anvers Torge… Adul… N1A1           
## 2 PAL0708                 2 Adelie Penguin … Anvers Torge… Adul… N1A2           
## 3 PAL0708                 3 Adelie Penguin … Anvers Torge… Adul… N2A1           
## 4 PAL0708                 4 Adelie Penguin … Anvers Torge… Adul… N2A2           
## 5 PAL0708                 5 Adelie Penguin … Anvers Torge… Adul… N3A1           
## 6 PAL0708                 6 Adelie Penguin … Anvers Torge… Adul… N3A2           
## # … with 10 more variables: `Clutch Completion` <chr>, `Date Egg` <date>,
## #   `Culmen Length (mm)` <dbl>, `Culmen Depth (mm)` <dbl>,
## #   `Flipper Length (mm)` <dbl>, `Body Mass (g)` <dbl>, Sex <chr>,
## #   `Delta 15 N (o/oo)` <dbl>, `Delta 13 C (o/oo)` <dbl>, Comments <chr>
```

```r
names(penguins_raw) # lists variable names
```

```
##  [1] "studyName"           "Sample Number"       "Species"            
##  [4] "Region"              "Island"              "Stage"              
##  [7] "Individual ID"       "Clutch Completion"   "Date Egg"           
## [10] "Culmen Length (mm)"  "Culmen Depth (mm)"   "Flipper Length (mm)"
## [13] "Body Mass (g)"       "Sex"                 "Delta 15 N (o/oo)"  
## [16] "Delta 13 C (o/oo)"   "Comments"
```

```r
summary(penguins_raw) # provides some descriptive statistics for each variable
```

```
##   studyName         Sample Number      Species             Region         
##  Length:344         Min.   :  1.00   Length:344         Length:344        
##  Class :character   1st Qu.: 29.00   Class :character   Class :character  
##  Mode  :character   Median : 58.00   Mode  :character   Mode  :character  
##                     Mean   : 63.15                                        
##                     3rd Qu.: 95.25                                        
##                     Max.   :152.00                                        
##                                                                           
##     Island             Stage           Individual ID      Clutch Completion 
##  Length:344         Length:344         Length:344         Length:344        
##  Class :character   Class :character   Class :character   Class :character  
##  Mode  :character   Mode  :character   Mode  :character   Mode  :character  
##                                                                             
##                                                                             
##                                                                             
##                                                                             
##     Date Egg          Culmen Length (mm) Culmen Depth (mm) Flipper Length (mm)
##  Min.   :2007-11-09   Min.   :32.10      Min.   :13.10     Min.   :172.0      
##  1st Qu.:2007-11-28   1st Qu.:39.23      1st Qu.:15.60     1st Qu.:190.0      
##  Median :2008-11-09   Median :44.45      Median :17.30     Median :197.0      
##  Mean   :2008-11-27   Mean   :43.92      Mean   :17.15     Mean   :200.9      
##  3rd Qu.:2009-11-16   3rd Qu.:48.50      3rd Qu.:18.70     3rd Qu.:213.0      
##  Max.   :2009-12-01   Max.   :59.60      Max.   :21.50     Max.   :231.0      
##                       NA's   :2          NA's   :2         NA's   :2          
##  Body Mass (g)      Sex            Delta 15 N (o/oo) Delta 13 C (o/oo)
##  Min.   :2700   Length:344         Min.   : 7.632    Min.   :-27.02   
##  1st Qu.:3550   Class :character   1st Qu.: 8.300    1st Qu.:-26.32   
##  Median :4050   Mode  :character   Median : 8.652    Median :-25.83   
##  Mean   :4202                      Mean   : 8.733    Mean   :-25.69   
##  3rd Qu.:4750                      3rd Qu.: 9.172    3rd Qu.:-25.06   
##  Max.   :6300                      Max.   :10.025    Max.   :-23.79   
##  NA's   :2                         NA's   :14        NA's   :13       
##    Comments        
##  Length:344        
##  Class :character  
##  Mode  :character  
##                    
##                    
##                    
## 
```


How to read in data and make note of the missing values:
 

```r
data <- read_csv("data.csv", na = c("-999", "NA", "000")) #note: the `na` argument only works for "read_csv" and not "read_sav."
```

---

## Clean names

We see that the variable names are not "clean." As in, they have spaces and capital letters.
Usually, we want to use a format called "snake case", where we have lower case letters and spaces are replaced with underscores.
We can easily do this with the `clean_names()` function in the `janitor` package:


```r
penguins_new <- penguins_raw %>% 
  clean_names()

#Compare names
names(penguins_new)
```

```
##  [1] "study_name"        "sample_number"     "species"          
##  [4] "region"            "island"            "stage"            
##  [7] "individual_id"     "clutch_completion" "date_egg"         
## [10] "culmen_length_mm"  "culmen_depth_mm"   "flipper_length_mm"
## [13] "body_mass_g"       "sex"               "delta_15_n_o_oo"  
## [16] "delta_13_c_o_oo"   "comments"
```

```r
names(penguins_raw)
```

```
##  [1] "studyName"           "Sample Number"       "Species"            
##  [4] "Region"              "Island"              "Stage"              
##  [7] "Individual ID"       "Clutch Completion"   "Date Egg"           
## [10] "Culmen Length (mm)"  "Culmen Depth (mm)"   "Flipper Length (mm)"
## [13] "Body Mass (g)"       "Sex"                 "Delta 15 N (o/oo)"  
## [16] "Delta 13 C (o/oo)"   "Comments"
```

---

## Move variable names

We can use the `move_columns()` function to rearrange our variable names. Lets say we want to move `clutch_completion` to the left of `body_mass_g`:


```r
penguins_move <- penguins_new %>% 
  move_columns(clutch_completion, .before = body_mass_g)

names(penguins_move)
```

```
##  [1] "study_name"        "sample_number"     "species"          
##  [4] "region"            "island"            "stage"            
##  [7] "individual_id"     "date_egg"          "culmen_length_mm" 
## [10] "culmen_depth_mm"   "flipper_length_mm" "clutch_completion"
## [13] "body_mass_g"       "sex"               "delta_15_n_o_oo"  
## [16] "delta_13_c_o_oo"   "comments"
```

---

## Recoding

### Examine class

Let's examine the `clutch_completion` variable.
(a character string denoting if the study nest observed with a full clutch, i.e., 2 eggs).
Let's see what the labels look like using `unique()`:


```r
unique(penguins_new$clutch_completion)
```

```
## [1] "Yes" "No"
```

Right now, the levels are `Yes` and `No`.
Let's also check the variable type using `class()`:


```r
class(penguins_new$clutch_completion)
```

```
## [1] "character"
```

Let's make this a factor.

---

### Create factor

Since this variable is not a character (but a factor), we need to convert it.
We can do that using `factor()`:


```r
penguins_factor <- penguins_new %>% 
  mutate(clutch_factor = factor(clutch_completion))
```

Let's check the variable again:


```r
class(penguins_factor$clutch_factor)
```

```
## [1] "factor"
```

```r
unique(penguins_factor$clutch_factor)
```

```
## [1] Yes No 
## Levels: No Yes
```

---

### Recoding a factor

Let's say we want to recode our labels from `Yes` and `No` to `1` and `0`:


```r
penguins_recoded <- penguins_factor %>% 
  mutate(clutch_recoded = recode(clutch_factor,"Yes" = "1", "No" = "0"))
```


```r
class(penguins_recoded$clutch_recoded)
```

```
## [1] "factor"
```

```r
unique(penguins_recoded$clutch_recoded)
```

```
## [1] 1 0
## Levels: 0 1
```

---

### Recoding a character into a factor

Let's look at another variable in the `penguins` dataset: `Sex` of the penguin:


```r
class(penguins_new$sex)
```

```
## [1] "character"
```

```r
unique(penguins_new$sex)
```

```
## [1] "MALE"   "FEMALE" NA
```

Let's say we didn't do the process above, and it was still in `character` format.
We can combine those two steps by recoding character variables into factors (from `MALE` and `FEMALE` to `0` and `1`) using `recode_factor():`


```r
penguins_sex_recoded <- penguins_new %>% 
  mutate(sex_recoded = recode_factor(sex,"MALE" = "0", "FEMALE" = "1"))
```

Lets check the variable again:


```r
class(penguins_sex_recoded$sex_recoded)
```

```
## [1] "factor"
```

```r
unique(penguins_sex_recoded$sex_recoded)
```

```
## [1] 0    1    <NA>
## Levels: 0 1
```

*Side Note*.
I also wanted to make a quick note on what happens when you select the variable of interest first vs. not:


```r
penguins_recoded_select <- penguins_new %>% 
  select(sex) %>% 
  mutate(sex_recoded = recode_factor(sex,"MALE" = "0", "FEMALE" = "1"))

head(penguins_recoded_select)
```

```
## # A tibble: 6 × 2
##   sex    sex_recoded
##   <chr>  <fct>      
## 1 MALE   0          
## 2 FEMALE 1          
## 3 FEMALE 1          
## 4 <NA>   <NA>       
## 5 FEMALE 1          
## 6 MALE   0
```

```r
head(penguins_sex_recoded)
```

```
## # A tibble: 6 × 18
##   study_name sample_number species             region island stage individual_id
##   <chr>              <dbl> <chr>               <chr>  <chr>  <chr> <chr>        
## 1 PAL0708                1 Adelie Penguin (Py… Anvers Torge… Adul… N1A1         
## 2 PAL0708                2 Adelie Penguin (Py… Anvers Torge… Adul… N1A2         
## 3 PAL0708                3 Adelie Penguin (Py… Anvers Torge… Adul… N2A1         
## 4 PAL0708                4 Adelie Penguin (Py… Anvers Torge… Adul… N2A2         
## 5 PAL0708                5 Adelie Penguin (Py… Anvers Torge… Adul… N3A1         
## 6 PAL0708                6 Adelie Penguin (Py… Anvers Torge… Adul… N3A2         
## # … with 11 more variables: clutch_completion <chr>, date_egg <date>,
## #   culmen_length_mm <dbl>, culmen_depth_mm <dbl>, flipper_length_mm <dbl>,
## #   body_mass_g <dbl>, sex <chr>, delta_15_n_o_oo <dbl>, delta_13_c_o_oo <dbl>,
## #   comments <chr>, sex_recoded <fct>
```

---

### Recoding multiple variables

The dataset doesn't have any two variables that need to be recoded the same, but let's recode both `sex` and `clutch_completion` at the same time. *Not*e: this code should be used if you have two variables that need to have the same recoding, but it looks like this works too. 


```r
penguins_multiple <- penguins_new %>%  
    mutate_at(c("sex", "clutch_completion"),
              ~ recode_factor(., "MALE" = "0", "FEMALE" = "1",
                       "Yes" = "1", "No" = "0"))
penguins_multiple %>% 
  select(sex, clutch_completion) %>% 
  head()
```

```
## # A tibble: 6 × 2
##   sex   clutch_completion
##   <fct> <fct>            
## 1 0     1                
## 2 1     1                
## 3 1     1                
## 4 <NA>  1                
## 5 1     1                
## 6 0     1
```

---

## Creating reference variables

Recall slide 33 in the Multiple Regression lecture:

![](dummycoding.jpg)

Remember that when we enter a categorical variable into the model using `lm()`, R creates the reference variable for us, and picks the first category as the reference category.
For example, let's look at the `species` variable:


```r
class(penguins_new$species)
```

```
## [1] "character"
```

```r
unique(penguins_new$species)
```

```
## [1] "Adelie Penguin (Pygoscelis adeliae)"      
## [2] "Gentoo penguin (Pygoscelis papua)"        
## [3] "Chinstrap penguin (Pygoscelis antarctica)"
```

Let's clean this variable.
First, let's make the labels lowercase using `str_to_lower()`:


```r
penguin_species <- penguins_new %>% 
  mutate(species = str_to_lower(species))

unique(penguin_species$species)
```

```
## [1] "adelie penguin (pygoscelis adeliae)"      
## [2] "gentoo penguin (pygoscelis papua)"        
## [3] "chinstrap penguin (pygoscelis antarctica)"
```

Now, let's just keep the simple names of the penguins: `adelie`, `gentoo`, and `chinstrap` using the `separate()` function:


```r
penguins_species_clean <- penguin_species %>% 
  separate(species, into = c("species", "delete"), sep = " penguin") %>% 
  select(-delete)

unique(penguins_species_clean$species)
```

```
## [1] "adelie"    "gentoo"    "chinstrap"
```

```r
class(penguins_species_clean$species)
```

```
## [1] "character"
```

Let's also convert this new variable into a factor:


```r
penguins_species_clean$species <- factor(penguins_species_clean$species)

unique(penguins_species_clean$species)
```

```
## [1] adelie    gentoo    chinstrap
## Levels: adelie chinstrap gentoo
```

```r
class(penguins_species_clean$species)
```

```
## [1] "factor"
```

Let's quickly run a simple linear regression using the `species` variable without creating the reference variables.
Let's predict `flipper_length_mm` from `species`:


```r
model_1 <- lm(flipper_length_mm ~ species, data = penguins_species_clean)
summary(model_1)
```

```
## 
## Call:
## lm(formula = flipper_length_mm ~ species, data = penguins_species_clean)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -17.9536  -4.8235   0.0464   4.8130  20.0464 
## 
## Coefficients:
##                  Estimate Std. Error t value Pr(>|t|)    
## (Intercept)      189.9536     0.5405 351.454  < 2e-16 ***
## specieschinstrap   5.8699     0.9699   6.052 3.79e-09 ***
## speciesgentoo     27.2333     0.8067  33.760  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 6.642 on 339 degrees of freedom
##   (2 observations deleted due to missingness)
## Multiple R-squared:  0.7782,	Adjusted R-squared:  0.7769 
## F-statistic: 594.8 on 2 and 339 DF,  p-value: < 2.2e-16
```

Here, it automatically created the reference variables and made `adelie` penguins the reference category.
What if we want to compare our results to `gentoo` penguins?
We can do that by converting the `chinstrap` and `adelie` into their own variables using `ifelse()`


```r
chinstrap <- ifelse(penguins_species_clean$species == 'chinstrap', 1, 0)
adelie <- ifelse(penguins_species_clean$species == 'adelie', 1, 0)
```

Then, we add them to our dataset using `cbind()`:


```r
penguins_reference <- penguins_species_clean %>% 
  cbind(chinstrap, adelie)
```

Let's look at the model again with `gentoo` as the reference variable.
Instead of `species`, we use `chinstrap` and `adelie`:


```r
model_2 <- lm(flipper_length_mm ~ chinstrap + adelie, data = penguins_reference)
summary(model_2)
```

```
## 
## Call:
## lm(formula = flipper_length_mm ~ chinstrap + adelie, data = penguins_reference)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -17.9536  -4.8235   0.0464   4.8130  20.0464 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 217.1870     0.5988  362.68   <2e-16 ***
## chinstrap   -21.3635     1.0036  -21.29   <2e-16 ***
## adelie      -27.2333     0.8067  -33.76   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 6.642 on 339 degrees of freedom
##   (2 observations deleted due to missingness)
## Multiple R-squared:  0.7782,	Adjusted R-squared:  0.7769 
## F-statistic: 594.8 on 2 and 339 DF,  p-value: < 2.2e-16
```

Changing the reference category does NOT change our model.
It only allows us to make a more meaningful comparison.

---

## Collapsing variables 

### Continous to categorical

In lab 1, I go over how to collapse continuous variables into categorical. Let's categorize `body_mass_g` into `1`, `2`, `3`, and `4` levels. 


```r
summary(penguins_new$body_mass_g)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
##    2700    3550    4050    4202    4750    6300       2
```

```r
cont_cat <- penguins_new %>% 
  mutate(body_mass_factor = cut(body_mass_g,
                      breaks = c(2700, 3550, 4202, 4750, 6300),
                      labels = c("low", "medium_low", "medium_high", "high")))
```

---

### Categorical to categorical

Let's collapse our `body_mass_g` even further. This code is useful if you have a Likert-type scale with four categories, such as `Strong Agree` to `Strongly Disagree` and want to collapse to a category with those who responded `Strongly Agree` and `Agree` as `1`, and `Disagree` and `Strongly Disagree` as `2`. We can use `fct_collapse` to collapse factor levels into groups.


```r
cat_cat <- cont_cat %>% 
  mutate(body_mass_collpase = 
           fct_collapse(body_mass_factor,
                        low = c("low", "medium_low"),
                        high = c("medium_high", "high")))
```

---
