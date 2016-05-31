---
layout: post
title: "Facebook Check Ins competition part1: exploratory analysis"
data: 2016-05-30 21:00:00
category: pydata
tags: python pandas kaggle
---
## Overview
This is the first part of my kaggle competition [Facebook V: Predicting Check Ins](https://www.kaggle.com/c/facebook-v-predicting-check-ins).The first step of a competition is to understand the provided datasets and how they are linked to the problem.

In the step, I will inspect the following things:

- basic info about the data set
- variables distribution and correlation
- dependent variables distribution 
- data set split rules 

## Basic info about the data set


```python
import numpy as np
import pandas as pd
#import os
import matplotlib.pyplot as plt
#from scipy.stats import gaussian_kde
#import time
import seaborn as sns 
%matplotlib inline
```


```python
train = pd.read_csv('data/train.csv')
test = pd.read_csv('data/test.csv')
train.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 29118021 entries, 0 to 29118020
    Data columns (total 6 columns):
    row_id      int64
    x           float64
    y           float64
    accuracy    int64
    time        int64
    place_id    int64
    dtypes: float64(2), int64(4)
    memory usage: 1.3 GB



```python
test.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 8607230 entries, 0 to 8607229
    Data columns (total 5 columns):
    row_id      int64
    x           float64
    y           float64
    accuracy    int64
    time        int64
    dtypes: float64(2), int64(3)
    memory usage: 328.3 MB



```python
train.describe()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>row_id</th>
      <th>x</th>
      <th>y</th>
      <th>accuracy</th>
      <th>time</th>
      <th>place_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>2.911802e+07</td>
      <td>2.911802e+07</td>
      <td>2.911802e+07</td>
      <td>2.911802e+07</td>
      <td>2.911802e+07</td>
      <td>2.911802e+07</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>1.455901e+07</td>
      <td>4.999770e+00</td>
      <td>5.001814e+00</td>
      <td>8.284912e+01</td>
      <td>4.170104e+05</td>
      <td>5.493787e+09</td>
    </tr>
    <tr>
      <th>std</th>
      <td>8.405649e+06</td>
      <td>2.857601e+00</td>
      <td>2.887505e+00</td>
      <td>1.147518e+02</td>
      <td>2.311761e+05</td>
      <td>2.611088e+09</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>1.000000e+00</td>
      <td>1.000000e+00</td>
      <td>1.000016e+09</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>7.279505e+06</td>
      <td>2.534700e+00</td>
      <td>2.496700e+00</td>
      <td>2.700000e+01</td>
      <td>2.030570e+05</td>
      <td>3.222911e+09</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>1.455901e+07</td>
      <td>5.009100e+00</td>
      <td>4.988300e+00</td>
      <td>6.200000e+01</td>
      <td>4.339220e+05</td>
      <td>5.518573e+09</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>2.183852e+07</td>
      <td>7.461400e+00</td>
      <td>7.510300e+00</td>
      <td>7.500000e+01</td>
      <td>6.204910e+05</td>
      <td>7.764307e+09</td>
    </tr>
    <tr>
      <th>max</th>
      <td>2.911802e+07</td>
      <td>1.000000e+01</td>
      <td>1.000000e+01</td>
      <td>1.033000e+03</td>
      <td>7.862390e+05</td>
      <td>9.999932e+09</td>
    </tr>
  </tbody>
</table>
</div>




```python
test.describe()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>row_id</th>
      <th>x</th>
      <th>y</th>
      <th>accuracy</th>
      <th>time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>8.607230e+06</td>
      <td>8.607230e+06</td>
      <td>8.607230e+06</td>
      <td>8.607230e+06</td>
      <td>8.607230e+06</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>4.303614e+06</td>
      <td>4.991417e+00</td>
      <td>5.006705e+00</td>
      <td>9.265208e+01</td>
      <td>8.904637e+05</td>
    </tr>
    <tr>
      <th>std</th>
      <td>2.484693e+06</td>
      <td>2.866409e+00</td>
      <td>2.886888e+00</td>
      <td>1.242906e+02</td>
      <td>6.446783e+04</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>1.000000e+00</td>
      <td>7.862420e+05</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2.151807e+06</td>
      <td>2.517000e+00</td>
      <td>2.502400e+00</td>
      <td>4.200000e+01</td>
      <td>8.332200e+05</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>4.303614e+06</td>
      <td>4.988000e+00</td>
      <td>5.000900e+00</td>
      <td>6.400000e+01</td>
      <td>8.874620e+05</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>6.455422e+06</td>
      <td>7.463600e+00</td>
      <td>7.505300e+00</td>
      <td>7.900000e+01</td>
      <td>9.454910e+05</td>
    </tr>
    <tr>
      <th>max</th>
      <td>8.607229e+06</td>
      <td>1.000000e+01</td>
      <td>1.000000e+01</td>
      <td>1.026000e+03</td>
      <td>1.006589e+06</td>
    </tr>
  </tbody>
</table>
</div>



## distribution and correlation


```python
train.shape
```




    (29118021, 6)



Take 10000 samples from 29 million for plot.


```python
from pandas.tools.plotting import scatter_matrix
scatter_matrix(train.sample(10000).iloc[:, 1:], alpha=0.2, figsize=(10, 10), diagonal='kde')
plt.show()
```


![png](/assets/fb_part1/output_9_0.png)


There're patterns in accuracy and time distribution.But for coordinates(x,y), no patterns observed in the whole train set.
Let's take some sample for specific place_id to check whether there's pattern in the distribution.


```python
place_count = train.groupby('place_id').size()
sorted_place = place_count.sort_values(ascending=False)
```


```python
sorted_place.describe()
```




    count    108390.000000
    mean        268.641212
    std         267.944598
    min           1.000000
    25%          98.000000
    50%         163.000000
    75%         333.000000
    max        1849.000000
    dtype: float64




```python
top10_places, last10_places = sorted_place.iloc[:10], sorted_place.iloc[-10:]
rand10_places = sorted_place.sample(10)
```


```python
sample_size = pd.DataFrame({'top10_ids':top10_places.index, 'top10_counts':top10_places.values,
                           'last10_ids':last10_places.index, 'last10_counts':last10_places.values,
                           'rand10_ids':rand10_places.index, 'rand10_counts':rand10_places.values})
sample_cols = sample_size.columns
sample_size[sorted(sample_cols,reverse=True)]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>top10_ids</th>
      <th>top10_counts</th>
      <th>rand10_ids</th>
      <th>rand10_counts</th>
      <th>last10_ids</th>
      <th>last10_counts</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>8772469670</td>
      <td>1849</td>
      <td>2968090761</td>
      <td>41</td>
      <td>4807155845</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1623394281</td>
      <td>1802</td>
      <td>3452964819</td>
      <td>83</td>
      <td>3507798159</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1308450003</td>
      <td>1757</td>
      <td>1803618063</td>
      <td>55</td>
      <td>9091399268</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4823777529</td>
      <td>1738</td>
      <td>7744682740</td>
      <td>830</td>
      <td>2840036477</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>9586338177</td>
      <td>1718</td>
      <td>5250492968</td>
      <td>90</td>
      <td>5714851113</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>9129780742</td>
      <td>1716</td>
      <td>8550631243</td>
      <td>96</td>
      <td>8402352480</td>
      <td>1</td>
    </tr>
    <tr>
      <th>6</th>
      <td>9544215131</td>
      <td>1702</td>
      <td>5908110919</td>
      <td>82</td>
      <td>5369696092</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7</th>
      <td>4638096372</td>
      <td>1699</td>
      <td>1846364461</td>
      <td>88</td>
      <td>9602765305</td>
      <td>1</td>
    </tr>
    <tr>
      <th>8</th>
      <td>5351837004</td>
      <td>1699</td>
      <td>7381107197</td>
      <td>85</td>
      <td>4741604432</td>
      <td>1</td>
    </tr>
    <tr>
      <th>9</th>
      <td>8610202964</td>
      <td>1693</td>
      <td>5427646526</td>
      <td>178</td>
      <td>5964251556</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
samples = train[train.place_id.isin(rand10_places.index)]
samples.shape
```




    (1628, 6)




```python
scatter_matrix(samples.iloc[:, 1:], alpha=0.2, figsize=(10, 10), diagonal='kde')
plt.show()
```


![png](/assets/fb_part1/output_16_0.png)


We can see some obvious pattern emerge here. Let's plot with place_id identified.



```python
cls = ['g','r','c','m','y','b','pink','darkblue','k','purple']
colors = dict(zip(rand10_places.index, cls))
samples[['x','y','place_id']].plot.scatter(x='x',y='y',c=samples.place_id.apply(lambda x:colors[x]),figsize=(10,10))
plt.show()
```


![png](/assets/fb_part1/output_18_0.png)


We see the place_id is distributed on the horizon line. It might be a hint to understand the meaning of the coordinates.


```python
sns.set_context("talk")
g = sns.FacetGrid(samples,col='place_id', col_wrap=5)
g.map(plt.scatter, "x", "y")
```




    <seaborn.axisgrid.FacetGrid at 0x7fecdabf4320>




![png](/assets/fb_part1/output_20_1.png)


Now let's plot the top 10 frequent places.


```python
samples = train[train.place_id.isin(top10_places.index)]
scatter_matrix(samples.iloc[:, 1:], alpha=0.2, figsize=(10, 10), diagonal='kde')
plt.show()
```


![png](/assets/fb_part1/output_22_0.png)



```python
cls = ['g','r','c','m','y','b','pink','darkblue','k','purple']
colors = dict(zip(top10_places.index, cls))
samples[['x','y','place_id']].plot.scatter(x='x',y='y',alpha=0.5,c=samples.place_id.apply(lambda x:colors[x]),figsize=(10,10))
plt.xlim(-0.1,10)
plt.ylim(-0.1,10)
plt.show()
```


![png](/assets/fb_part1/output_23_0.png)



```python
sns.set_context("talk")
g = sns.FacetGrid(samples,col='place_id', col_wrap=5)
g.map(plt.scatter, "x", "y")
```




    <seaborn.axisgrid.FacetGrid at 0x7fecdb875a58>




![png](/assets/fb_part1/output_24_1.png)


## dependent variables distribution


```python
sorted_place.plot.hist(bins=100, title='Train place distribution')
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7fed00078780>




![png](/assets/fb_part1/output_26_1.png)



```python
less_places = sorted_place[sorted_place<11]
less_places.shape
```




    (1278,)




```python
samples = train[train.place_id.isin(less_places.index)]
cls = ['g','r','c','m','y','b','pink','darkblue','k','purple']
samples[['x','y','place_id']].plot.scatter(x='x',y='y',alpha=0.5,c=samples.place_id.apply(lambda x:cls[less_places.ix[x]-1]))
plt.xlim(-0.1,10)
plt.ylim(-0.1,10)
plt.show()
```


![png](/assets/fb_part1/output_28_0.png)


The low frequent places spread all aroud, so we might set a thresh hold to just ignore the low frequent place.

## data set split rules

#### coordinates distribution


```python
fig, axes = plt.subplots(1, 2, figsize=(12, 6));
train.plot.hexbin(ax=axes[0],x='x',y='y',gridsize=50)
axes[0].set_title('Train');
test.plot.hexbin(ax=axes[1],x='x',y='y',gridsize=50)
axes[1].set_title('Test');
```


![png](/assets/fb_part1/output_32_0.png)


For coordinates, the test is extract randomly from the population.

#### accuracy distribution


```python
fig, axes = plt.subplots(1, 2, figsize=(12, 6));
train.sample(10000).accuracy.plot.kde(ax=axes[0])
axes[0].set_title('Train Accuracy');
test.sample(10000).accuracy.plot.kde(ax=axes[1])
axes[1].set_title('Test Accuracy');

```


![png](/assets/fb_part1/output_34_0.png)


#### Time distribution


```python
fig, axes = plt.subplots(1, 2, figsize=(12, 6));
train.sample(10000).time.plot.kde(ax=axes[0])
axes[0].set_title('Train Time');
for tick in axes[0].get_xticklabels():
    tick.set_rotation(45)
test.sample(10000).time.plot.kde(ax=axes[1])
axes[1].set_title('Test Time');
for tick in axes[1].get_xticklabels():
    tick.set_rotation(45)
```


![png](/assets/fb_part1/output_36_0.png)


We see the test time is expanding the the time, let's plot the train and test in one plot.


```python
train_sample = train.sample(10000).iloc[:, 1:5]
test_sample = test.sample(10000).iloc[:,1:]
fig, ax1 = plt.subplots()
t = np.arange(0,110000, 10000)
s1 = np.exp(t)
train_sample.time.plot.kde(ax=ax1)
test_sample.time.plot.kde(ax=ax1)
plt.show()

```


![png](/assets/fb_part1/output_38_0.png)


We see from this plot the time in the test is exceed the time in train set. So the time is a key factor for a good prediction.

## Next step

From the exploratory analysis above, we understand the data information and know some patterns of the variable distribution and correlation. In next step, we will dig into the details of accuray and time to check whether we can extract more useful information for our feature engineering.
