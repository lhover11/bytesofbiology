+++
title = 'Download R and R Studio'
date = 2024-05-11T14:39:31-04:00
draft = false
comments = true
+++


You're ready to get started coding in R!!!  YAAAAY!!!!

![jpg](../images/pineapple-supply-co.jpg "Yay")
Photo by Pineapple Supply Co. on Unsplash


### Part I: Install R
First you'll need to install R:  
You'll want to do this from The Comprehensive R Archive Network or CRAN  
https://cran.r-project.org/

![png](../images/cran_screenshot1.png "CRAN")  

Click on the appropriate link for your machine (Windows or Mac)  

#### For Windows machines:
Click on base on the next screen  
![png](../images/cran_screenshot2.png "CRAN_base")  

Then click on the Download R-4.4.0 for Windows (this version will continue to be updated so your screen may look different)  
![png](../images/cran_screenshot3.png "CRAN_Win")  

Once the installation program has downloaded, open it up and install R.  I would recommend installing it with the default settings   

#### For a Mac
note: untested as I do not own a Mac, but this follows other online tutorials:  
To install R on a Mac, after clicking on Download R for macOS, click on   
For Apple silicon (M1-3) Macs:  
R-4.4.0-arm64.pkg  
(again, the version will continue to be updated, so you may see a different version here)  
![png](../images/cran_screenshot4.png "CRAN_Mac")

Then open the installation guide to install R  


### Part II: Install RStudio
Next, to work with R, you'll probably want to install RStudio.  You can download the free desktop version  
https://posit.co/downloads/  

![png](../images/rstudio.png "rstudio")


### Part III: Open RStudio

Here's where the magic begins:  

When you first open RStudio, you may see 3 panes:  
On the left you see 3 tabs at the top: Console, Terminal and Jobs  
On the top right you see 4 tabs: Environment, History, Connections and Tutorial  
On the bottom right you see 5 tabs: Files, Plots, Packages, Help and Viewer  
![png](../images/Rstudio_screenshot1.png "rstudio")  


These will make a lot more sense once you start using RStudio, so let's jump into it  

First click File --> New File --> R Script   
You'll see now a new pane has opened called Untitled 1  
This is where you'll write your code  

To run your code, put your mouse at the beginning of your line of code or highlight all lines of code you want to run and click the Run button 
![png](../images/Rstudio_screenshot2.png "rstudio")  

Or to run a line of code, use the keyboard shortcut **Ctrl + Enter**  

Now you can try out all sorts of code like the following:  

Copy the following code into your R script and hit "Run"  
```r
# basic math:
2 + 2

# create variables (can be numeric or comprised of characters)
variable1 <- c(1,2,3,4)
variable2 <- c("a", "b", "c", "d")

# see what sample datasets are available in R
data()

# Create plots
## rivers is a dataset available in R
## create histogram of River lengths
hist(rivers)
```

By running the code above you can see how the code you write in the top left pane is run in the Console on the bottom left  

You can see the variables you create are in the "Environment" panel on the top right.  You can continue to use those variables throughout your script  

Plots show up in the "Plots" panel on the bottom right

Now you've sucessfully installed R and Rstudio AND started to code!  
Keep exploring RStudio and check out my next posts on installing packages and loading in data!  

Bonus: if you want to learn more R, copy, paste and run the following lines of code 
```r
# Install the swirl package -- only need to install swirl the first time you run it
install.packages("swirl")

# load the swirl library
library("swirl")

# Call the swirl function to start learning
swirl()
```
https://swirlstats.com/


![jpg](../images/everyonecancode.jpg "coding!")  
Photo by Adi Goldstein on Unsplash












