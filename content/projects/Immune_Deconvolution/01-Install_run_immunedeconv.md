+++
title = '01 Install_run_immunedeconv'
date = 2024-05-04T16:15:01-04:00
draft = false
comments = true
+++

# Immune Deconvolution

What is deconvolution? 
"Deconvolution methods define the problem as mathematical equations that model the gene expression of a tissue sample as the weighted sum of the expression profiles from the cells in the population mix" (TIMER2.0 for analysis of tumor-infiltrating immune cells, [Li et al, 2020](https://doi.org/10.1093/nar/gkaa407))

Today, I'll run deconvolution using the R package: immunedeconv
https://omnideconv.org/immunedeconv/index.html

You can find instructions for the installation and more information about the package at the link above

Briefly, this package allows you to run multiple different deconvolution methods (TIMER, xCell, MCP-counter, CIBERSORT, EPIC & quanTIseq) to estimate immune cell fractions using bulk RNA-seq data
In practice, this may look like trying to estimate the immune cell infiltrates in a tumor sample

The link above will also guide you through examples of how to run the various methods


I found a several papers looking at immune deconvolution results in breast cancer samples including TCGA and Metabric samples, so I'm going to use Metabric sample data and compare my results to published results
This data can be found at cbioportal (https://www.cbioportal.org/study/summary?id=brca_metabric)


## Step 1 : Install the package Immunedeconv

I installed this in a separate environment as to not disrupt my other R projects  
# You can do that in RStudio by going to: 
File --> New Project --> Choose if you want this to be in a new directory or an existing directory --> New Project --> Create your new directory name, where you want this directory and check the box "Use renv with this project" to create a new environment associated with this project  
I did that so I wouldn't disrupt the package versions and set-up I have for my other work in R  

Next, following the instruction on the website:
```r
install.packages("remotes")
remotes::install_github("omnideconv/immunedeconv")

# if the installation fails, make sure you have Rtools installed.  This is how I installed it:
# install Rtools from CRAN
# https://cran.r-project.org/bin/windows/Rtools/rtools40.html

# once it's installed, add Rtools to path by running the line below in R:
write('PATH="${RTOOLS40_HOME}\\usr\\bin;${PATH}"', file = "~/.Renviron", append = TRUE)
```
## Step 2: Run Deconvolution

```r
# Load libraries
library(tidyverse)
library(immunedeconv)

# Load data
metabric <- read.delim("../../data-raw/brca_metabric/data_mrna_illumina_microarray.txt")

# Check out our dataframe
dim(metabric) # 20603 rows(genes), 1982 columns(samples)

# check the format of the columns and make sure it's as expected
str(metabric)

# Look at the top of the dataframe
metabric[1:5, 1:5]

# Look at the end to see if there's anything weird at the bottom (sometimes there will be additional text that's not part of the actual data)
metabric[nrow(metabric), 1:5]

# The format we need for running the deconvolution methods is a gene x sample matrix with HGNC gene symbols as the rownames
## Format metabric df
metabric2 <- metabric
row.names(metabric2) <- metabric2$Hugo_Symbol
# When we try to do this, we get a warning message that there are duplicate gene symbol so we can't make those row names
# let's look at some of the duplicates
check <- metabric %>% filter(Hugo_Symbol == "BIRC5")

# How many are duplicated
length(metabric$Hugo_Symbol[duplicated(metabric$Hugo_Symbol)]) # 217

## For duplicated genes, we will take the average of the values
dup_genes <- metabric$Hugo_Symbol[duplicated(metabric$Hugo_Symbol)]

dups <- metabric %>% filter(Hugo_Symbol %in% dup_genes)

### Get average values for all metabric sample columns (all those start with MB)
average_values <- dups %>%
  group_by(Hugo_Symbol) %>%
  summarize(across(starts_with("MB"), mean))

# QC check:
dups[dups$Hugo_Symbol == "BIRC5", 1:4]
average_values[average_values$Hugo_Symbol == "BIRC5", 1:4]

## averages check out, keep going!

metabric2 <- metabric
metabric2 <- metabric2 %>% filter(!Hugo_Symbol %in% dup_genes)
metabric2 <- metabric2 %>% select(-c(Entrez_Gene_Id))
metabric2 <- rbind(metabric2, average_values)

row.names(metabric2) <- metabric2$Hugo_Symbol
metabric2$Hugo_Symbol <- NULL

# If we look at the file: meta_mrna_illumina_microarray, we can see that the expression
# values are log2 transformed
# profile_description: Expression log2 intensity levels (Illumina HT-12 v3 microarray)

# For the deconvolution methods, we need to use linear data, so we need to convert the data back to linear format

metabric2 <- apply(metabric2, 2, function(x) (2^(x)))

# QC check - make sure data was transformed correctly
metabric[1:4, 1:4]
metabric2[1:4, 1:4]

dim(metabric2)


# perform deconvolution
# Use TIMER method, for TIMER we need to indicate the tumor type (here it'll be BRCA)
# indications needs to be a vector that specifies an indication for each sample 
tumor_type <- rep("BRCA", times = ncol(metabric2))
timer <- deconvolute(metabric2, "timer",
            indications=tumor_type)

timer
# We can see the output of TIMER is another dataframe with the cell type as the rows and each tumor is a column
# For TIMER scores, only between-sample comparisons are allowed -- such as in sample 1 there are more CD8+ T cells than in sample 2, but with TIMER scores we can't say there are more CD8 T cells than CD4 T cells in sample 1


# Transform our outputs so the columns are the cell types and the rows are the samples
cell_types <- timer$cell_type
timer$cell_type <- NULL
timer_res <- as.data.frame(t(timer))
names(timer_res) <- cell_types

# Add method to column names
names(timer_res) <- paste0("TIMER", "_", names(timer_res))

# Make sample ID a column
timer_res$SampleID <- row.names(timer_res)


#--------------------

# Run quantiseq
## quantiseq won't work if there are any NA values in the matrix, let's check for those
any(is.na(metabric2))
columns_with_na <- colnames(metabric2)[which(colSums(is.na(metabric2)) > 0)] # 16 samples
rows_with_na <- which(rowSums(is.na(metabric2)) > 0) # 11 genes

# For now, we can remove the samples with missing values since it's only 16 samples
metabric3 <- metabric2[, !(colnames(metabric2) %in% columns_with_na)]

# check we only removed 16 samples
dim(metabric2)
dim(metabric3)


quantiseq <- deconvolute_quantiseq(metabric3, tumor = TRUE, arrays = TRUE, scale_mrna = FALSE)
# tumor = TRUE if using tumor samples
# arrays = TRUE if working with Microarray data
# scale_mrna = FALSE to disable correction for cell type-specific differences in mRNA content

# quantiseq output represents absolute scores - scores that can be interpreted as a cell fraction
# allows for comparisons between samples AND between cell-types
# examples: There are more T-cells than B-cells in sample 1
# and sample 1 has more B cells than sample 2

# Transform our outputs so the columns are the cell types and the rows are the samples
quant_res <- as.data.frame(t(quantiseq))
names(quant_res) <- paste0("Quantiseq", "_", names(quant_res))
quant_res$SampleID <- row.names(quant_res)


#--------------------

# Join TIMER and Quantiseq results together
res <- full_join(timer_res, quant_res, by = c("SampleID"))

# Rearrange columns
res <- res %>% select(SampleID, everything())

# write out results
write.csv(res, "../../data-final/metabric_deconvolution_scores.csv", row.names = FALSE)
```

At the end, you should have a file that looks like this:
![Plot1](../images/01-deconv-output.png)