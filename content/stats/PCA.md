+++
title = 'PCA'
date = 2024-04-27T14:30:02-04:00
draft = false
comments = true
+++

### PCA analysis
What is PCA analysis and why use it?
PCA stands for Principal Component Analysis and it can help us understand complex datasets through what is called dimensional reduction

PCA takes a set of data with many variables (also called features -- in bioinformatics this will often be something like RNAseq data where you have a set of samples with expression values for many genes (>20K)) and identifies the most important underlying patterns which capture the most variance

PCA does this by combining the features (or in our example, genes) into a smaller set of variables called principal components (or PCs).  These new variables are the weighted linear combinations of the original features.  This smaller set of new variables "explain" as much of the variability of the original data as they can.  This is how we've reduced the dimensionality of the data - we've gone from trying to look at and understand 20k+ features to maybe somewhere around 10

The weights in the linear combinations show the relative contributions of the original variables/features/genes to the new principal components (note: the weights are called loadings)

The first principal component is the linear combination that best explains the variance.  Prinicpal component 2 explains the next largest percentage of the variance and so on and so forth

If you want a great explanation of the underlying mathmematics performed in PCA, I would recommend watching the StatQuest video: https://www.youtube.com/watch?v=FgakZw6K1QQ


Running PCA can easily be done in both R and Python.  
Here we'll use scikit-learn but you can find lots of online tutorials for doing PCA in both R and python


```python
# Step 1: import libraries needed
import pandas as pd
import numpy as np

import matplotlib.pyplot as plt
# %matplotlib inline  # this tells Jupyter to display Matplotlib plots directly within the notebook and means we also don't have to call plt.show()

from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler
```

For practice, we'll use the Breast cancer wisconsin (diagnostic) dataset from scikit-learn
This dataset consists of data from 569 patient and contains 30 features from a digitized image of a fine needle aspirate from a breast mass.  The features describe characterisitcs of the cell nuclei in the image  

There are 2 classes in the data: Malignant and Benign


```python
# Load in the dataset
from sklearn.datasets import load_breast_cancer
data = load_breast_cancer()
 
data.keys()

```




    dict_keys(['data', 'target', 'frame', 'target_names', 'DESCR', 'feature_names', 'filename', 'data_module'])




```python
# Let's explore the data a bit
```


```python
# Description of the data
print(data['DESCR'])
```


```python
# What's in target and target_names?
print(data['target'][:10]) # show first 10 values
print(data['target_names'])
```

    [0 0 0 0 0 0 0 0 0 0]
    ['malignant' 'benign']



```python
# ok so it looks like target and target names shows us our classes 
```


```python
# Look at the features
print(data['feature_names'])
```

    ['mean radius' 'mean texture' 'mean perimeter' 'mean area'
     'mean smoothness' 'mean compactness' 'mean concavity'
     'mean concave points' 'mean symmetry' 'mean fractal dimension'
     'radius error' 'texture error' 'perimeter error' 'area error'
     'smoothness error' 'compactness error' 'concavity error'
     'concave points error' 'symmetry error' 'fractal dimension error'
     'worst radius' 'worst texture' 'worst perimeter' 'worst area'
     'worst smoothness' 'worst compactness' 'worst concavity'
     'worst concave points' 'worst symmetry' 'worst fractal dimension']



```python
# Ok so now that we have a handle on the format of the data, let's get into the PCA
```


```python
# First create a dataframe of the needle biopsy features
df = pd.DataFrame(data['data'],columns=data['feature_names'])
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mean radius</th>
      <th>mean texture</th>
      <th>mean perimeter</th>
      <th>mean area</th>
      <th>mean smoothness</th>
      <th>mean compactness</th>
      <th>mean concavity</th>
      <th>mean concave points</th>
      <th>mean symmetry</th>
      <th>mean fractal dimension</th>
      <th>...</th>
      <th>worst radius</th>
      <th>worst texture</th>
      <th>worst perimeter</th>
      <th>worst area</th>
      <th>worst smoothness</th>
      <th>worst compactness</th>
      <th>worst concavity</th>
      <th>worst concave points</th>
      <th>worst symmetry</th>
      <th>worst fractal dimension</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>17.99</td>
      <td>10.38</td>
      <td>122.80</td>
      <td>1001.0</td>
      <td>0.11840</td>
      <td>0.27760</td>
      <td>0.3001</td>
      <td>0.14710</td>
      <td>0.2419</td>
      <td>0.07871</td>
      <td>...</td>
      <td>25.38</td>
      <td>17.33</td>
      <td>184.60</td>
      <td>2019.0</td>
      <td>0.1622</td>
      <td>0.6656</td>
      <td>0.7119</td>
      <td>0.2654</td>
      <td>0.4601</td>
      <td>0.11890</td>
    </tr>
    <tr>
      <th>1</th>
      <td>20.57</td>
      <td>17.77</td>
      <td>132.90</td>
      <td>1326.0</td>
      <td>0.08474</td>
      <td>0.07864</td>
      <td>0.0869</td>
      <td>0.07017</td>
      <td>0.1812</td>
      <td>0.05667</td>
      <td>...</td>
      <td>24.99</td>
      <td>23.41</td>
      <td>158.80</td>
      <td>1956.0</td>
      <td>0.1238</td>
      <td>0.1866</td>
      <td>0.2416</td>
      <td>0.1860</td>
      <td>0.2750</td>
      <td>0.08902</td>
    </tr>
    <tr>
      <th>2</th>
      <td>19.69</td>
      <td>21.25</td>
      <td>130.00</td>
      <td>1203.0</td>
      <td>0.10960</td>
      <td>0.15990</td>
      <td>0.1974</td>
      <td>0.12790</td>
      <td>0.2069</td>
      <td>0.05999</td>
      <td>...</td>
      <td>23.57</td>
      <td>25.53</td>
      <td>152.50</td>
      <td>1709.0</td>
      <td>0.1444</td>
      <td>0.4245</td>
      <td>0.4504</td>
      <td>0.2430</td>
      <td>0.3613</td>
      <td>0.08758</td>
    </tr>
    <tr>
      <th>3</th>
      <td>11.42</td>
      <td>20.38</td>
      <td>77.58</td>
      <td>386.1</td>
      <td>0.14250</td>
      <td>0.28390</td>
      <td>0.2414</td>
      <td>0.10520</td>
      <td>0.2597</td>
      <td>0.09744</td>
      <td>...</td>
      <td>14.91</td>
      <td>26.50</td>
      <td>98.87</td>
      <td>567.7</td>
      <td>0.2098</td>
      <td>0.8663</td>
      <td>0.6869</td>
      <td>0.2575</td>
      <td>0.6638</td>
      <td>0.17300</td>
    </tr>
    <tr>
      <th>4</th>
      <td>20.29</td>
      <td>14.34</td>
      <td>135.10</td>
      <td>1297.0</td>
      <td>0.10030</td>
      <td>0.13280</td>
      <td>0.1980</td>
      <td>0.10430</td>
      <td>0.1809</td>
      <td>0.05883</td>
      <td>...</td>
      <td>22.54</td>
      <td>16.67</td>
      <td>152.20</td>
      <td>1575.0</td>
      <td>0.1374</td>
      <td>0.2050</td>
      <td>0.4000</td>
      <td>0.1625</td>
      <td>0.2364</td>
      <td>0.07678</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 30 columns</p>
</div>



### Next we need to scale the data
if we use StandardScaler that will:  
center the data by subtracting the mean of each feature  
divide each feature by its standard deviation so each feature will have a variance of one


```python
scaling = StandardScaler()

# First we need to fit the scaler to the data so it can calculate the mean and sd for each feature
scaling.fit(df)

# apply the scaling transformation
scaled_data = scaling.transform(df)
```


```python
# let's prove to ourselves that the data was scaled as described above
```


```python
mean_data = df['mean radius'].mean()
print(mean_data)
sd_data = df['mean radius'].std()
print(sd_data)
```

    14.127291739894552
    3.5240488262120775



```python
# subtract the mean from each value
mean_radius_transformed = df['mean radius']-mean_data

# divide each value by the standard deviation
mean_radius_transformed = mean_radius_transformed/sd_data
```


```python
# compare our manual scaling to the scikit learn scaling to make sure we know what's going on
mean_radius_transformed[:5]
```




    0    1.096100
    1    1.828212
    2    1.578499
    3   -0.768233
    4    1.748758
    Name: mean radius, dtype: float64




```python
# show first 5 values of mean radius after fitting scikit-learn scaling
pd.DataFrame(scaled_data).iloc[:5, 0]
```




    0    1.097064
    1    1.829821
    2    1.579888
    3   -0.768909
    4    1.750297
    Name: 0, dtype: float64




```python
# They match, so now we understand what the scaling is doing to our data prior to the PCA
```


```python
# Next we can pick the number of components, we'll choose to keep 10 here
```


```python
# number of components to keep
pca = PCA(n_components = 10)

# fit and transform the data 
pca.fit(scaled_data)
x = pca.transform(scaled_data)
 
# Check the dimensions of data after PCA
print(x.shape)
```

    (569, 10)



```python
# Plot PCA
```


```python
# Get percent of variance explained by each PC, this will be explained below the plot
perc1 = str(round(pca.explained_variance_ratio_[0]*100,2))
perc2 = str(round(pca.explained_variance_ratio_[1]*100,2))
```


```python
# Plot the PCA and color the points by the tumor class (malignant or benign)
plt.figure(figsize=(10,10))

# plot first 2 dimensions
# color samples by target (benign or malignant)
scatter = plt.scatter(x[:,0],x[:,1], c=data['target'], cmap = 'tab20b') 
plt.xlabel('PC1 ' + perc1 + '%', fontsize = 16)
plt.ylabel('PC2 ' + perc2 + '%', fontsize = 16)

# add legend for coloration
plt.legend(*scatter.legend_elements(), fontsize = 16)

# Adjust tick label font size
plt.tick_params(axis='both', which='major', labelsize=14)  # Set font size for tick labels
```


    
![png](../PCA_scikitlearn_files/PCA_scikitlearn_24_0.png)
    



```python
# Here we can clearly see that there's some clustering by sample type (benign vs malignant samples)
```


```python
# Next we can plot the variance explained by each principal component

plt.bar(
    range(1,len(pca.explained_variance_ratio_)+1),
    pca.explained_variance_ratio_,
    color = 'purple'
    )
 
 
plt.xlabel('PCA Feature')
plt.ylabel('Proportion of Variance Explained')
plt.title('Proportion of Variance Explained in top 10 PCs')
plt.show()
```


    
![png](../PCA_scikitlearn_files/PCA_scikitlearn_26_0.png)
    



```python
# QC check 
# As expected, PC1 explains the most variance, then PC2 and so on which matches what we would expect

# We can see PC1 explains about 40% of the variance and PC2 explains about 20% of the variance
# Together, PC1 and 2 account for >60% of the variance which is a lot!

# We would definitely want to drill down into which features contribute most to PC1 and 2
```


```python
# I noticed there was both a "explained_variance" and "explained_variance_ratio", let's compare those two values
```


```python
# explained_variance_
print(pca.explained_variance_)

# This shows the amount of variance captured by each principal component in the PCA
# these values are in the same units as the data's variance and represent absolute variance values
```

    [13.30499079  5.7013746   2.82291016  1.98412752  1.65163324  1.20948224
      0.67640888  0.47745625  0.41762878  0.35131087]



```python
# explained_variance_ratio_
print(pca.explained_variance_ratio_)
# This however shows the ratio of how much variance is explained by each principal component
# This is helpful to see what proportion of the total variance each component explains
```

    [0.44272026 0.18971182 0.09393163 0.06602135 0.05495768 0.04024522
     0.02250734 0.01588724 0.01389649 0.01168978]



```python
# Since we only kept 10 PCs, lets make sure this doesn't sum up to 100% as we don't want the proportion of variance explained just among the PCs we chose
(pca.explained_variance_ratio_).sum()
```




    0.9515688143336781




```python
# Next we want to look at the important features for PC1

# Get the loadings for PC1
loadings = pca.components_[0]  # First row represents PC1
print(loadings)

# Get the feature names
feature_names = df.columns

# Create a DataFrame to pair feature names with loadings
pc1_importance = pd.DataFrame({
    'Feature': feature_names,
    'Loading': loadings
})

# Sort by the absolute value of the loadings (we want to look at the largest + and - values)
pc1_importance['Absolute Loading'] = pc1_importance['Loading'].abs()
pc1_importance = pc1_importance.sort_values(by='Absolute Loading', ascending=False)

print(pc1_importance.head())
```

    [0.21890244 0.10372458 0.22753729 0.22099499 0.14258969 0.23928535
     0.25840048 0.26085376 0.13816696 0.06436335 0.20597878 0.01742803
     0.21132592 0.20286964 0.01453145 0.17039345 0.15358979 0.1834174
     0.04249842 0.10256832 0.22799663 0.10446933 0.23663968 0.22487053
     0.12795256 0.21009588 0.22876753 0.25088597 0.12290456 0.13178394]
                     Feature   Loading  Absolute Loading
    7    mean concave points  0.260854          0.260854
    6         mean concavity  0.258400          0.258400
    27  worst concave points  0.250886          0.250886
    5       mean compactness  0.239285          0.239285
    22       worst perimeter  0.236640          0.236640



```python
# We can see mean concave points, mean concavity and worst concave points are the top 3 contributors to PC1 and may be worth following up on
# Although we can see the loadings for the top 5 here are all pretty similar, so they all may be important features to consider
```

And there you have a PCA analysis from start to finish!

