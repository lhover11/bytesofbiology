+++
title = 'Adjusted p-values'
date = 2024-02-19T21:48:15-05:00
draft = false
+++


Why do we need to adjust p-values? When running analyses on 1000s of
genes, we can get many false positives, or genes that look significantly
different just by chance

```r 
# Load libraries
library(tidyverse)
```

If you want to generate adjusted p-values for a set of p-values in R, I
use: p.adjust(p, method = p.adjust.methods) from the stats package

Let’s break this function down and see how it works
```r 
# Let's start by creating some fake example p-values
set.seed(999)

## Here we generate a random vector of 1000 values between 1e-10 and 1
pvalues <- runif(n=50, min=1e-10, max=1)
head(pvalues) # Look at first 6 values

## [1] 0.38907138 0.58306072 0.09466569 0.85263123 0.78674676 0.11934226

## Look at distribution of p-values
hist(pvalues)

# Next we compute the adjusted p-values The available methods:

p.adjust.methods

## [1] "holm"       "hochberg"   "hommel"     "bonferroni" "BH"         "BY"         "fdr"       
## [8] "none"
```

The Benjamini & Hochberg (1995), method = “BH” or “fdr”, controls the
false discovery rate or in other words controls the number of false
positives

See note below from Gordon Smyth regarding the BH method in the stats
package: “In 2002, I re-interpreted the Benjamini and Hochberg (BH) and
Benjamini and Yekutieli (BY) procedures in terms of adjusted p-values. I
implemented the resulting algorithms in the function p.adjust() in the
stats package” <https://support.bioconductor.org/p/49864/>

```r
# First let’s generate the adjusted p-values using p.adjust

# Put p-values into dataframe
df <- data.frame(pvalues = pvalues)

# Calculate adjusted p-value and add those to our dataframe
df$padjust <- p.adjust(df$pvalues, method = "BH")
```

Now we’ll calculate it by hand and compare the results

The 3 things we need to calculate the BH adjusted p-values are:
- the nominal p-value
- the p-value ranking (smallest to largest)
- the total number of tests

```r
# First we need to rank the p-values from smallest to largest

df <- df %>% arrange(pvalues)
df$rank <- 1:nrow(df)

# Next, multiply each p-value by the total number of tests and divide that value by its rank
df$BH <- (df$pvalues * nrow(df))/df$rank
```

Second, make sure that the resulting sequence is non-decreasing: if it ever starts decreasing, make the preceding p-value equal to the subsequent (repeatedly, until the whole sequence becomes non-decreasing).

Next, start at the largest p-value and we make sure the list of adjusted p-values is consistently decreasing So in our example we start at rank 50 and all the values are decreasing until we hit row 45 0.903 < 0.917

```r
df[45:50, ]

##      pvalues   padjust rank        BH
## 45 0.8260703 0.9031309   45 0.9178559
## 46 0.8308805 0.9031309   46 0.9031309
## 47 0.8526312 0.9052239   47 0.9070545
## 48 0.8690149 0.9052239   48 0.9052239
## 49 0.9074913 0.9260115   49 0.9260115
## 50 0.9875201 0.9875201   50 0.9875201
```

So then we make the rank 45 adjusted p-value equal to the rank 46 p-value since that one was smaller. And we continue that until we hit the next p-value smaller than
0.903 which happens at rank 36

```r
df[35:45, ]

##      pvalues   padjust rank        BH
## 35 0.6196068 0.8802792   35 0.8851526
## 36 0.6338010 0.8802792   36 0.8802792
## 37 0.7399023 0.9031309   37 0.9998680
## 38 0.7765914 0.9031309   38 1.0218308
## 39 0.7867468 0.9031309   39 1.0086497
## 40 0.7872129 0.9031309   40 0.9840162
## 41 0.8051165 0.9031309   41 0.9818493
## 42 0.8162975 0.9031309   42 0.9717827
## 43 0.8195141 0.9031309   43 0.9529233
## 44 0.8212559 0.9031309   44 0.9332454
## 45 0.8260703 0.9031309   45 0.9178559
```

And we continue that process all the way through rank 1

We can assign these p-values in R to make sure our calculated adjusted
p-value matches those output from p.adjust()

```r
# First assign our new column equal to the BH adjusted p-value
df$BH_final <- df$BH

# For each row, starting at the highest rank, if the larger ranked p-value is smaller than the subsequent ranked p-value, make the subsequent ranked p-value equal to the larger ranked p-value
for(i in nrow(df):2){
    if (df$BH_final[i] < df$BH_final[i-1]){
    df$BH_final[i - 1] <- df$BH_final[i]
    }
}
```
Everything matches! Now we have a little better understanding of what’s
going on when we using the BH method in p.adjust. You can try the other
methods in p.adjust as well

side note: Choosing a false discovery rate is about a trade-off. Things
to consider, is it easy to follow-up on any “significant” results to
confirm any findings, is this hypothesis generating? In that case then
maybe setting a higher FDR is better
