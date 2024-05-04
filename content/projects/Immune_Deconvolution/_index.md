+++
title = 'Immune_Deconvolution'
date = 2024-05-04T16:14:17-04:00
draft = true
+++


I recently came across this publication: *Machine learning-based cluster analysis of immune cell subtypes and breast cancer survival*
Wang, Z., Katsaros, D., Wang, J. et al. Machine learning-based cluster analysis of immune cell subtypes and breast cancer survival. Sci Rep 13, 18962 (2023). https://doi.org/10.1038/s41598-023-45932-4

in which the authors show that by using immune cell infiltration estimates they could identify distinct tumor groupings which were associated with differences in survival
The authors: 
- perform immune deconvolution on breast cancer samples from TCGA and Metabric
- use the deconvolution scores to perform hierarchical clustering splitting the samples into 2 main clusters
- look at the association of those clusters with survival
- explore differentially expressed genes between clusters


As Metabric and TCGA data is publicly available, I wanted to perform similar analyses and compare my results to this publication

