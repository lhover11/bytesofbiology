+++
title = 'Set your WD and install packages'
date = 2024-02-20T13:42:10-05:00
draft = false
+++

### Setting your working directory and installing libraries the easy(ier) way

#### Setting your working directory

I remember very distinctly in my first R course, at the beginning I couldn't do any of the assignments because I couldn't figure out how to set my working directory.  
So here I'll show you the easy way to do it in Rstudio so you don't get stuck where I did


Find the Session tab at the top of your Rtudio  
Click on Session >> Set working directory >> Choose Directory

![Set WD](../SetWD.png "Install Packages")

Then you can choose whichever directory on your computer you want to be your working directory.  
The working directory is important because it tells R where to find your files that you try to load in.  

So for example, if I'm trying to load in the excel file: C:\Users\user1\Documents\Bioinformatics\MyVeryImportantData.xlsx
I can set my working directory to: C:\Users\user1\Documents\Bioinformatics  
And then I can read in my data as read_excel("MyVeryImportantData.xslx")

---

#### Installing Packages
You can do lots of things in base R, but pretty much guaranteed you will want to install packages along the way.  One way to do this is:  
Go to the Packages tab in the pane on the bottom right >> install >> start typing in the package you want to install

![Install packages](R_getting_started/install_lib.png "Install Packages")
The "Install from" should be Repository (CRAN)  
And leave the check by Install dependencies

You will also probably want to install bioconductor packages during your work  
If you haven't checked out bioconductor yet... https://bioconductor.org/

Let's say you want to analyze some RNAseq data using DESeq2.  
You can A. find simple installation instructions on the bioconductor website (like I did below)   
And B. you can install it by running the following in R:

```r
if (!require("BiocManager", quietly = TRUE))
    install.packages("BiocManager")

BiocManager::install("DESeq2")
```

---

### Finally, don't forget to load your libraries
Once you have installed your library, in order to use it you'll need to load it first.  
This is done by running:
```r  
library(dplyr)  
library(your-package)
```