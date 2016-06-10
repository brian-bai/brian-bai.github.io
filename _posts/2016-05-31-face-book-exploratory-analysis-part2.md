---
layout: post
title: "Facebook Check Ins competition part2: exploratory analysis"
date: 2016-05-31 21:00:00
category: pydata
tags: python pandas kaggle
---
## Overview
This is the second part of my kaggle competition [Facebook V: Predicting Check Ins](https://www.kaggle.com/c/facebook-v-predicting-check-ins).

In the step, I will inspect the data set to get some insight about accuracy and time.

## Accuracy


```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns 
%matplotlib inline
train = pd.read_csv('data/train.csv')
test = pd.read_csv('data/test.csv')
```


```python
#top1 place id : 8772469670
top1_places = train[train['place_id']==8772469670]
```


```python
top1_places.describe()
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
      <td>1.849000e+03</td>
      <td>1849.000000</td>
      <td>1849.000000</td>
      <td>1849.000000</td>
      <td>1849.000000</td>
      <td>1.849000e+03</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>1.442055e+07</td>
      <td>2.946995</td>
      <td>8.287942</td>
      <td>71.382369</td>
      <td>422902.660357</td>
      <td>8.772470e+09</td>
    </tr>
    <tr>
      <th>std</th>
      <td>8.458861e+06</td>
      <td>0.356140</td>
      <td>0.011544</td>
      <td>92.131766</td>
      <td>161314.022878</td>
      <td>0.000000e+00</td>
    </tr>
    <tr>
      <th>min</th>
      <td>5.334000e+03</td>
      <td>0.069300</td>
      <td>8.177200</td>
      <td>1.000000</td>
      <td>51.000000</td>
      <td>8.772470e+09</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>7.099431e+06</td>
      <td>2.873900</td>
      <td>8.281200</td>
      <td>37.000000</td>
      <td>341688.000000</td>
      <td>8.772470e+09</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>1.426620e+07</td>
      <td>2.907000</td>
      <td>8.287800</td>
      <td>61.000000</td>
      <td>476978.000000</td>
      <td>8.772470e+09</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>2.191178e+07</td>
      <td>2.981400</td>
      <td>8.294700</td>
      <td>70.000000</td>
      <td>522984.000000</td>
      <td>8.772470e+09</td>
    </tr>
    <tr>
      <th>max</th>
      <td>2.911272e+07</td>
      <td>9.401100</td>
      <td>8.395800</td>
      <td>987.000000</td>
      <td>649726.000000</td>
      <td>8.772470e+09</td>
    </tr>
  </tbody>
</table>
</div>



We got the descriptive information about the most frequent place, let's add accuracy to the plot.


```python
top1_places.plot.scatter(x='x',y='y',alpha=0.5,s=top1_places.accuracy)
plt.show()
```


![png](/assets/fb_part2/output_6_0.png)


No visible pattern here, let's try 3D plot.


```python
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D
fig = plt.figure(figsize=(8,8))
ax = fig.add_subplot(111, projection='3d')
ax.scatter(top1_places.x, top1_places.y, top1_places.accuracy, c=top1_places.accuracy,cmap='jet')
ax.set_xlabel('x')
ax.set_ylabel('y')
ax.set_zlabel('Accuracy')
plt.title("Most frequent place distribution over Accuracy")
plt.show()
```


![png](/assets/fb_part2/output_8_0.png)


From this plot, we know **the high accuracy points are more likely to appear near the centroid**.

Let's plot the top 10 frequent place to check whether this pattern can be applied to all places.


```python
place_count = train.groupby('place_id').size()
sorted_place = place_count.sort_values(ascending=False)
top10_places = sorted_place.iloc[:10]
samples = train[train.place_id.isin(top10_places.index)]

cls = ['g','r','c','m','y','b','pink','darkblue','k','purple']
colors = dict(zip(top10_places.index, cls))

fig = plt.figure(figsize=(10,10))
ax = fig.add_subplot(111, projection='3d')
ax.scatter(samples.x, samples.y, samples.accuracy, c=samples.place_id.apply(lambda x:colors[x]))
ax.set_xlabel('x')
ax.set_ylabel('y')
ax.set_zlabel('Accuracy')
plt.xlim(-0.1,10)
plt.ylim(-0.1,10)
plt.title("Top 10 frequent place distribution over Accuracy")
plt.show()
```


![png](/assets/fb_part2/output_11_0.png)


The same rule applies to the other places. 

We can also see that the variance of x is bigger than variance of y.

## Time


```python
fig = plt.figure(figsize=(8,8))
ax = fig.add_subplot(111, projection='3d')
ax.scatter(top1_places.x, top1_places.y, top1_places.time, c=top1_places.time, cmap='jet')
ax.set_xlabel('x')
ax.set_ylabel('y')
ax.set_zlabel('time')
plt.title("Most frequent place distribution over Time")
plt.show()
```


![png](/assets/fb_part2/output_14_0.png)


The distribution vary over time. Let's plot the top 10 frequent place to check whether there are common patterns.


```python
fig = plt.figure(figsize=(10,10))
ax = fig.add_subplot(111, projection='3d')
ax.scatter(samples.x, samples.y, samples.time, c=samples.place_id.apply(lambda x:colors[x]))
ax.set_xlabel('x')
ax.set_ylabel('y')
ax.set_zlabel('time')
plt.xlim(-0.1,10)
plt.ylim(-0.1,10)
plt.title("Top 10 frequent place distribution over Time")
plt.show()
```


![png](/assets/fb_part2/output_16_0.png)


The patterns of the distribution over time is not same for different places.

## RadViz
We use RadViz plot to have a view on different places.


```python
from pandas.tools.plotting import radviz
plt.figure(figsize=(10,10))
radviz(samples.loc[:, ['x','y','accuracy','time','place_id']], 'place_id')
plt.show()
```


![png](/assets/fb_part2/output_18_0.png)


From this plot, we know the all the the variables should be used to predict the tartget.

## Result
In this part, we get the following insight:

- the high accuracy points are more likely to appear near the centroid
- the distribution vary over time, the patterns are different for different places.
- variance of x is bigger than variance of y

## Next Step
We already got some insight about the distribution of places over x,y,accuracy and time. But the pattern of distribution over time is not clear. We will focus on the time in next step.
