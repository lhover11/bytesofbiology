+++
title = '02 Plot Deconvolution Scores'
date = 2024-05-04T16:59:42-04:00
draft = false
comments = true
weight = 2
+++



As described on the main Immune Deconvolution page, I want to compare my results to previously published deconvolution results run on breast cancer samples

Figure 1 of the Wang et al publication shows the cell fraction estimates of Metabric samples using CIBERSORT  
![Plot1](../images/Wang_2023_Fig1.png)

From the publication:
"Figure 1 shows the relative abundances of each immune cell subtypes in METABRIC. Over half of the cell subtypes had very low abundances. Cell subtypes with relatively high abundances were M0 macrophages (14.8%), M2 macrophages (11.5%), plasma cells (9.3%), M1 macrophages (8.2%), resting mast cells (8.0%), follicular helper T cells (6.2%), CD8 positive T cells (5.5%), gamma delta T cells (4.8%), activated NK cells (3.9%), and memory B cells (3.2%)."


Let's see if I get similar results when using QuantiSeq

```r
# Load libraries
library(tidyverse)
library(RColorBrewer)

# Set parameters

## set your working directory to the correct directory
#setwd("your-project-directory")

# Read in data (file created in step 01 of this project)
scores <- read.csv("../../data-final/metabric_deconvolution_scores.csv")
head(scores)

# Transform data for plotting
## make dataframe into a long skinny format

df <- scores %>% pivot_longer(-SampleID, names_to = "signature", values_to = "score")

# split signature column into method and cell type columns
df$method <- ifelse(grepl("Quantiseq", df$signature), "Quantiseq", "Timer")
df$cell_type <- gsub(".*_", "", df$signature)


# For now subset to quantiseq
quant <- df %>% filter(method == "Quantiseq")

ggplot(df %>% filter(grepl("Quantiseq", signature)),
       aes(x = cell_type, y = score)) +
  geom_boxplot()

# As expected the highest frequency is for "Other" cells, which would represent the tumor cells
# this is a good qc check

# remove "Other" cells
quant <- quant %>% filter(cell_type != "Other")

# Get cell type into a similar order as the publication
order <- c("B.cells", "T.cells.CD8", "T.cells.CD4", "Tregs",
           "NK.cells", "Monocytes", "Macrophages.M1", "Macrophages.M2",
           "Dendritic.cells", "Neutrophils")
quant$cell_type <- factor(quant$cell_type, levels = c(order))


# Use same colors as used in the publication by cell type
colors <- c("#FE7F7A", "#C69615", "#9AB123", "#35BC44",
            "#56D0B9", "#5BC2DF", "#63A6D6", "#919BDB",
            "#CA84D4", "#FA71A0")

# get median fraction per cell type
med <- quant %>%
  group_by(cell_type) %>%
  summarize(median = signif(median(score, na.rm = TRUE)*100, 2))

# Put medians in the correct order
med <- med[match(order, med$cell_type), ]

# Make boxplot of cell type fractions
ggplot(quant,
       aes(x = cell_type, y = (score * 100), color = cell_type)) +
  geom_boxplot(outlier.shape = NA, width = 0.3) +
  geom_point(size = 1.5) +
  # format plot
  theme_bw() +
  theme(legend.position="none", # remove legend
        axis.title = element_text(size =16), # make text larger
        axis.text.x = element_text(angle = 45, hjust=1, size = 12), # rotate x-axis text
        axis.text.y = element_text(size = 12)
        ) + 
  xlab("Cell Type") +
  ylab("Cell Percentage") +
  # add median % per cell type
  annotate('text', x = c(1:length(unique(quant$cell_type))), y = 55,
           label =  paste0(med$median, "%"),
           size = 4
           )
  
ggsave("../../results/02-QuantiSeq_Fractions_Boxplot.png", w = 8, h = 4)
```
![Plot2](../images/02-QuantiSeq_Fractions_Boxplot.png)



Let's compare my plot and the CIBERSORT plot side by side
![Plot2](../images/02-Quantiseq_CIBERSORT_comparison.png)

We can see some similarities and differences
We should keep in mind that the methods used are different and estimate different populations  
For example, CIBERSORT estimates 22 cell types whereas quanTIseq outputs only 10

Also CIBERSORT results are fractions relative to the immune-cell content (unless CIBERSORT absolute was used in the publication), whereas quanTIseq scores are relative to the total cells generating an absolute score so clearly we're not comparing apples to apples 

But given all those differences, we still see some similarities:
In both CIBERSORT and quanTIseq we see a higher fraction of M2 macrophages compared to M1 macrophages  
We also see a low fraction of monoctyes in both results, with the exception of a few samples with a high fraction of monocytes.  It would be interesting to see if the sample with the highest monocyte fraction is the same in both plots.  
It's a bit hard to compare patterns in the T cell and B cell populations as CIBERSORT breaks those down into more granular subsets.  
The neutrophil and dendritic cell estimates are higher in quanTIseq comapred to very low estimates for almost all samples in CIBERSORT  

Maybe it's of note that even though we're comparing different methods, different cell types and different scores we're not seeing wildly different results which seems promising.    
Instead of trying to compare exact estimates across methods and different cell types, we should:  
a. look at correlations -- do tumors with higher estimates in 1 method show higher estimates in a 2nd method AND  
b. perform and compare hierarchical clustering to see if we get similar groupings to the publication  
c. check, if we see an association between cell type estimtes and clinical data, do we see that across deconvolution methods?  That would be an important QC check