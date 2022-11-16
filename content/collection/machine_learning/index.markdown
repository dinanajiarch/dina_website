---
title: "Machine Learning Course Project"
date: 2021-05-06
author: 'Dina Arch'
subtitle: Completion of a final project for PSTAT 231 at UC Santa Barbara.
weight: 4
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
  name: code and dataset
  url: https://github.com/dinanajiarch/dina_website/tree/main/content/collection/machine_learning    
---




# Background

The U.S. presidential election in 2012 did not come as a surprise. Some correctly predicted the outcome of the election correctly including [Nate Silver](https://en.wikipedia.org/wiki/Nate_Silver), 
and [many speculated about his approach](https://www.theguardian.com/science/grrlscientist/2012/nov/08/nate-sliver-predict-us-election).

Despite the success in 2012, the 2016 presidential election came as a 
[big surprise](https://fivethirtyeight.com/features/the-polls-missed-trump-we-asked-pollsters-why/) 
to many, and it underscored that predicting voter behavior is complicated for many reasons despite the tremendous effort in collecting, analyzing, and understanding many available datasets.

Your final project will be to merge census data with 2016 voting data to analyze the election outcome. 

# Data

The `project_data.RData` binary file contains three datasets: tract-level 2010 census data, stored as `census`; metadata `census_meta` with variable descriptions and types; and county-level vote tallies from the 2016 election, stored as `election_raw`.



```r
load('project_data.RData')
```

## Election data

Some example rows of the election data are shown below:



```r
filter(election_raw, !is.na(county)) %>% 
  head()
```

```
## # A tibble: 6 × 5
##   county             fips  candidate       state   votes
##   <chr>              <chr> <chr>           <chr>   <dbl>
## 1 Los Angeles County 6037  Hillary Clinton CA    2464364
## 2 Los Angeles County 6037  Donald Trump    CA     769743
## 3 Los Angeles County 6037  Gary Johnson    CA      88968
## 4 Los Angeles County 6037  Jill Stein      CA      76465
## 5 Los Angeles County 6037  Gloria La Riva  CA      21993
## 6 Cook County        17031 Hillary Clinton IL    1611946
```

The meaning of each column in `election_raw` is self-evident except `fips`. The accronym is short for [Federal Information Processing Standard](https://en.wikipedia.org/wiki/FIPS_county_code). In this dataset, `fips` values denote the area (nationwide, statewide, or countywide) that each row of data represent.

Nationwide and statewide tallies are included as rows in `election_raw` with `county` values of `NA`. There are two kinds of these summary rows:

* Federal-level summary rows have a `fips` value of `US`.
* State-level summary rows have the state name as the `fips` value.

4. Inspect rows with `fips=2000`. Provide a reason for excluding them. Drop these observations -- please write over `election_raw` -- and report the data dimensions after removal. 


```r
election_raw %>% 
  filter(fips == 2000)
```

```
## # A tibble: 6 × 5
##   county fips  candidate          state  votes
##   <chr>  <chr> <chr>              <chr>  <dbl>
## 1 <NA>   2000  Donald Trump       AK    163387
## 2 <NA>   2000  Hillary Clinton    AK    116454
## 3 <NA>   2000  Gary Johnson       AK     18725
## 4 <NA>   2000  Jill Stein         AK      5735
## 5 <NA>   2000  Darrell Castle     AK      3866
## 6 <NA>   2000  Rocky De La Fuente AK      1240
```

**The only reason I see for exclusion is lack of county information and the `fips` value is identified as neither federal level (`fips` value of US) or state-level (`fips` value as the state).**

**Dimensions:**



```r
# Drop observations

election_raw <- election_raw %>% 
  filter(fips != 2000)

dim(election_raw) 
```

```
## [1] 18345     5
```


## Census data

The first few rows and columns of the `census` data are shown below.


```r
census %>% 
  select(1:6) %>% 
  head()
```

```
## # A tibble: 6 × 6
##   CensusTract State   County  TotalPop   Men Women
##         <dbl> <chr>   <chr>      <dbl> <dbl> <dbl>
## 1  1001020100 Alabama Autauga     1948   940  1008
## 2  1001020200 Alabama Autauga     2156  1059  1097
## 3  1001020300 Alabama Autauga     2968  1364  1604
## 4  1001020400 Alabama Autauga     4423  2172  2251
## 5  1001020500 Alabama Autauga    10763  4922  5841
## 6  1001020600 Alabama Autauga     3851  1787  2064
```

Variable descriptions are given in the `metadata` file. The variables shown above are:


```r
census_meta %>% head() 
```

```
## # A tibble: 6 × 3
##   variable       description                     type       
##   <chr>          <chr>                           <chr>      
## 1 "CensusTract " " Census tract ID "             " numeric "
## 2 "State "       " State, DC, or Puerto Rico "   " string " 
## 3 "County "      " County or county equivalent " " string " 
## 4 "TotalPop "    " Total population "            " numeric "
## 5 "Men "         " Number of men "               " numeric "
## 6 "Women "       " Number of women "             " numeric "
```


## Data preprocessing

5. Separate the rows of `election_raw` into separate federal-, state-, and county-level data frames:

    * Store federal-level tallies as `election_federal`.
    
    * Store state-level tallies as `election_state`.
    
    * Store county-level tallies as `election`. Coerce the `fips` variable to numeric.
    
    

```r
election_federal <- election_raw %>% 
  filter(fips == "US")

election_state <- election_raw %>% 
  filter(is.na(county)) %>% 
  filter(fips != "US")

election <- election_raw %>% 
  filter(fips != "US") %>% 
  filter(!is.na(county)) %>% 
  mutate(fips = as.numeric(fips))
```


Federal-Level Tallies (First 5 Rows):


```r
election_federal %>% 
  head()
```

```
## # A tibble: 6 × 5
##   county fips  candidate       state    votes
##   <chr>  <chr> <chr>           <chr>    <dbl>
## 1 <NA>   US    Donald Trump    US    62984825
## 2 <NA>   US    Hillary Clinton US    65853516
## 3 <NA>   US    Gary Johnson    US     4489221
## 4 <NA>   US    Jill Stein      US     1429596
## 5 <NA>   US    Evan McMullin   US      510002
## 6 <NA>   US    Darrell Castle  US      186545
```


State-Level Tallies (First 5 Rows):


```r
election_state %>% 
  head() 
```

```
## # A tibble: 6 × 5
##   county fips  candidate       state   votes
##   <chr>  <chr> <chr>           <chr>   <dbl>
## 1 <NA>   CA    Hillary Clinton CA    8753788
## 2 <NA>   CA    Donald Trump    CA    4483810
## 3 <NA>   CA    Gary Johnson    CA     478500
## 4 <NA>   CA    Jill Stein      CA     278657
## 5 <NA>   CA    Gloria La Riva  CA      66101
## 6 <NA>   FL    Donald Trump    FL    4617886
```


County-Level Tallies (First 5 Rows):


```r
election %>% 
  head() 
```

```
## # A tibble: 6 × 5
##   county              fips candidate       state   votes
##   <chr>              <dbl> <chr>           <chr>   <dbl>
## 1 Los Angeles County  6037 Hillary Clinton CA    2464364
## 2 Los Angeles County  6037 Donald Trump    CA     769743
## 3 Los Angeles County  6037 Gary Johnson    CA      88968
## 4 Los Angeles County  6037 Jill Stein      CA      76465
## 5 Los Angeles County  6037 Gloria La Riva  CA      21993
## 6 Cook County        17031 Hillary Clinton IL    1611946
```
    

6. How many named presidential candidates were there in the 2016 election? Draw a bar graph of all votes received by each candidate, and order the candidate names by decreasing vote counts. (You may need to log-transform the vote axis.)


```r
pres_ct <- election_federal %>% 
  filter(candidate != " None of these candidates") %>% 
  group_by(candidate) %>% 
  mutate(sum = sum(votes)) %>% 
  select(candidate, sum) %>% 
  mutate(candidate = as.factor(candidate))
  

pres_ct %>% 
  ggplot(aes(x = reorder(candidate, sum), y = log(sum), fill = candidate)) +
  geom_bar(colour = "black", stat = "identity") +
  guides(x = guide_axis(angle = 90), fill = FALSE) +
  labs(x = "Candidate", y = "Votes", title = "Vote Counts (Log-Transformed) of Each Presidential Candidate, 2016") +
  coord_flip()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-11-1.png" width="576" style="display: block; margin: auto;" />
**There are 31 named presidential candidates**

7. Create `county_winner` and `state_winner` by taking the candidate with the highest proportion of votes. (Hint: to create `county_winner`, start with `election`, group by `fips`, compute `total` votes, and `pct = votes/total`. Then choose the highest row using `slice_max` (variable `state_winner` is similar).) 


```r
county_winner <- election %>% 
  group_by(fips) %>% 
  mutate(total = sum(votes)) %>% 
  mutate(pct = votes/total) %>% 
  slice_max(pct, n=1)

state_winner <- election_state %>% 
  group_by(fips) %>% 
  mutate(total = sum(votes)) %>% 
  mutate(pct = votes/total) %>% 
  slice_max(pct, n=1)
```



County Winner:


```r
county_winner %>% 
  ungroup() %>% 
  slice_max(pct, n=1)
```

```
## # A tibble: 1 × 7
##   county          fips candidate    state votes total   pct
##   <chr>          <dbl> <chr>        <chr> <dbl> <dbl> <dbl>
## 1 Roberts County 48393 Donald Trump TX      524   550 0.953
```


State Winner:


```r
state_winner %>% 
  ungroup() %>% 
  slice_max(pct, n=1)
```

```
## # A tibble: 1 × 7
##   county fips  candidate       state  votes  total   pct
##   <chr>  <chr> <chr>           <chr>  <dbl>  <dbl> <dbl>
## 1 <NA>   DC    Hillary Clinton DC    282830 304717 0.928
```

# Visualization


Here you'll generate maps of the election data using `ggmap`. The .Rmd file for this document contains codes to generate the following map.



```r
states <- map_data("state")

ggplot(states) + 
  geom_polygon(aes(x = long, 
                   y = lat, 
                   fill = region, 
                   group = group), 
               color = "white") + 
  coord_fixed(1.3) + # avoid stretching
  guides(fill=FALSE) + # no fill legend
  theme_nothing() # no axes
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-15-1.png" width="576" style="display: block; margin: auto;" />

8. Draw a county-level map with `map_data("county")` and color by county.



```r
counties <- map_data("county")

ggplot(counties) + 
  geom_polygon(aes(x = long, 
                   y = lat, 
                   fill = region, 
                   group = group), 
               color = "white") + 
  coord_fixed(1.3) + # avoid stretching
  guides(fill=FALSE) + # no fill legend
  theme_nothing() # no axes
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-16-1.png" width="576" style="display: block; margin: auto;" />


In order to map the winning candidate for each state, the map data (`states`) must be merged with with the election data (`state_winner`).

The function `left_join()` will do the trick, but needs to join the data frames on a variable with values that match. In this case, that variable is the state name, but abbreviations are used in one data frame and the full name is used in the other.

9. Use the following function to create a `fips` variable in the `states` data frame with values that match the `fips` variable in `election_federal`.

```r
name2abb <- function(statename){
  ix <- match(statename, tolower(state.name))
  out <- state.abb[ix]
  return(out)
}
states <- states %>% 
  mutate(fips = name2abb(region))
```


First 5 rows in the `states` data frame:


```r
states %>% 
  head() 
```

```
##        long      lat group order  region subregion fips
## 1 -87.46201 30.38968     1     1 alabama      <NA>   AL
## 2 -87.48493 30.37249     1     2 alabama      <NA>   AL
## 3 -87.52503 30.37249     1     3 alabama      <NA>   AL
## 4 -87.53076 30.33239     1     4 alabama      <NA>   AL
## 5 -87.57087 30.32665     1     5 alabama      <NA>   AL
## 6 -87.58806 30.32665     1     6 alabama      <NA>   AL
```


 - Now the data frames can be merged. `left_join(df1, df2)` takes all the rows from `df1` and looks for matches in `df2`. For each match, `left_join()` appends the data from the second table to the matching row in the first; if no matching value is found, it adds missing values.

10. Use `left_join` to merge the tables and use the result to create a map of the election results by state. Your figure will look similar to this state level [New York Times map](https://www.nytimes.com/elections/results/president). (Hint: use `scale_fill_brewer(palette="Set1")` for a red-and-blue map.)


Merged states by `fips` (First 5 rows):

```r
merged_states <- left_join(states, state_winner) 

ggplot(merged_states) + 
  geom_polygon(aes(x = long, 
                   y = lat, 
                   fill = candidate, 
                   group = group), 
               color = "Black") + 
  coord_fixed(1.3) + # avoid stretching
  guides(fill=FALSE) + # no fill legend
  theme_nothing() + # no axes
  scale_fill_brewer(palette="Set1") 
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-19-1.png" width="576" style="display: block; margin: auto;" />

11. Now create a county-level map. The county-level map data does not have a `fips` value, so to create one, use information from `maps::county.fips`: split the `polyname` column to `region` and `subregion` using `tidyr::separate`, and use `left_join()` to combine `county.fips` with the county-level map data. Then construct the map. Your figure will look similar to county-level [New York Times map](https://www.nytimes.com/elections/results/president).



```r
county_fips <- maps::county.fips %>% 
  tidyr::separate(polyname, c("region", "subregion"), sep = ",")

counties <- left_join(counties, county_fips)

county_map <- left_join(counties, county_winner)

ggplot(county_map) + 
  geom_polygon(aes(x = long, 
                   y = lat, 
                   fill = candidate, 
                   group = group), 
               color = "white") + 
  coord_fixed(1.3) + # avoid stretching
  guides(fill=FALSE) + # no fill legend
  theme_nothing() + # no axes
  scale_fill_brewer(palette="Set1") 
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-20-1.png" width="576" style="display: block; margin: auto;" />

  
12. Create a visualization of your choice using `census` data. Many exit polls noted that [demographics played a big role in the election](https://fivethirtyeight.com/features/demographics-not-hacking-explain-the-election-results/). If you need a starting point, use [this Washington Post article](https://www.washingtonpost.com/graphics/politics/2016-election/exit-polls/) and [this R graph gallery](https://www.r-graph-gallery.com/) for ideas and inspiration.


```r
census_12 <- census %>% 
  select(State, Men, Women, TotalPop, Income, Unemployment, Hispanic, White, Black) %>%
  group_by(State) %>% 
  summarise(across(Men:Income, ~ sum(.x, na.rm = TRUE)),
            across(Unemployment:Black, ~ mean(.x, na.rm = TRUE)))

p <- census_12 %>% 
  mutate(State = as.factor(State)) %>% 
  group_by(State) %>% 
  ggplot(aes(x = Unemployment, y = Income, size = TotalPop, fill = State, colour = State)) + 
  geom_point(show.legend = FALSE) +
  geom_text(label=census_12$State, 
            nudge_x = 0.25, 
            nudge_y = 0.25, 
            check_overlap = F,
            colour = "Black")

p 
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-21-1.png" width="1152" style="display: block; margin: auto;" />

    
13. The `census` data contains high resolution information (more fine-grained than county-level). Aggregate the information into county-level data by computing population-weighted averages of each attribute for each county by carrying out the following steps:
    
* Clean census data, saving the result as `census_del`: 
  
   + filter out any rows of `census` with missing values;
   + convert `Men`, `Employed`, and `Citizen` to percentages;
   + compute a `Minority` variable by combining `Hispanic`, `Black`, `Native`, `Asian`, `Pacific`, and remove these variables after creating `Minority`; and
   + remove `Walk`, `PublicWork`, and `Construction`.
 
 

```r
census_del <- census %>% 
  drop_na() %>% 
  mutate(Men = (Men/TotalPop)*100,
         Citizen = (Citizen/TotalPop)*100, 
         Employed = (Employed/TotalPop)*100) %>%
  mutate(Minority = rowSums(across(c(Hispanic, Black, Native, Asian, Pacific)))) %>% 
  select(-Hispanic, -Black, -Native, -Asian, -Pacific, -Walk, -PublicWork, -Construction)
```


* Create population weights for sub-county census data, saving the result as `census_subct`: 
    + group `census_del` by `State` and `County`;
    + use `add_tally()` to compute `CountyPop`; 
    + compute the population weight as `TotalPop/CountyTotal`;
    + adjust all quantitative variables by multiplying by the population weights.
    

```r
census_subct <- census_del %>% 
  group_by(State, County) %>% 
  mutate(CountyPop = sum(TotalPop)) %>% 
  mutate(PopWeight = TotalPop/CountyPop) %>% 
  mutate(across(Men:Minority, ~.x*PopWeight))
```

    
* Aggregate census data to county level, `census_ct`: group the sub-county data `census_subct` by state and county and compute popluation-weighted averages of each variable by taking the sum (since the variables were already transformed by the population weights)


```r
census_ct <- census_subct %>% 
  group_by(State, County) %>% 
  summarise(across(Men:Minority, sum))
```
    
* Print the first few rows and columns of `census_ct`. 


```r
census_ct %>% 
  head()
```

```
## # A tibble: 6 × 28
## # Groups:   State [1]
##   State   County    Men Women White Citizen Income IncomeErr IncomePerCap
##   <chr>   <chr>   <dbl> <dbl> <dbl>   <dbl>  <dbl>     <dbl>        <dbl>
## 1 Alabama Autauga  48.4 3349.  75.8    73.7 51696.     7771.       24974.
## 2 Alabama Baldwin  48.8 3934.  83.1    75.7 51074.     8745.       27317.
## 3 Alabama Barbour  53.8 1492.  46.2    76.9 32959.     6031.       16824.
## 4 Alabama Bibb     53.4 2930.  74.5    77.4 38887.     5662.       18431.
## 5 Alabama Blount   49.4 3562.  87.9    73.4 46238.     8696.       20532.
## 6 Alabama Bullock  53.0 1968.  22.2    75.5 33293.     9000.       17580.
## # … with 19 more variables: IncomePerCapErr <dbl>, Poverty <dbl>,
## #   ChildPoverty <dbl>, Professional <dbl>, Service <dbl>, Office <dbl>,
## #   Production <dbl>, Drive <dbl>, Carpool <dbl>, Transit <dbl>,
## #   OtherTransp <dbl>, WorkAtHome <dbl>, MeanCommute <dbl>, Employed <dbl>,
## #   PrivateWork <dbl>, SelfEmployed <dbl>, FamilyWork <dbl>,
## #   Unemployment <dbl>, Minority <dbl>
```


14. If you were physically located in the United States on election day for the 2016 presidential election, what state and county were you in? Compare and contrast the results and demographic information for this county with the state it is located in. If you were not in the United States on election day, select any county. Do you find anything unusual or surprising? If so, explain; if not, explain why not.

**On election day, I was in Los Angeles, CA. Taking the average values of some of the variables to look at overall California demographics, it looks like minorities make up 70% of LA, and 40% of California. Men and Unemployement is unchanged. Having lived here a long time, I am not surprised at the demographics. LA is much more diverse than the rest of California.** 


```r
census_ct %>% 
  filter(County == "Los Angeles")
```

```
## # A tibble: 1 × 28
## # Groups:   State [1]
##   State      County        Men Women White Citizen Income IncomeErr IncomePerCap
##   <chr>      <chr>       <dbl> <dbl> <dbl>   <dbl>  <dbl>     <dbl>        <dbl>
## 1 California Los Angeles  49.2 2460.  26.9    60.1 61385.    10572.       28390.
## # … with 19 more variables: IncomePerCapErr <dbl>, Poverty <dbl>,
## #   ChildPoverty <dbl>, Professional <dbl>, Service <dbl>, Office <dbl>,
## #   Production <dbl>, Drive <dbl>, Carpool <dbl>, Transit <dbl>,
## #   OtherTransp <dbl>, WorkAtHome <dbl>, MeanCommute <dbl>, Employed <dbl>,
## #   PrivateWork <dbl>, SelfEmployed <dbl>, FamilyWork <dbl>,
## #   Unemployment <dbl>, Minority <dbl>
```

```r
census_ct %>% 
  filter(State == "California") %>%
  summarise(across(Men:Minority, mean)) 
```

```
## # A tibble: 1 × 27
##   State    Men Women White Citizen Income IncomeErr IncomePerCap IncomePerCapErr
##   <chr>  <dbl> <dbl> <dbl>   <dbl>  <dbl>     <dbl>        <dbl>           <dbl>
## 1 Calif…  50.4 2665.  55.7    68.2 59031.     9828.       28014.           3977.
## # … with 18 more variables: Poverty <dbl>, ChildPoverty <dbl>,
## #   Professional <dbl>, Service <dbl>, Office <dbl>, Production <dbl>,
## #   Drive <dbl>, Carpool <dbl>, Transit <dbl>, OtherTransp <dbl>,
## #   WorkAtHome <dbl>, MeanCommute <dbl>, Employed <dbl>, PrivateWork <dbl>,
## #   SelfEmployed <dbl>, FamilyWork <dbl>, Unemployment <dbl>, Minority <dbl>
```



# Exploratory analysis

15. Carry out PCA for both county & sub-county level census data. Compute the first two principal components PC1 and PC2 for both county and sub-county respectively. Discuss whether you chose to center and scale the features and the reasons for your choice. Examine and interpret the loadings.


County Level: 

```r
# County Level Census

# Center and scale the data
county_std <- census_ct %>% 
  as.data.frame() %>% 
  select(-c('State','County')) %>% 
  scale(center = T, scale = T)


# Compute SVD
x_svd <- svd(county_std)

# Get loadings
v_svd <- x_svd$v

# compute principal components
county_pc <- county_std %*% v_svd
colnames(county_pc) <- paste('PC', 1:26, sep = '')

# scatterplot
as_tibble(county_pc[,1:2]) %>%
  bind_cols(select(census_ct, County)) %>% 
  ggplot(aes(x = PC1, y = PC2)) +
  geom_point(aes(color = County)) +
  theme_bw() +
  theme(legend.position="none")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-27-1.png" width="576" style="display: block; margin: auto;" />

```r
v_svd[, 1:2] %>%
  as.data.frame() %>%
  mutate(variable = colnames(county_std)) %>%
  gather(key = 'PC', value = 'Loading', 1:2) %>%
  arrange(variable) %>%
  ggplot(aes(x = variable, y = Loading)) +
  geom_point(aes(shape = PC)) +
  theme_bw() +
  geom_hline(yintercept = 0, color = 'blue') +
  geom_path(aes(linetype = PC, group = PC)) +
  theme(axis.text.x = element_text(angle = 90)) +
  labs(x = '') 
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-27-2.png" width="576" style="display: block; margin: auto;" />

```r
v_svd[, 1:2] %>%
  as.data.frame() %>%
  mutate(variable = colnames(county_std)) %>%
  gather(key = 'PC', value = 'Loading', 1:2) %>%
  arrange(variable) %>%
  ggplot(aes(x = variable, y = Loading)) +
  geom_point(aes(shape = PC)) +
  theme_bw() +
  geom_hline(yintercept = 0, color = 'blue') +
  geom_path(aes(linetype = PC, group = PC)) +
  theme(axis.text.x = element_text(angle = 90)) +
  labs(x = '') +
  facet_wrap(~PC)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-27-3.png" width="576" style="display: block; margin: auto;" />

**The first PC has a high percentage of children under poverty level, a high percentage of people under poverty level, and high percentage of those unemployed. The first PC also has low income per capital, low percentage of those employed (16+), and low median household income. I would label this principal component as *Under Poverty Level*.**


**The second PC has a high percentage of those who are self-employed, a high percentage of those working at home, and high percentage of those working in unpaid family work. The second PC also has low number of women, low percentage of working in private industry, and low percentage of employed in office jobs. I would label this principal component as *Working Class - Family Oriented*.**


Sub-county Level:


```r
# Sub-County Level Census

# Center and scale the data
subcounty_std <- census_subct %>% 
  as.data.frame() %>% 
  select(-c('State','County','CensusTract', 'PopWeight', 'TotalPop', 'CountyPop')) %>% 
  scale(center = T, scale = T)


# Compute SVD
x_svd2 <- svd(subcounty_std)

# Get loadings
v_svd2 <- x_svd2$v

# compute principal components
subcounty_pc <- subcounty_std %*% v_svd2
colnames(subcounty_pc) <- paste('PC', 1:26, sep = '')

# scatterplot
as_tibble(subcounty_pc[,1:2]) %>%
  bind_cols(select(census_subct, County)) %>% 
  ggplot(aes(x = PC1, y = PC2)) +
  geom_point(aes(color = County)) +
  theme_bw() +
  theme(legend.position="none")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-28-1.png" width="576" style="display: block; margin: auto;" />

```r
v_svd2[, 1:2] %>%
  as.data.frame() %>%
  mutate(variable = colnames(subcounty_std)) %>%
  gather(key = 'PC', value = 'Loading', 1:2) %>%
  arrange(variable) %>%
  ggplot(aes(x = variable, y = Loading)) +
  geom_point(aes(shape = PC)) +
  theme_bw() +
  geom_hline(yintercept = 0, color = 'blue') +
  geom_path(aes(linetype = PC, group = PC)) +
  theme(axis.text.x = element_text(angle = 90)) +
  labs(x = '') 
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-28-2.png" width="576" style="display: block; margin: auto;" />

```r
v_svd2[, 1:2] %>%
  as.data.frame() %>%
  mutate(variable = colnames(subcounty_std)) %>%
  gather(key = 'PC', value = 'Loading', 1:2) %>%
  arrange(variable) %>%
  ggplot(aes(x = variable, y = Loading)) +
  geom_point(aes(shape = PC)) +
  theme_bw() +
  geom_hline(yintercept = 0, color = 'blue') +
  geom_path(aes(linetype = PC, group = PC)) +
  theme(axis.text.x = element_text(angle = 90)) +
  labs(x = '') +
  facet_wrap(~PC)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-28-3.png" width="576" style="display: block; margin: auto;" />

```r
data <- v_svd2[, 1:2] %>%
  as.data.frame() %>% 
  mutate(variable = colnames(subcounty_std))
```

**This PC is interesting because all 26 variables are positive and relatively high, where most have loadings greater than .20. The first PC has a high percentage of Men, a high number of citizens, high percentage of those employed in private industry, high percentage of those who commute alone in a car, and a high percentage of those employed (+16). I would label this principal component as *Working-Class*.** 


**The second PC has a high percentage of minorities, a high percentage of unemployment, and high percentage of child poverty and under poverty level. The second PC also has low percentage of those who work from home, low percentage of inpaid family work, and low percentage of those who are self-employed. I would label this principal component as *Under Poverty Level*.** 

**It's usually important to both center and scale your variables as they are most likely not on the same metric. As in this case, I center and scaled the dataset because some variables are count and others are percentages. Either way, it doesn't hurt.**



16. Determine the minimum number of PCs needed to capture 90% of the variance for both the county and sub-county analyses. Plot the proportion of variance explained and cumulative variance explained for both county and sub-county analyses.

County Level:


```r
# scree and cumulative variance plots
pc_vars <- x_svd$d^2/(nrow(county_std) - 1)

tibble(PC = 1:min(dim(county_std)),
       Proportion = pc_vars/sum(pc_vars),
       Cumulative = cumsum(Proportion)) %>%
  gather(key = 'measure', value = 'Variance Explained', 2:3) %>%
  ggplot(aes(x = PC, y = `Variance Explained`)) +
  geom_point() +
  geom_path() +
  facet_wrap(~ measure) +
  theme_bw() 
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-29-1.png" width="576" style="display: block; margin: auto;" />

```r
#tibble(PC = 1:min(dim(county_std)),
#       Proportion = pc_vars/sum(pc_vars),
#       Cumulative = cumsum(Proportion)) %>% 
#  head(15) %>% 
#  pander()
```

**At least *13* PCs are needed to capture 90% of the variance for the county analysis.**



Sub-County Level:


```r
# scree and cumulative variance plots
pc_vars2 <- x_svd2$d^2/(nrow(subcounty_std) - 1)

tibble(PC = 1:min(dim(subcounty_std)),
       Proportion = pc_vars2/sum(pc_vars2),
       Cumulative = cumsum(Proportion)) %>%
  gather(key = 'measure', value = 'Variance Explained', 2:3) %>%
  ggplot(aes(x = PC, y = `Variance Explained`)) +
  geom_point() +
  geom_path() +
  facet_wrap(~ measure) +
  theme_bw() 
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-30-1.png" width="576" style="display: block; margin: auto;" />

```r
#tibble(PC = 1:min(dim(subcounty_std)),
#       Proportion = pc_vars2/sum(pc_vars2),
#       Cumulative = cumsum(Proportion)) %>% 
#  head(10) %>% 
#  pander()
```
**At least *7* PCs are needed to capture 90% of the variance for the sub-county analysis.**

17. With `census_ct`, perform hierarchical clustering with complete linkage.  Cut the tree to partition the observations into 10 clusters. Re-run the hierarchical clustering algorithm using the first 5 principal components the county-level data as inputs instead of the original features. Compare and contrast the results. For both approaches investigate the cluster that contains San Mateo County. Which approach seemed to put San Mateo County in a more appropriate cluster? Comment on what you observe and discuss possible explanations for these observations.


Hierarchical Clustering:

```r
library(ggridges)
library(dendextend)
# compute distances between points
d_mx <- dist(county_std, method = 'euclidean')

# compute hierarchical clustering
hclust_out <- hclust(d_mx, method = 'complete')

# cut at 10 clusters
clusters <- cutree(hclust_out, k = 10) %>% 
  factor(labels = paste('cluster', 1:10))

# count number of data points per cluster
tibble(clusters) %>% 
  count(clusters) 
```

```
## # A tibble: 10 × 2
##    clusters       n
##    <fct>      <int>
##  1 cluster 1   2307
##  2 cluster 2    779
##  3 cluster 3     67
##  4 cluster 4      8
##  5 cluster 5     20
##  6 cluster 6      4
##  7 cluster 7      3
##  8 cluster 8      3
##  9 cluster 9     23
## 10 cluster 10     4
```

Cluster for San Mateo County:

```r
#filter San Mateo County
san_mateo <- which(grepl('San Mateo', census_ct$County))
clusters[san_mateo]
```

```
## [1] cluster 1
## 10 Levels: cluster 1 cluster 2 cluster 3 cluster 4 cluster 5 ... cluster 10
```


Hierarchical Clustering for first five PCs:

```r
# First 5 PC

# compute distances between points
d_mx <- dist(county_pc[,1:5], method = 'euclidean')

# compute hierarchical clustering
hclust_out <- hclust(d_mx, method = 'complete')

# cut at 10 clusters
clusters <- cutree(hclust_out, k = 10) %>% 
  factor(labels = paste('cluster', 1:10))

# count number of data points per cluster
tibble(clusters) %>% 
  count(clusters)
```

```
## # A tibble: 10 × 2
##    clusters       n
##    <fct>      <int>
##  1 cluster 1   1950
##  2 cluster 2    631
##  3 cluster 3    312
##  4 cluster 4     57
##  5 cluster 5      8
##  6 cluster 6    167
##  7 cluster 7     43
##  8 cluster 8      7
##  9 cluster 9     40
## 10 cluster 10     3
```

Cluster for San Mateo County:

```r
#filter San Mateo County
san_mateo <- which(grepl('San Mateo', census_ct$County))
clusters[san_mateo] 
```

```
## [1] cluster 3
## 10 Levels: cluster 1 cluster 2 cluster 3 cluster 4 cluster 5 ... cluster 10
```

**It seems that the counties in the PC cluster are more evenly spread out than using the entire dataset. When using the first 5 principal components, we are capturing 64% of the variance in the county dataset with only five variables. Because of the smaller dimension, it may be more effective in clustering the counties as opposed to using 26 variables with the full dataset. It makes sense that San Mateo county is put into cluster 1 with the full dataset as most counties are grouped there. Thus, we may want to rely on the cluster that used the PCs, where San Mateo is in cluster 3.** 

# Classification

In order to train classification models, we need to combine `county_winner` and `census_ct` data. This seemingly straightforward task is harder than it sounds. Codes are provided in the .Rmd file that make the necessary changes to merge them into `election_cl` for classification.


```r
abb2name <- function(stateabb){
  ix <- match(stateabb, state.abb)
  out <- tolower(state.name[ix])
  return(out)
}


tmpwinner <- county_winner %>%
  ungroup %>%
  # coerce names to abbreviations
  mutate(state = abb2name(state)) %>%
  # everything lower case
  mutate(across(c(state, county), tolower)) %>%
  # remove county suffixes
  mutate(county = gsub(" county| columbia| city| parish", 
                       "", 
                       county)) 

tmpcensus <- census_ct %>% 
  ungroup %>% 
  mutate(across(c(State, County), tolower))

election_county <- tmpwinner %>%
  left_join(tmpcensus, 
            by = c("state"="State", "county"="County")) %>% 
  na.omit()

## save meta information
election_meta <- election_county %>% 
  select(c(county, fips, state, votes, pct, total))

## save predictors and class labels
election_county <- election_county %>% 
  select(-c(county, fips, state, votes, pct, total))
```

After merging the data, partition the result into 80% training and 20% testing partitions.


```r
# hold out 20% of data as a test set
set.seed(12521)
county_part <- resample_partition(election_county, c(test = 0.2, train = 0.8))
train <- as_tibble(county_part$train)
test <- as_tibble(county_part$test)
```

18. Decision tree: train a decision tree on the training partition, and apply cost-complexity pruning. Visualize the tree before and after pruning. Estimate the misclassification errors on the test partition, and interpret and discuss the results of the decision tree analysis. Use your plot to tell a story about voting behavior in the US (see this [NYT infographic](https://archive.nytimes.com/www.nytimes.com/imagepages/2008/04/16/us/20080416_OBAMA_GRAPHIC.html)).


```r
#Train a decision tree - training partition
nmin <- 5
tree_opts <- tree.control(nobs = nrow(train), 
                          minsize = nmin, 
                          mindev = exp(-6))
t_small <- tree(as.factor(candidate) ~ ., data = train,
                control = tree_opts, split = 'deviance') 
summary(t_small)
```

```
## 
## Classification tree:
## tree(formula = as.factor(candidate) ~ ., data = train, control = tree_opts, 
##     split = "deviance")
## Variables actually used in tree construction:
##  [1] "White"           "Men"             "Unemployment"    "IncomePerCapErr"
##  [5] "MeanCommute"     "Service"         "Women"           "Citizen"        
##  [9] "IncomePerCap"    "Drive"           "Transit"         "Office"         
## [13] "Production"      "SelfEmployed"    "Income"          "WorkAtHome"     
## [17] "Poverty"         "Employed"        "Carpool"         "Professional"   
## [21] "PrivateWork"     "OtherTransp"     "FamilyWork"     
## Number of terminal nodes:  84 
## Residual mean deviance:  0.04184 = 99.29 / 2373 
## Misclassification error rate: 0.008547 = 21 / 2457
```

Tree before pruning:


```r
draw.tree(t_small, cex = 0.5, size = 2)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-38-1.png" width="960" style="display: block; margin: auto;" />

Add cost-complexity pruning:


```r
# cost-complexity pruning
nfolds <- 10
cv_out <- cv.tree(t_small, K = nfolds, method = 'deviance')

# convert to tibble
cv_df <- tibble(alpha = cv_out$k,
                impurity = cv_out$dev,
                size = cv_out$size)

# choose optimal alpha
best_alpha <- slice_min(cv_df, impurity) %>%
  slice_min(size)

# select final tree
t_opt <- prune.tree(t_small, k = best_alpha$alpha)
summary(t_opt) 
```

```
## 
## Classification tree:
## snip.tree(tree = t_small, nodes = c(4L, 14L, 5L, 15L, 6L))
## Variables actually used in tree construction:
## [1] "White"        "Men"          "Transit"      "Professional"
## Number of terminal nodes:  5 
## Residual mean deviance:  0.4521 = 1108 / 2452 
## Misclassification error rate: 0.0814 = 200 / 2457
```

Tree after pruning:


```r
draw.tree(t_opt, cex = 0.8, size = 2)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-40-1.png" width="576" style="display: block; margin: auto;" />


Classification Errors on Test Partition - Before Pruning: 


```r
# predict on test data
preds_test_t0 <- predict(t_small, newdata = county_part$test, type = 'class')
# examin misclassification errors on test set - First Tree
classes_test <- as.data.frame(county_part$test) %>% pull(candidate)
test_errors_t0 <- table(class = classes_test, pred = preds_test_t0)
(test_errors_t0/rowSums(test_errors_t0)) 
```

```
##                  pred
## class             Donald Trump Hillary Clinton
##   Donald Trump       0.9496124       0.0503876
##   Hillary Clinton    0.3505155       0.6494845
```

Classification Errors on Test Partition - After Pruning:


```r
# predict on test data
preds_test_t0 <- predict(t_opt, newdata = county_part$test, type = 'class')
# Pruned - Tree Error
classes_test <- as.data.frame(county_part$test) %>% pull(candidate)
test_errors_t0 <- table(class = classes_test, pred = preds_test_t0)
(test_errors_t0/rowSums(test_errors_t0)) 
```

```
##                  pred
## class             Donald Trump Hillary Clinton
##   Donald Trump      0.94379845      0.05620155
##   Hillary Clinton   0.44329897      0.55670103
```


**After training a decision tree before pruning, the resulting classification tree had 84 terminal nodes and a misclassification error rate of 0.0085. There were 23 variables used. After pruning the classification tree, now there are 5 terminal nodes and a misclassifcation error rate of 0.0814. There were four variables used: White, Men, Transit, Professional. Similar to the infographic, it's not surprising that "White" is used as the first splitting variable and Men is one of the second spliting variables. Counties that are less than 49.969% White and less than 49.6965 Male were likely to vote for Hillary Clinton. Which makes sense given the demographics of the election results, however it isn't very telling as splitting on almost 50% isn't much information. On the other side of the decision tree, counties that are more than almost 50% white, more than 1.138% use public transportation, and more than 35.4651% are employed in management positions voted for Hillary. It's interesting that counties that have more than 1.138% of people using public transit are likely to vote Trump. It may indicate that the model needs some adjustments since the misclassification error rate is higher for this model.**


19. Train a logistic regression model on the training partition to predict the winning candidate in each county and estimate errors on the test partition. What are the significant variables? Are these consistent with what you observed in the decision tree analysis? Interpret the meaning of one or two significant coefficients of your choice in terms of a unit change in the variables. Did the results in your particular county (from question 14) match the predicted results?  

 

```r
fit_glm <- glm(as.factor(candidate) ~ ., family = 'binomial', data=train)

summary(fit_glm)
```

```
## 
## Call:
## glm(formula = as.factor(candidate) ~ ., family = "binomial", 
##     data = train)
## 
## Deviance Residuals: 
##     Min       1Q   Median       3Q      Max  
## -3.9700  -0.2610  -0.1094  -0.0414   3.5310  
## 
## Coefficients:
##                   Estimate Std. Error z value Pr(>|z|)    
## (Intercept)     -1.230e+01  9.576e+00  -1.284 0.199042    
## Men             -3.202e-02  5.466e-02  -0.586 0.558020    
## Women            4.849e-05  1.675e-04   0.289 0.772235    
## White           -1.876e-01  6.417e-02  -2.924 0.003453 ** 
## Citizen          1.139e-01  2.934e-02   3.881 0.000104 ***
## Income          -5.835e-05  2.752e-05  -2.120 0.033970 *  
## IncomeErr        4.269e-05  6.807e-05   0.627 0.530554    
## IncomePerCap     1.900e-04  6.601e-05   2.879 0.003991 ** 
## IncomePerCapErr -2.421e-04  1.405e-04  -1.723 0.084840 .  
## Poverty          4.498e-02  4.267e-02   1.054 0.291820    
## ChildPoverty    -4.893e-03  2.611e-02  -0.187 0.851348    
## Professional     2.568e-01  3.852e-02   6.665 2.64e-11 ***
## Service          3.509e-01  4.974e-02   7.055 1.73e-12 ***
## Office           1.209e-01  4.660e-02   2.595 0.009448 ** 
## Production       1.849e-01  4.262e-02   4.337 1.44e-05 ***
## Drive           -2.236e-01  4.639e-02  -4.821 1.43e-06 ***
## Carpool         -2.164e-01  6.138e-02  -3.525 0.000423 ***
## Transit          4.788e-02  9.227e-02   0.519 0.603815    
## OtherTransp     -9.843e-02  1.018e-01  -0.967 0.333650    
## WorkAtHome      -1.356e-01  7.447e-02  -1.821 0.068677 .  
## MeanCommute      5.417e-02  2.500e-02   2.167 0.030224 *  
## Employed         1.960e-01  3.376e-02   5.807 6.36e-09 ***
## PrivateWork      7.211e-02  2.230e-02   3.233 0.001224 ** 
## SelfEmployed     1.535e-02  4.992e-02   0.308 0.758415    
## FamilyWork      -8.790e-01  4.130e-01  -2.128 0.033336 *  
## Unemployment     1.742e-01  4.075e-02   4.275 1.91e-05 ***
## Minority        -6.081e-02  6.186e-02  -0.983 0.325579    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 2071.81  on 2456  degrees of freedom
## Residual deviance:  841.25  on 2430  degrees of freedom
## AIC: 895.25
## 
## Number of Fisher Scoring iterations: 7
```

Errors (Test Partition):


```r
# compute estimated probabilities
p_hat_glm <- predict(fit_glm, test, type = 'response')

# bayes classifier
y_hat_glm <- factor(p_hat_glm > 0.5, labels = c('No', 'Yes'))

# errors
error_glm <- table(test$candidate, y_hat_glm)
(error_glm / rowSums(error_glm)) 
```

```
##                  y_hat_glm
##                           No        Yes
##   Donald Trump    0.95930233 0.04069767
##   Hillary Clinton 0.32989691 0.67010309
```
**Significant predictors of candidate are White, Citizen, Income, IncomePerCap, Professional, Service, Office, Production, Drive, Carpool, MeanCommute, Employed, PrivateWork, FamilyWork, and Unemployement. Only one of the variables overlap with the variables chosen in the pruned decision tree which is inconsistent.** 

**The probability of a county voting for Donald Trump increases by an estimated 0.1139 among counties with Citizens, after accounting for all other predictors.**

**The probability of a county voting for Donald Trump decreases by an estimated -.879 among counties with unpaid family workers, after accounting for all other predictors.**

**Yes, the results in Los Angeles county (Hillary Clinton) matched the predicted results.**

20.  Compute ROC curves for the decision tree and logistic regression using predictions on the test data, and display them on the same plot. Based on your classification results, discuss the pros and cons of each method. Are the different classifiers more appropriate for answering different kinds of questions about the election?


Decision Tree: 


```r
#Decision Tree
t_predict <- predict(t_small, newdata = test) %>% 
  as_tibble()

tree_predict <- prediction(t_predict[,2], labels = test$candidate)

# compute error rates as a function of the probability threshold
perf_log <- performance(prediction.obj = tree_predict, 'tpr', 'fpr')

# extract rates and threshold from perf_lda as a tibble
rates_log <- tibble(fpr = perf_log@x.values,
                    tpr = perf_log@y.values,
                    thresh = perf_log@alpha.values) %>%
  unnest(everything())

# compute youden's stat
rates_log <- rates_log %>%
  mutate(youden = tpr - fpr)

# find the optimal value
optimal_thresh <- rates_log %>%
  slice_max(youden)

# ROC curve
rates_log %>%
  ggplot(aes(x = fpr, y = tpr, color = thresh)) +
  geom_path() +
  geom_point(data = optimal_thresh, color = "red") +
  theme_bw()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-45-1.png" width="576" style="display: block; margin: auto;" />


Logistic Regression:


```r
#Logistic Regression

# create prediction object
spam_predict <- prediction(predictions = p_hat_glm, labels = test$candidate)

# compute error rates as a function of the probability threshold
perf_log <- performance(prediction.obj = spam_predict, 'tpr', 'fpr')

# extract rates and threshold from perf_lda as a tibble
rates_log <- tibble(fpr = perf_log@x.values,
                    tpr = perf_log@y.values,
                    thresh = perf_log@alpha.values) %>%
  unnest(everything())

# compute youden's stat
rates_log <- rates_log %>%
  mutate(youden = tpr - fpr)

# find the optimal value
optimal_thresh <- rates_log %>%
  slice_max(youden)

# ROC curve
rates_log %>%
  ggplot(aes(x = fpr, y = tpr, color = thresh)) +
  geom_path() +
  geom_point(data = optimal_thresh, color = "red") +
  theme_bw()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-46-1.png" width="576" style="display: block; margin: auto;" />


**Both ROC curves look very different, with the logistic curve looking cleaner and has a higher optimal threshold. Both analyses have pros and cons and different classifiers are more appropriate for answering different kinds of questions about the election. Regarding decision trees, as we saw in the output, they are intuitive and easy to implement visualize and interpret. They also have the ability to be pruned by cost complexity. However, they are sensitive to small changes in the data, have high variance, and could fail if they separation in the feature space is hard to approximate. I think working with this scale of data is difficult to implement decision trees with because there is a lot of variability in data. Regarding logistic regression, they are interpretable, flexible, and can take in a large number of predictors. However, like decision trees, they can fail if data are separated in the feature space.**

# Taking it further

21. This is an open question. Interpret and discuss any overall insights gained in this analysis and possible explanations. Use any tools at your disposal to make your case: visualize errors on the map, discuss what does or doesn't seem reasonable based on your understanding of these methods, propose possible directions (for example, collecting additional data or domain knowledge).  In addition, propose and tackle _at least_ one more interesting question. Creative and thoughtful analyses will be rewarded!

**As mentioned previously, both analyses have pros and cons and different classifiers are more appropriate for answering different kinds of questions about the election. In this example, it is shown that the logistic model performed better, providing a better misclassification error rate. I think the model would perform even better with theory-based predictors as that can influence the model, more predictors doesn't necessarily mean better.**

  * Instead of using the native attributes (the original features), we can use principal components to create new (and lower dimensional) sets of features with which to train a classification model. This sometimes improves classification performance.  Compare classifiers trained on the original features with those trained on PCA features.  
  
**I like the idea of using PC components to reevaluate the classification methods used. The minimum number of PCs needed to capture 90% of the variance for county data is *13*, however, using 13 components makes interpretation tricky, as evaluating and labeling all 13 will be extremely time-consulting. To assess differences in bias-variance trade off between using PC and raw data, I will rerun both the decision tree analysis and the logistic regress using the first 4 principal components to predict the election results and compare.**


Rerun PCs with combined `county_winner` column for easier wrangling:


```r
# Center and scale the data
county_winner_std <- election_county %>% 
  select(-candidate) %>% 
  scale(center = T, scale = T)


# Compute SVD
x_svd <- svd(county_winner_std)

# Get loadings
v_svd <- x_svd$v

# compute principal components
county_winner_pc <- county_winner_std %*% v_svd
colnames(county_winner_pc) <- paste('PC', 1:26, sep = '')

# scree and cumulative variance plots
pc_winner_vars <- x_svd$d^2/(nrow(county_winner_std) - 1)

tibble(PC = 1:min(dim(county_winner_std)),
       Proportion = pc_winner_vars/sum(pc_winner_vars),
       Cumulative = cumsum(Proportion)) %>%
  gather(key = 'measure', value = 'Variance Explained', 2:3) %>%
  ggplot(aes(x = PC, y = `Variance Explained`)) +
  geom_point() +
  geom_path() +
  facet_wrap(~ measure) +
  theme_bw() 
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-47-1.png" width="576" style="display: block; margin: auto;" />

```r
#tibble(PC = 1:min(dim(county_winner_std)),
#       Proportion = pc_winner_vars/sum(pc_winner_vars),
#       Cumulative = cumsum(Proportion)) %>% 
#  head(15) %>% 
#  pander()

v_svd[, 1:4] %>%
  as.data.frame() %>%
  mutate(variable = colnames(county_winner_std)) %>%
  gather(key = 'PC', value = 'Loading', 1:4) %>%
  arrange(variable) %>%
  ggplot(aes(x = variable, y = Loading)) +
  geom_point(aes(shape = PC)) +
  theme_bw() +
  geom_hline(yintercept = 0, color = 'blue') +
  geom_path(aes(linetype = PC, group = PC)) +
  theme(axis.text.x = element_text(angle = 90)) +
  labs(x = '') +
  facet_wrap(~PC)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-47-2.png" width="576" style="display: block; margin: auto;" />

```r
county_pc_4 <- county_winner_pc %>% 
  as_tibble() %>% 
  bind_cols(select(election_county, candidate)) %>% 
  select(PC1:PC4, candidate)
```

Note: Eigenvalues are slightly off because some observations were removed in the process of combining `county_winner` and `census_ct` but they are generally the same. 


*PC 1/Affluent*: High IncomePerCap, Employed, Income, and Professional. Low Child Poverty, Poverty, and Unemployment. 

*PC 2/Family Oriented*: High SelfEmployed, WorkAtHome, and FamilyWork. Low Women, PrivateWork, IncomeErr.


*PC 3/Working Class in Suburb*: High White, Drive, Production, and Private Work. Low Minority, and Transit.

*PC 4/Working Class in City*: High Carpool, Production, Employed. Low Citizen, Service, Office. 



```r
# hold out 20% of data as a test set
set.seed(12521)
county_part <- resample_partition(county_pc_4, c(test = 0.2, train = 0.8))
train <- as_tibble(county_part$train)
test <- as_tibble(county_part$test)
```


Decision Tree: 


```r
#Train a decision tree - training partition
nmin <- 5
tree_opts <- tree.control(nobs = nrow(train), 
                          minsize = nmin, 
                          mindev = exp(-6))
t_small <- tree(as.factor(candidate) ~ ., data = train,
                control = tree_opts, split = 'deviance') 
summary(t_small) 
```

```
## 
## Classification tree:
## tree(formula = as.factor(candidate) ~ ., data = train, control = tree_opts, 
##     split = "deviance")
## Number of terminal nodes:  56 
## Residual mean deviance:  0.3242 = 778.3 / 2401 
## Misclassification error rate: 0.0639 = 157 / 2457
```

Tree before pruning:


```r
draw.tree(t_small, cex = 0.8, size = 2)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-50-1.png" width="960" style="display: block; margin: auto;" />

Add cost-complexity pruning:


```r
# cost-complexity pruning
nfolds <- 10
cv_out <- cv.tree(t_small, K = nfolds, method = 'deviance')

# convert to tibble
cv_df <- tibble(alpha = cv_out$k,
                impurity = cv_out$dev,
                size = cv_out$size)

# choose optimal alpha
best_alpha <- slice_min(cv_df, impurity) %>%
  slice_min(size)

# select final tree
t_opt <- prune.tree(t_small, k = best_alpha$alpha)
summary(t_opt) 
```

```
## 
## Classification tree:
## snip.tree(tree = t_small, nodes = c(10L, 7L, 54L, 55L, 26L, 91L, 
## 23L, 25L, 90L, 9L))
## Number of terminal nodes:  13 
## Residual mean deviance:  0.4694 = 1147 / 2444 
## Misclassification error rate: 0.09565 = 235 / 2457
```

Tree after pruning:


```r
draw.tree(t_opt, cex = 0.8, size = 5)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-52-1.png" width="960" style="display: block; margin: auto;" />


Classification Errors on Test Partition - Before Pruning: 


```r
# predict on test data
preds_test_t0 <- predict(t_small, newdata = county_part$test, type = 'class')
# examin misclassification errors on test set - First Tree
classes_test <- as.data.frame(county_part$test) %>% pull(candidate)
test_errors_t0 <- table(class = classes_test, pred = preds_test_t0)
(test_errors_t0/rowSums(test_errors_t0)) 
```

```
##                  pred
## class             Donald Trump Hillary Clinton
##   Donald Trump      0.95348837      0.04651163
##   Hillary Clinton   0.49484536      0.50515464
```

Classification Errors on Test Partition - After Pruning: 


```r
# predict on test data
preds_test_t0 <- predict(t_opt, newdata = county_part$test, type = 'class')
# Pruned - Tree Error
classes_test <- as.data.frame(county_part$test) %>% pull(candidate)
test_errors_t0 <- table(class = classes_test, pred = preds_test_t0)
(test_errors_t0/rowSums(test_errors_t0)) 
```

```
##                  pred
## class             Donald Trump Hillary Clinton
##   Donald Trump      0.93410853      0.06589147
##   Hillary Clinton   0.51546392      0.48453608
```


**After training a decision tree before pruning, the resulting classification tree had 56 terminal nodes and a misclassification error rate of 0.0639. The four PCs were used. Already, we see that the misclassifciation error is much higher than using the full dataset shown earlier. After pruning the classification tree, now there are 13 terminal nodes and a misclassifcation error rate of 0.09565. Compared to the full dataset the pruned decision tree had less terminal nodes (5), but the misclassification error rate was slightly similar (0.08). The interpretation is slightly different now that we have to identify the PC on the tree. For example, if the loading on PC3 is less than -.610 (or in this case less affluent counties) and the loading on PC2 is less than -1.47912 (or in this case less family oriented counties), and the loading on PC3 is less than -3.40997 (even less affleunt counties) then it is likely they voted for Hillary Clinton. Already, this interpretation is extremely confusing compared to using all predictors.**


Logistic Regression: 
 

```r
fit_glm <- glm(as.factor(candidate) ~ ., family = 'binomial', data=train)

summary(fit_glm) 
```

```
## 
## Call:
## glm(formula = as.factor(candidate) ~ ., family = "binomial", 
##     data = train)
## 
## Deviance Residuals: 
##     Min       1Q   Median       3Q      Max  
## -3.1258  -0.4289  -0.2081  -0.1065   3.3147  
## 
## Coefficients:
##             Estimate Std. Error z value Pr(>|z|)    
## (Intercept) -2.76655    0.11026 -25.091  < 2e-16 ***
## PC1         -0.05276    0.02655  -1.987   0.0469 *  
## PC2         -0.64696    0.04784 -13.523  < 2e-16 ***
## PC3         -0.86938    0.05006 -17.368  < 2e-16 ***
## PC4         -0.39303    0.06011  -6.538 6.23e-11 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 2071.8  on 2456  degrees of freedom
## Residual deviance: 1262.3  on 2452  degrees of freedom
## AIC: 1272.3
## 
## Number of Fisher Scoring iterations: 6
```

**All four PCs are significant predictors of candidate. And it looks like all for PCs have similar interpretation: the probability of a county voting for Donald Trump decreases for each PC estimate. This was not the case when all variables were entered in the logistic model.**


Errors (Test Partition):


```r
# compute estimated probabilities
p_hat_glm <- predict(fit_glm, test, type = 'response')

# bayes classifier
y_hat_glm <- factor(p_hat_glm > 0.5, labels = c('No', 'Yes'))

# errors
error_glm <- table(test$candidate, y_hat_glm)
(error_glm / rowSums(error_glm))
```

```
##                  y_hat_glm
##                           No        Yes
##   Donald Trump    0.95155039 0.04844961
##   Hillary Clinton 0.52577320 0.47422680
```

**The true positve for Hillary Clinton in the model is a lot less than the model with all variables, however.**


ROC Curves:


Decision Tree: 


```r
#Decision Tree
t_predict <- predict(t_small, newdata = test) %>% 
  as_tibble()

tree_predict <- prediction(t_predict[,2], labels = test$candidate)

# compute error rates as a function of the probability threshold
perf_log <- performance(prediction.obj = tree_predict, 'tpr', 'fpr')

# extract rates and threshold from perf_lda as a tibble
rates_log <- tibble(fpr = perf_log@x.values,
                    tpr = perf_log@y.values,
                    thresh = perf_log@alpha.values) %>%
  unnest(everything())

# compute youden's stat
rates_log <- rates_log %>%
  mutate(youden = tpr - fpr)

# find the optimal value
optimal_thresh <- rates_log %>%
  slice_max(youden)

# ROC curve
rates_log %>%
  ggplot(aes(x = fpr, y = tpr, color = thresh)) +
  geom_path() +
  geom_point(data = optimal_thresh, color = "red") +
  theme_bw()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-57-1.png" width="576" style="display: block; margin: auto;" />


Logistic Regression:


```r
#Logistic Regression

# create prediction object
spam_predict <- prediction(predictions = p_hat_glm, labels = test$candidate)

# compute error rates as a function of the probability threshold
perf_log <- performance(prediction.obj = spam_predict, 'tpr', 'fpr')

# extract rates and threshold from perf_lda as a tibble
rates_log <- tibble(fpr = perf_log@x.values,
                    tpr = perf_log@y.values,
                    thresh = perf_log@alpha.values) %>%
  unnest(everything())

# compute youden's stat
rates_log <- rates_log %>%
  mutate(youden = tpr - fpr)

# find the optimal value
optimal_thresh <- rates_log %>%
  slice_max(youden)

# ROC curve
rates_log %>%
  ggplot(aes(x = fpr, y = tpr, color = thresh)) +
  geom_path() +
  geom_point(data = optimal_thresh, color = "red") +
  theme_bw()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-58-1.png" width="576" style="display: block; margin: auto;" />

**Overall, it seems like using the first four principal components did not improve neither the decision tree or logistic regression model. This could be due to many factors but the main one I can think of is that the first four PCs only capture 59% of the variance in the variables. The decision tree did not make sense to use even if it had a better misclassification rate as the interpretability of the tree did not as much sense as the model with all variables. While the logistic model did have all significant predictors, the error rate was much higher than the model with all variables, but less predictors were used. In the future, I would select variables based on theory, use PCA to reduce the dimensions and capture more variance than I did now, and then re-run the model.**
