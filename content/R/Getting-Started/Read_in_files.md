+++
title = 'Reading in Files'
date = 2024-02-20T10:51:58-05:00
draft = false
+++

Here are 3 simple ways you can use to read in various data formats that
you may commonly come across in bioinformatics analyses

Note: all the files used in this code are linked in the Files page

First we need to set our working directory so R knows where to locate
the files

    setwd("~/Bioinformatics/practice") # change this to whatever directory you saved your files in

Let’s start with reading in csv and txt files csv = comma-separated
values, this is a text file that separates fields using commas or
sometimes semicolons

    # data downloaded from https://depmap.org/portal/gene/KRAS?tab=dependency

    file <- read.csv("KRAS CRISPR_DepMap Public 23Q4+Score Chronos.csv")
    head(file)

    ##    Depmap.ID CRISPR..DepMap.Public.23Q4.Score..Chronos. Cell.Line.Name
    ## 1 ACH-001300                                 -0.5756392         CHLA15
    ## 2 ACH-001322                                 -0.5538883           CME1
    ## 3 ACH-001318                                 -0.3898990        PLCPRF5
    ## 4 ACH-001310                                 -0.6193042           HA1E
    ## 5 ACH-001307                                 -0.3254792          8505C
    ## 6 ACH-001306                                 -0.4410579          8305C
    ##             Primary.Disease                   Lineage           Lineage.Subtype
    ## 1             Neuroblastoma Peripheral Nervous System             Neuroblastoma
    ## 2          Synovial Sarcoma               Soft Tissue          Synovial Sarcoma
    ## 3  Hepatocellular Carcinoma                     Liver  Hepatocellular Carcinoma
    ## 4             Non-Cancerous                    Kidney             Non-Cancerous
    ## 5 Anaplastic Thyroid Cancer                   Thyroid Anaplastic Thyroid Cancer
    ## 6 Anaplastic Thyroid Cancer                   Thyroid Anaplastic Thyroid Cancer
    ##   Expression.Public.23Q4 Mutations_Prioritized
    ## 1               4.732812                 Other
    ## 2               4.395748                 Other
    ## 3               3.785551                 Other
    ## 4               4.409391                 Other
    ## 5               3.539779                 Other
    ## 6               3.051372                 Other

Next we’ll read in a txt file

    # here we specify that the fields are tab delimited with sep = "\t"
    # the default separator for this function is space

    file <- read.table(file = "KRAS CRISPR_DepMap Public 23Q4+Score Chronos.txt", sep = "\t")
    head(file)

    ##           V1                                         V2             V3
    ## 1  Depmap ID CRISPR (DepMap Public 23Q4+Score, Chronos) Cell Line Name
    ## 2 ACH-001300                                -0.57563923         CHLA15
    ## 3 ACH-001322                               -0.553888338           CME1
    ## 4 ACH-001318                               -0.389899033        PLCPRF5
    ## 5 ACH-001310                                -0.61930416           HA1E
    ## 6 ACH-001307                               -0.325479245          8505C
    ##                          V4                        V5                        V6
    ## 1           Primary Disease                   Lineage           Lineage Subtype
    ## 2             Neuroblastoma Peripheral Nervous System             Neuroblastoma
    ## 3          Synovial Sarcoma               Soft Tissue          Synovial Sarcoma
    ## 4  Hepatocellular Carcinoma                     Liver  Hepatocellular Carcinoma
    ## 5             Non-Cancerous                    Kidney             Non-Cancerous
    ## 6 Anaplastic Thyroid Cancer                   Thyroid Anaplastic Thyroid Cancer
    ##                       V7                    V8
    ## 1 Expression Public 23Q4 Mutations_Prioritized
    ## 2            4.732811872                 Other
    ## 3            4.395748328                 Other
    ## 4            3.785550552                 Other
    ## 5            4.409390936                 Other
    ## 6            3.539779192                 Other

Now let’s read in the file saved as an excel file

    # One of the packages I like to use for this readxl
    library(readxl) # first we load the library

    ## Warning: package 'readxl' was built under R version 4.1.3

    file <- read_excel("KRAS CRISPR_DepMap Public 23Q4+Score Chronos.xlsx")
    head(file)

    ## # A tibble: 6 x 8
    ##   `Depmap ID` CRISPR (DepMap Public~1 `Cell Line Name` `Primary Disease` Lineage
    ##   <chr>                         <dbl> <chr>            <chr>             <chr>  
    ## 1 ACH-001300                   -0.576 CHLA15           Neuroblastoma     Periph~
    ## 2 ACH-001322                   -0.554 CME1             Synovial Sarcoma  Soft T~
    ## 3 ACH-001318                   -0.390 PLCPRF5          Hepatocellular C~ Liver  
    ## 4 ACH-001310                   -0.619 HA1E             Non-Cancerous     Kidney 
    ## 5 ACH-001307                   -0.325 8505C            Anaplastic Thyro~ Thyroid
    ## 6 ACH-001306                   -0.441 8305C            Anaplastic Thyro~ Thyroid
    ## # i abbreviated name: 1: `CRISPR (DepMap Public 23Q4+Score, Chronos)`
    ## # i 3 more variables: `Lineage Subtype` <chr>, `Expression Public 23Q4` <dbl>,
    ## #   Mutations_Prioritized <chr>

    # some handy things you can do in read_excel are:
    file <- read_excel("KRAS CRISPR_DepMap Public 23Q4+Score Chronos.xlsx",
                       sheet = "KRAS CRISPR", # specify the sheet name to read in
                       skip = 0) # number of rows to skip before reading in data 

If you run into issues a couple things you can check:  
Is the path to your file the correct path?  
The fields may have a different separator than you specified, is it a semicolon, space, tab?
