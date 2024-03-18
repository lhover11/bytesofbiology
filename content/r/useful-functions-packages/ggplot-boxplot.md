+++
title = 'Customizing ggplots'
date = 2024-02-19T21:48:15-05:00
draft = false
+++

# Step by step creation of a ggplot

I use ggplot for most of my plotting in R.  There are *lots* of different types of plots you can make with ggplot and there are *lots* of customizations you can make.
To see different types of plots and customizations: https://r-graph-gallery.com/ggplot2-package.html

Here is a short overview of some common customizations that you may want to make to your plots


```r
# Load our packages
library(ggplot2)


## Here we're going to use a built-in dataset so you don't need to load anything in
Just run:
data("iris")
head(iris)

# Let's start plotting
ggplot(iris, aes(x = Species, y = Sepal.Length)) +
  geom_boxplot()
```
![Plot 1: Basic Plot](../post1_ggplot_v1.png)
```
# Let's add the indiviual points to the plot so we can see each sample
ggplot(iris, aes(x = Species, y = Sepal.Length)) +
  geom_boxplot() +
  geom_point()

# Next let's make the plot a littler prettier by adding colors
ggplot(iris, aes(x = Species, y = Sepal.Length, fill = Species)) + # fill = Species will fill in the background of my boxplots
  geom_boxplot(alpha = 0.1) + # setting alpha to a low value (between 0-1) will make my boxplots more transparent
  geom_point(aes(color = Species)) +
  # I use scale_fill_manual and scale_color_manual to manually choose my colors to use for the boxplots and the points
  scale_fill_manual(values = c("pink", "purple", "blue")) +
  scale_color_manual(values = c("pink", "purple", "blue"))

# Now let's make the background look better
ggplot(iris, aes(x = Species, y = Sepal.Length, fill = Species)) + 
  geom_boxplot(alpha = 0.1) + 
  geom_point(aes(color = Species)) +
  scale_fill_manual(values = c("pink", "purple", "blue")) +
  scale_color_manual(values = c("pink", "purple", "blue")) +
  theme_classic() # there are lots of different themes you can try out (theme_light, theme_bw, etc)

# Now let's add in our custom X and Y axis labels
ggplot(iris, aes(x = Species, y = Sepal.Length, fill = Species)) + 
  geom_boxplot(alpha = 0.1) + 
  geom_point(aes(color = Species)) +
  scale_fill_manual(values = c("pink", "purple", "blue")) +
  scale_color_manual(values = c("pink", "purple", "blue")) +
  xlab("Iris Species") +
  ylab("Sepal Length") +
  theme_classic()

# Finally we'll add in a title
ggplot(iris, aes(x = Species, y = Sepal.Length, fill = Species)) + 
  geom_boxplot(alpha = 0.1) + 
  geom_point(aes(color = Species)) +
  scale_fill_manual(values = c("pink", "purple", "blue")) +
  scale_color_manual(values = c("pink", "purple", "blue")) +
  xlab("Iris Species") +
  ylab("Sepal Length") +
  ggtitle("Sepal Length by Iris Species") +
  theme_classic()

# Actually one more step, let's make the text bigger
ggplot(iris, aes(x = Species, y = Sepal.Length, fill = Species)) + 
  geom_boxplot(alpha = 0.1) + 
  geom_point(aes(color = Species)) +
  scale_fill_manual(values = c("pink", "purple", "blue")) +
  scale_color_manual(values = c("pink", "purple", "blue")) +
  xlab("Iris Species") +
  ylab("Sepal Length") +
  ggtitle("Sepal Length by Iris Species") +
  theme_classic() +
  theme(text = element_text(size = 18)) # This makes all our plot text bigger, but you can also modify each text element separately

# Save our final plot
ggsave("ggplot_boxplot_example.png", w = 6, h = 6) # this will save the plot wherever you have set your working directory
  ```
![Plot 7: Basic Plot](../ggplot_boxplot_example.png)



Troubleshooting:
If ggplot complains or doesn't add one of your features it might be because you forgot a "+" at the end of a line












```










