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

Second, make sure that the smaller p-values always have smaller FDR values (rank 1 pvalue < rank 2 pvalue < rank 3 pvalue ...)  
If the adjusted pvalues start decreasing, make the preceding adj. p-value equal to the subsequent adj. pvalue (repeatedly, until the whole sequence becomes non-decreasing).

To do this, we start at the largest p-value and we make sure the list of adjusted p-values is consistently decreasing.  
So in our example we start at rank 50 and all the values are decreasing until we hit row 47: 0.907 > 0.905

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

So then we make the rank 47 adjusted p-value equal to the rank 48 adjusted p-value since the rank 48 adj. p-value one was smaller.  
Next, the rank 46 adj pvalue is smaller than 0.905, so we don't need to do anything there  
But the rank 45 adj pvalue is larger than 0.903, so we'll set that adjusted pvalue to 0.903

And we continue that process all the way through rank 1

We can also write a quick loop in R to adjust our adjusted p-values as described above  
We'll also make sure our adjusted p-values match those output from p.adjust() to make sure we understand what's going on 

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
