+++
title = 'Importance of color scales in heatmaps'
date = 2024-04-21T14:13:06-04:00
draft = false
comments = true
+++


### Importance of color scales when generating a heatmap
Heatmaps can be a really useful way to visualize certain data, like changes in gene expression across samples or time.  However, it's important to always note the color scale used and what units it represents when interpreting your plot.  I recently generated a plot and I initially got really excited about the results until I realized the color scale wasn't representing what I thought it was.  Here's an example of a potentially misleading plot:   

Load our libraries
```r
library(pheatmap)
library(tidyverse)
library(RColorBrewer)
```


For the sake of this example, let's say we're plotting the logFCs of different genes at day 5 vs day 1.  And we want to see how the gene expression changes in untreated and treated samples.  And maybe we hypothesize that this "gene signature" increases after treatment 

First we'll start by generating the matrix we will plot as a heatmap  
```r
nums <- seq(from = -1, to = 3, by = .05)
# <- sample(nums)

mat <- matrix(nums, nrow = 20, ncol = 4)
```
Let's add some rownames and column names
Each row is a gene and each column is a sample
```r
row.names(mat) <- paste0("Gene", 1:nrow(mat))
colnames(mat) <- c("Sample1", "Sample2", "Sample3", "Sample4")
```

Next we want to add some annotations for the heatmap  
We'll say samples 1 and 2 are the untreated samples and Samples 3 and 4 are the treated samples  
```r
anno <- data.frame(status = c("Untreated", "Untreated", "Treated", "Treated"))
row.names(anno) <- colnames(mat)
head(anno)
```

Next let's specifiy the colors we want to use for the annotations on the heatmap
```r
# get colors to use from an RColorBrewer palette
greens <-(rev(brewer.pal(7, "Greens")))

# make treated a dark green color and untreated a light green color
anno_color <- list(
  status= (c(Untreated = greens[5], Treated = greens[2]))
)
```

Next we'll specifiy the color scheme we want to use for the gene logFCs
```r
color_scale <- colorRampPalette(c("navy", "white", "red"))(250)
```

We're ready to plot the heatmap
```r
pheatmap(mat,
         color = color_scale, # blue-white-red color we specified above
         show_rownames = TRUE,
         show_colnames = TRUE,
         cluster_cols = FALSE, # don't cluster the columns, we want untreated and treated to stay together
         annotation_col = anno, # our column annotations
         annotation_colors = anno_color, # our column annotation colors
         border_color = NA, # don't draw lines between cells
         cellwidth = 30, # width of cells
         scale = "none" # don't center and scale the values in either the row or column direction
         )
```

![Plot1](../images/pheatmap_not_centered.png)

Now if we stopped here and looked at this plot, we might make the conclusion that this gene signature is upregulated in the treated group and downregulated in the untreated group and we *might* get super excited and show our PI and collaborators and friends and celebrate!  
BUT if we looked at the scale more carefully, we would see that 0 logFC is actually showing up as blue on our plot and logFC = 1 is showing up as white on our plot.  So what may appear as downregulated or neutral is actually a positive logFC  

To see this effect a little more clearly, we can display the values on the plot and we can see almost all the genes for sample 2 have a positive logFC which might be a very different interpretation than we first had looking at the plot
```r
pheatmap(mat,
         color = color_scale,
         show_rownames = TRUE,
         show_colnames = TRUE,
         cluster_cols = FALSE,
         annotation_col = anno,
         annotation_colors = anno_color,
         border_color = NA,
         cellwidth = 30,
         scale = "none",
         display_numbers = TRUE # show values in each cell
         )
```
![Plot2](../images/pheatmap_uncentered_display_values.png)
So now let's force the scale to be centered at 0, so 0 logFC is white  
We can do this by creating breaks for our color palette  

Definition of breaks from the pheatmap manual:  
a sequence of numbers that covers the range of values in mat and is one element longer than color vector. Used for mapping values to colors. Useful, if needed to map certain values to certain colors, to certain values. If value is NA then the breaks are calculated automatically. When breaks do not cover the range of values, then any value larger than max(breaks) will have the largest color and any value lower than min(breaks) will get the lowest color.  

```r
# length of our color palette
color_length <- 250

# scale from smallest value in our matrix to 0:
## Get all values from the minimum of your matrix to 0 and break that into 126 values (half of our color palette (which we set to have a length of 250) + 1)
neg_scale <- seq(min(mat), 0, length.out=(color_length/2) + 1)

## Get all values from 0.001 to the max of your matrix and break that into 125 values (again half of our color palette length) 
## We start at 0.001 so it doesn't overlap with the negative color scale which includes 0
## You may need to modify that starting value (0.001) depending on your values
pos_scale <- seq(0.001, max(mat), length.out=(color_length/2))

# put it all together:
breaks <- c(neg_scale, pos_scale)


# Or to put it all together at once:
# breaks <- c(seq(min(mat), 0, length.out=(color_length/2) + 1), 
#               seq(0.001, max(mat), length.out=(color_length/2)))
```


```r
pheatmap(mat,
         color = color_scale,
         breaks = breaks, # add breaks to heatmap
         show_rownames = TRUE,
         show_colnames = TRUE,
         cluster_cols = FALSE,
         annotation_col = anno,
         annotation_colors = anno_color,
         border_color = NA,
         cellwidth = 30,
         scale = "none"
         )
```
![Plot3](../images/pheatmap_centered.png)


Now this version of the heatmap looks quite different from first version  
Although it looks like the genes are still upregulated to a larger degree after treatment, we may not be quite as excited after seeing the data visualized with a centered color scale.  I'm not saying a result like this is not interesting, it could still be a very valuable result, but it may give a different meaning than we originally thought.    
If for some reason in our hypothetical world we were looking for a gene signature that ONLY gets upregulated after treatment, this would not be the gene signature to move forward with  

But the main takeaway here is just to notice the difference the color scale can have on the interpretation of your plots
![Plot4](../images/Uncentered_and_centered_plots.png)


Have you found any examples of misleading heatmaps in the wild in your field?  I'd love to hear your feedback and see some published examples.



