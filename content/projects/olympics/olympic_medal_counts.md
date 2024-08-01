+++
title = '2024 Olympic Medal Counts'
date = 2024-07-31T21:00:31-04:00
draft = false
weight = 6
+++


I wanted to see if I could use python to automatically get the medal counts for the 2024 Olympics each day    
Here I'm getting the medal counts from bbc.com and then plot the medal counts for all countries with at least 1 medal   
I run this each morning so I can see how the medal counts change each day

```python
# Import libraries
import requests
from bs4 import BeautifulSoup
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
from datetime import date, timedelta
```


```python
# Enter today's date to keep track of each day as we want to update our plot daily
today = "07312024"
```


```python
# Get today's date - not working correctly on my system, but for others this would be better than having to remember to change the date each day
# today = date.today()
# today = today.strftime("%m-%d-%Y") # Format the date as MM-DD-YYYY
# print(today)
```

## Get the olympic medal counts
Medal counts are scraped each morning, so these are the counts as of the AM of each day


```python
# How to scrape the medal counts with help from ChatGPT
```


```python
# URL of the webpage to scrape for medal counts
url = "https://www.bbc.com/sport/olympics/paris-2024/medals/"

# make the request to get the data
html = requests.get(url).content

# Parse the HTML content using BeautifulSoup and the html5lib parser
soup = BeautifulSoup(html, 'html5lib') # run 'pip install html5lib' if this errors out

# Look through the html results to see how we can identify the portions with the medal counts for each country
# Here can we use this specified class to get the medal info
country_sections = soup.find_all('div', {'class': 'ssrcss-7kfmgb-BadgeContainer ezmsq4q1'})

# Create dictionary to store our medal results
country_medals = {}

# Iterate through all the country sections (there's a section for each country)
for country_section in country_sections:
    # Get the country name
    country_name = country_section.find_next('span', {'class': 'ssrcss-ymac56-CountryName ew4ldjd0'}).get_text(strip=True)
    
    #  Create an empty list to store medal counts for the current country
    medal_counts = []
    
    # Find the first 'td' element next to the current country section
    current = country_section.find_next('td')

    # Iterate over the sibling 'td' elements to get all the medal counts
    ## extracts the text content, checks if it's a digit, and if so adds it to the medal counts list
    while current:
        value = current.get_text(strip=True)
        if value.isdigit():
            medal_counts.append(int(value))
        current = current.find_next_sibling('td')
    
    # Add the medal counts to the dictionary with the country name as the key
    country_medals[country_name] = medal_counts

#print(country_medals)
```

## Convert to a dataframe and store the data


```python
# convert our medal count dictionary to a dataframe
```


```python
df = pd.DataFrame.from_dict(country_medals)

# rotate df so the medals are the columns
df = df.T

# rename columns
new_column_names = ['Gold', 'Silver', 'Bronze', 'Total']
df.columns = new_column_names

df.head()
```


```python
# save as dataframe to keep each day's data
file_name = "./data/Olympic_medal_count_" + today + ".csv"
file_name
df.to_csv(file_name)
```

## Plot the data


```python
# remove countries with 0 medals
plot_df = df[df["Total"] > 0]
plot_df.shape
```

```python
# make the countries a column instead of the rownames
plot_df = plot_df.reset_index().rename(columns={'index': 'Country'})
plot_df.head()
```


```python
# sort the countries by the highest total count
plot_df = plot_df.sort_values('Total', ascending = True)
plot_df.head()
```


```python
# Create the bar plot
Country = plot_df.Country
Gold = plot_df.Gold
Silver = plot_df.Silver
Bronze = plot_df.Bronze
Total = plot_df.Total

# Define width of stacked chart
w = 0.8

# Define figure size
plt.figure(figsize=(12, 8))

# Plot stacked bar chart
plt.barh(Country, Gold, w, color = '#E2C053', label = 'Gold')
plt.barh(Country, Silver, w, left=Gold, color = '#A5ACB4', label = 'Silver')
plt.barh(Country, Bronze, w, left=Gold+Silver, color = '#B6775B', label = 'Bronze')

# Add total number of medals to the end of each bar
for i, (country, total) in enumerate(zip(Country, Total)):
    plt.text(total + 0.1, i, str(total), va='center')

plt.title("Olympic Medal Counts on " + today, size = 18)  # Add title to the plot

# Add legend and set its location to the lower right corner
plt.legend(loc='lower right', fontsize = 20)

# Modify axes
plt.xlabel("Medal Count", size = 14)
     
# Make y-axis labels smaller
plt.tick_params(axis='y', labelsize=8)
plt.tick_params(axis='x', labelsize=14)

# set x-axis tick values
plt.xticks(np.arange(0, max(Total)+1, 5))


# save plot
plot_name = "./Olympic_medal_plots/Olympic_medal_count_" + today + ".png"
plt.savefig(plot_name)  # Save as PNG file

# Display
plt.show()
```


![png](../images/Olympic_medal_count_07-30-2024.png)
![png](../images/Olympic_medal_count_07312024.png)
    



```python
# May need to adjust plot size as more countries get added
```
