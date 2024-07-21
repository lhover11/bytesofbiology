+++
title = 'Billboard Top 100 barplot'
date = 2024-07-21T13:58:20-04:00
draft = false
comments = true
+++

``` r
# Get data:
## Link to data here: https://www.kaggle.com/datasets/suparnabiswas/billboard-hot-1002000-2023-data-with-features?resource=download
## Data from Kaggle: Compilation of Billboard Hot-100 songs chart with lyrics and Spotify features

# Load packages
library(tidyverse)

# set your working directory
## Make sure your data is in your working directory
setwd("~/Bioinformatics/R_videos")


# read in data
data <- read.csv("billboard_top100_2000_2023.csv")

# Make sure data is in long format
data[1:5, 1:5]


# Look at column names of data
names(data)


# Explore the data

## how many artists had top 100 song
length(unique(data$band_singer)) # 1039

## how many times did each artist have a top 100 song
top <- data %>% group_by(band_singer) %>%
  summarize(count = n()) %>%
  arrange(-count)


# Plot data
## Is there a correlation between danceability and ranking?  
ggplot(data, aes(x = danceability, y = ranking)) +
  geom_point() +
  theme_classic() # add theme


## Plot artists with most hits
## Plot top 20 artists

ggplot(top %>% top_n(n = 20,  wt = count), aes(x = reorder(band_singer, -count), y = count, fill = count)) +
  geom_col() +
  theme_classic() +
  theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust=1), # rotate x axis and make text bigger
        axis.title.x = element_blank(),
        text=element_text(size=20)) + # remove x axis title
  scale_fill_gradient(low = "goldenrod", high = "red") + # add color scale
  ggtitle("Artists with most top 100 hits from 2000-2023")
  
ggsave("./plots/billboard_top100_2000_2023_RdOr_barplot.png", w = 10, h = 8)

```
![barplot](../plots/billboard_top100_2000_2023_RdOr_barplot.png "barplot")