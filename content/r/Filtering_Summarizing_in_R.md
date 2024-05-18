+++
title = 'Filtering, Selecting and Summarizing Data in R'
date = 2024-03-12T21:40:05-04:00
draft = false
comments = true
+++

Load our libraries

``` r
library(tidyverse)
```

    ## Warning: package 'tidyverse' was built under R version 4.1.3

    ## Warning: package 'ggplot2' was built under R version 4.1.3

    ## Warning: package 'tibble' was built under R version 4.1.3

    ## Warning: package 'tidyr' was built under R version 4.1.3

    ## Warning: package 'readr' was built under R version 4.1.3

    ## Warning: package 'purrr' was built under R version 4.1.3

    ## Warning: package 'dplyr' was built under R version 4.1.3

    ## Warning: package 'stringr' was built under R version 4.1.3

    ## Warning: package 'forcats' was built under R version 4.1.3

    ## Warning: package 'lubridate' was built under R version 4.1.3

    ## -- Attaching core tidyverse packages ------------------------ tidyverse 2.0.0 --
    ## v dplyr     1.1.2     v readr     2.1.4
    ## v forcats   1.0.0     v stringr   1.5.0
    ## v ggplot2   3.4.2     v tibble    3.2.1
    ## v lubridate 1.9.2     v tidyr     1.3.0
    ## v purrr     1.0.1     
    ## -- Conflicts ------------------------------------------ tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()
    ## i Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(knitr)
library(rmarkdown)
```

Import your data

Here we’re going to read in the clinical metadata from Metabric available from cBioPortal (don’t forget to set your working directory, then access the data in relation to your working directory)  
This file is also available from the Files section on the blog

``` r
metabric <- read.delim("./data_clinical_patient.txt", header = FALSE)

# Just look at the first 10 rows and first 5 columns
metabric[1:10, 1:5]
```

    ##                                            V1                            V2
    ## 1                         #Patient Identifier Lymph nodes examined positive
    ## 2  #Identifier to uniquely specify a patient. Number of lymphnodes positive
    ## 3                                     #STRING                        STRING
    ## 4                                          #1                             1
    ## 5                                  PATIENT_ID LYMPH_NODES_EXAMINED_POSITIVE
    ## 6                                     MB-0000                            10
    ## 7                                     MB-0002                             0
    ## 8                                     MB-0005                             1
    ## 9                                     MB-0006                             3
    ## 10                                    MB-0008                             8
    ##                             V3            V4            V5
    ## 1  Nottingham prognostic index   Cellularity  Chemotherapy
    ## 2  Nottingham prognostic index Tumor Content Chemotherapy.
    ## 3                       NUMBER        STRING        STRING
    ## 4                            1             1             1
    ## 5                          NPI   CELLULARITY  CHEMOTHERAPY
    ## 6                        6.044                          NO
    ## 7                         4.02          High            NO
    ## 8                         4.03          High           YES
    ## 9                         4.05      Moderate           YES
    ## 10                        6.08          High           YES

Here we can see the dataframe includes several rows that give
explanations of the column, but we don’t want to include those in our
analysis

Let’s keep those to reference what the various columns means

``` r
#dataframe[1:5, ] tells R to keep rows 1-5 and all columns
reference <- metabric[1:5, ]
```

Now I’ll re-read in the data, skipping the first 4 rows and making the
5th row the column names with header = TRUE

``` r
metabric <- read.delim("./data_clinical_patient.txt", skip = 4, header = TRUE)
head(metabric) %>% kable() 
```

| PATIENT_ID | LYMPH_NODES_EXAMINED_POSITIVE |   NPI | CELLULARITY | CHEMOTHERAPY | COHORT | ER_IHC  | HER2_SNP6 | HORMONE_THERAPY | INFERRED_MENOPAUSAL_STATE | SEX    | INTCLUST | AGE_AT_DIAGNOSIS | OS_MONTHS | OS_STATUS  | CLAUDIN_SUBTYPE | THREEGENE             | VITAL_STATUS    | LATERALITY | RADIO_THERAPY | HISTOLOGICAL_SUBTYPE | BREAST_SURGERY    | RFS_MONTHS | RFS_STATUS     |
|:-----------|------------------------------:|------:|:------------|:-------------|-------:|:--------|:----------|:----------------|:--------------------------|:-------|:---------|-----------------:|----------:|:-----------|:----------------|:----------------------|:----------------|:-----------|:--------------|:---------------------|:------------------|-----------:|:---------------|
| MB-0000    |                            10 | 6.044 |             | NO           |      1 | Positve | NEUTRAL   | YES             | Post                      | Female | 4ER+     |            75.65 | 140.50000 | 0:LIVING   | claudin-low     | ER-/HER2-             | Living          | Right      | YES           | Ductal/NST           | MASTECTOMY        | 140.500000 | 0:Not Recurred |
| MB-0002    |                             0 | 4.020 | High        | NO           |      1 | Positve | NEUTRAL   | YES             | Pre                       | Female | 4ER+     |            43.19 |  84.63333 | 0:LIVING   | LumA            | ER+/HER2- High Prolif | Living          | Right      | YES           | Ductal/NST           | BREAST CONSERVING |  84.633333 | 0:Not Recurred |
| MB-0005    |                             1 | 4.030 | High        | YES          |      1 | Positve | NEUTRAL   | YES             | Pre                       | Female | 3        |            48.87 | 163.70000 | 1:DECEASED | LumB            |                       | Died of Disease | Right      | NO            | Ductal/NST           | MASTECTOMY        | 153.300000 | 1:Recurred     |
| MB-0006    |                             3 | 4.050 | Moderate    | YES          |      1 | Positve | NEUTRAL   | YES             | Pre                       | Female | 9        |            47.68 | 164.93333 | 0:LIVING   | LumB            |                       | Living          | Right      | YES           | Mixed                | MASTECTOMY        | 164.933333 | 0:Not Recurred |
| MB-0008    |                             8 | 6.080 | High        | YES          |      1 | Positve | NEUTRAL   | YES             | Post                      | Female | 9        |            76.97 |  41.36667 | 1:DECEASED | LumB            | ER+/HER2- High Prolif | Died of Disease | Right      | YES           | Mixed                | MASTECTOMY        |  18.800000 | 1:Recurred     |
| MB-0010    |                             0 | 4.062 | Moderate    | NO           |      1 | Positve | NEUTRAL   | YES             | Post                      | Female | 7        |            78.77 |   7.80000 | 1:DECEASED | LumB            | ER+/HER2- High Prolif | Died of Disease | Left       | YES           | Ductal/NST           | MASTECTOMY        |   2.933333 | 1:Recurred     |

``` r
#note: %>% kable()  is primarily used here for visualization of our dataframe in a markdown file, when working in your own RStudio session you wouldn't need to have that part
# if you're trying this on your own in an Rscript (.R file), just delete '%>% kable()' portion of the code
```

### A few types of manipulation that we can do:

1.  Filter the data (only keep the rows or columns you need)

2.  Summarize the data

## 1. Filtering

First, let’s start with filtering the data side note: the %>% operator
in dplyr allows you pass objects to functions so you’ll see that
throughout the following code

``` r
# Only keep rows where the CLAUDIN_SUBTYPE is LumA
## we can use the filter function in dplyr
## use the '==' in the filter function to tell R what condition you want to filter for
## filter(column_name == condition)

lumA <- metabric %>% filter(CLAUDIN_SUBTYPE == "LumA")
nrow(lumA)
```

    ## [1] 700

``` r
# 700 patients are the LumA subtype

# You can do a quick QC check here
unique(lumA$CLAUDIN_SUBTYPE) # ask R what are the unique values in the Claudin subtype column, after our filtering they should only be LumA subtype
```

    ## [1] "LumA"

``` r
# OR we can filter by numeric values (> or < a certain value)
age_df <- metabric %>% filter(AGE_AT_DIAGNOSIS > 50)
nrow(age_df)
```

    ## [1] 1926

``` r
# 1926 patients were diagnosed when over 50 years old


# combine filters
lumA_age <- metabric %>% filter(CLAUDIN_SUBTYPE == "LumA" & AGE_AT_DIAGNOSIS > 50)
nrow(lumA_age)
```

    ## [1] 574

``` r
# 574 patients are lumA subtype and were diagnosed >50 yrs
```

Now you have separate dataframes for patients where those conditions are
true and you can perform an analysis on those subset of patients

Filtering can also be done using base R

``` r
# bracket notation: dataframe[rows, columns]
# dataframe[condition, ] -- this tell R keep all rows where that condition is met and keep all columns
lumA <- metabric[metabric$CLAUDIN_SUBTYPE == "LumA", ]
nrow(lumA)
```

    ## [1] 700

### Subset the data

Only keep the columns you need You can do this using the select function
in dplyr  
For example, I may only need to keep the patient ID, Claudin subtype and
age at diagnosis columns  
I can create another smaller dataframe with only those specific columns

``` r
# dataframe %>% select(list columns to keep)

metabric_small <- metabric %>% select(PATIENT_ID, CLAUDIN_SUBTYPE, AGE_AT_DIAGNOSIS)
metabric_small %>% head()
```

    ##   PATIENT_ID CLAUDIN_SUBTYPE AGE_AT_DIAGNOSIS
    ## 1    MB-0000     claudin-low            75.65
    ## 2    MB-0002            LumA            43.19
    ## 3    MB-0005            LumB            48.87
    ## 4    MB-0006            LumB            47.68
    ## 5    MB-0008            LumB            76.97
    ## 6    MB-0010            LumB            78.77

Or you can tell the select function which column(s) you DON’T want by
using “-c(columns_to_remove))”

``` r
metabric_small <- metabric_small %>% select(-c(AGE_AT_DIAGNOSIS))
metabric_small %>% head()
```

    ##   PATIENT_ID CLAUDIN_SUBTYPE
    ## 1    MB-0000     claudin-low
    ## 2    MB-0002            LumA
    ## 3    MB-0005            LumB
    ## 4    MB-0006            LumB
    ## 5    MB-0008            LumB
    ## 6    MB-0010            LumB

Or in base R

``` r
# dataframe[, c(columns to keep)]
metabric_small <- metabric[, c("PATIENT_ID", "CLAUDIN_SUBTYPE", "AGE_AT_DIAGNOSIS")]

# I find this less intuitive than the dplyr select, but you can tell R that you don't want specific column names 
# ! = not, so here I'm saying keep all columns except for columns that are named AGE_AT_DIAGNOSIS
metabric_small <- metabric_small[, (!names(metabric_small) %in% "AGE_AT_DIAGNOSIS")]
```

## 2. Summarize the data

Now we can summarize the data, for example, how many patients are each
subtype? Here we can use the summarize function from dplyr

``` r
metabric %>%
  group_by(CLAUDIN_SUBTYPE) %>%  # what columns you want to group the patients by
  summarize(count = n()) # count will be our new column name, n() = count the number of rows per group as specified in group_by
```

    ## # A tibble: 8 x 2
    ##   CLAUDIN_SUBTYPE count
    ##   <chr>           <int>
    ## 1 ""                529
    ## 2 "Basal"           209
    ## 3 "Her2"            224
    ## 4 "LumA"            700
    ## 5 "LumB"            475
    ## 6 "NC"                6
    ## 7 "Normal"          148
    ## 8 "claudin-low"     218

``` r
# We can also add in 1 line to sort the output from highest to lowest
metabric %>%
  group_by(CLAUDIN_SUBTYPE) %>% 
  summarize(count = n()) %>%
  arrange(-count)
```

    ## # A tibble: 8 x 2
    ##   CLAUDIN_SUBTYPE count
    ##   <chr>           <int>
    ## 1 "LumA"            700
    ## 2 ""                529
    ## 3 "LumB"            475
    ## 4 "Her2"            224
    ## 5 "claudin-low"     218
    ## 6 "Basal"           209
    ## 7 "Normal"          148
    ## 8 "NC"                6

Now we can see most patients are LumA, followed by 529 patients with a
missing subtype, then LumB, Her2, Claudin-low and so on  
It may be important to note here that is says 148 patients are “Normal”, and we
may need to investigate that further to see if those patients should be
removed from analyses of tumor samples  

We can also use summarize to get useful values such as the mean, median,
standard deviation, min or max by group
<https://dplyr.tidyverse.org/reference/summarise.html>

``` r
metabric %>%
  summarize(mean_lymph_nodes = mean(LYMPH_NODES_EXAMINED_POSITIVE, na.rm = TRUE)) # include na.rm to remove rows with NAs from our calculation of the mean
```

    ##   mean_lymph_nodes
    ## 1         1.950513

This becomes more helpful if we want to calculate summaries by groups

``` r
# Here we can get the mean + lymph nodes per Hormone receptor group
metabric %>%
  group_by(THREEGENE) %>%
  summarize(mean_lymph_nodes = mean(LYMPH_NODES_EXAMINED_POSITIVE, na.rm = TRUE)) %>%
  arrange(-mean_lymph_nodes)
```

    ## # A tibble: 5 x 2
    ##   THREEGENE               mean_lymph_nodes
    ##   <chr>                              <dbl>
    ## 1 "HER2+"                             3.08
    ## 2 "ER-/HER2-"                         2.24
    ## 3 "ER+/HER2- High Prolif"             2.11
    ## 4 ""                                  1.81
    ## 5 "ER+/HER2- Low Prolif"              1.44

Here we can see Her2+ positive tumors have the highest mean positive
lymph nodes

We can group by multiple columns as well, so for example, do we see a
higher mean of positive lymph nodes in Her2+ patients with high
cellularity?

``` r
metabric %>%
  group_by(THREEGENE, CELLULARITY) %>%
  summarize(mean_lymph_nodes = mean(LYMPH_NODES_EXAMINED_POSITIVE, na.rm = TRUE)) %>%
  arrange(-mean_lymph_nodes)
```

    ## `summarise()` has grouped output by 'THREEGENE'. You can override using the
    ## `.groups` argument.

    ## # A tibble: 20 x 3
    ## # Groups:   THREEGENE [5]
    ##    THREEGENE               CELLULARITY mean_lymph_nodes
    ##    <chr>                   <chr>                  <dbl>
    ##  1 "HER2+"                 "High"                 3.53 
    ##  2 "HER2+"                 ""                     2.86 
    ##  3 "ER-/HER2-"             "Moderate"             2.64 
    ##  4 "HER2+"                 "Moderate"             2.63 
    ##  5 "ER+/HER2- High Prolif" "Moderate"             2.35 
    ##  6 ""                      "High"                 2.24 
    ##  7 ""                      "Moderate"             2.19 
    ##  8 "HER2+"                 "Low"                  2.16 
    ##  9 "ER-/HER2-"             "High"                 2.15 
    ## 10 "ER+/HER2- High Prolif" "High"                 2.07 
    ## 11 "ER-/HER2-"             "Low"                  2.02 
    ## 12 ""                      ""                     1.65 
    ## 13 "ER+/HER2- Low Prolif"  "Low"                  1.62 
    ## 14 "ER-/HER2-"             ""                     1.57 
    ## 15 "ER+/HER2- High Prolif" "Low"                  1.47 
    ## 16 "ER+/HER2- Low Prolif"  "Moderate"             1.45 
    ## 17 "ER+/HER2- Low Prolif"  "High"                 1.42 
    ## 18 "ER+/HER2- High Prolif" ""                     0.727
    ## 19 ""                      "Low"                  0.647
    ## 20 "ER+/HER2- Low Prolif"  ""                     0.529

Combine summarize and filter to just look at the HER2+ patients

``` r
metabric %>%
  group_by(THREEGENE, CELLULARITY) %>%
  summarize(mean_lymph_nodes = mean(LYMPH_NODES_EXAMINED_POSITIVE, na.rm = TRUE)) %>%
  arrange(-mean_lymph_nodes) %>%
  filter(THREEGENE == "HER2+")
```

    ## `summarise()` has grouped output by 'THREEGENE'. You can override using the
    ## `.groups` argument.

    ## # A tibble: 4 x 3
    ## # Groups:   THREEGENE [1]
    ##   THREEGENE CELLULARITY mean_lymph_nodes
    ##   <chr>     <chr>                  <dbl>
    ## 1 HER2+     "High"                  3.53
    ## 2 HER2+     ""                      2.86
    ## 3 HER2+     "Moderate"              2.63
    ## 4 HER2+     "Low"                   2.16

Now you can filter, subset and summarize your data  
Hopefully this allows you to subset your data into workable formats and allows you to see some new meaningful patterns in your data
