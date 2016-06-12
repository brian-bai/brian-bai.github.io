---
layout: post
title: "Facebook Check Ins competition part3: All about the time"
date: 2016-06-11 21:00:00
category: pydata
tags: python pandas kaggle

---

## Overview
This is the third part of my kaggle competition [Facebook V: Predicting Check Ins](https://www.kaggle.com/c/facebook-v-predicting-check-ins).

In the step, I will inspect the data set to get some insight about the time.



```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from matplotlib.pylab import rcParams
%matplotlib inline
rcParams['figure.figsize'] = 10, 6
train = pd.read_csv('data/train.csv')
test = pd.read_csv('data/test.csv')
```

## Range of time


```python
train.time.count(),train.time.min(), train.time.max()
```




    (29118021, 1, 786239)




```python
test.time.count(), test.time.min(), test.time.max()
```




    (8607230, 786242, 1006589)



We know the train and test data sets are splitted by time.The unit of time is not given, we can check the value by assuming its second or minutes.


```python
#in case of second
1006589/60/60/24
```




    11.650335648148149




```python
#in case of minutes
1006589/60/24
```




    699.0201388888889



## Make a Time Series
We assume the time unit is minute, then make a time series in pandas to check the asssumption.


```python
#get histgram data in hours
checkins, bins = np.histogram(train.time, bins=range(0,train.time.max()+60,60))
hour_rng = pd.date_range('1/1/2011', periods=len(bins)-1, freq='H')
#take first 14 days 
hts = pd.Series(checkins, hour_rng).ix[:24*14]
hts.plot()
plt.show()
```


![png](/assets/fb_part3/output_10_0.png)


## Dickey-Fuller Test
We check the time series with  [Dickey-Fuller Test](http://statsmodels.sourceforge.net/stable/generated/statsmodels.tsa.stattools.adfuller.html).


```python
from statsmodels.tsa.stattools import adfuller
def test_stationarity(timeseries):
    rolmean = timeseries.rolling(center=False,window=24).mean()
    rolstd = timeseries.rolling(center=False,window=24).std()

    orig = plt.plot(timeseries, color='blue',label='Original')
    mean = plt.plot(rolmean, color='red', label='Rolling Mean')
    std = plt.plot(rolstd, color='black', label = 'Rolling Std')
    plt.legend(loc='best')
    plt.title('Rolling Mean & Standard Deviation')
    plt.show(block=False)
    
    print('Results of Dickey-Fuller Test:')
    dftest = adfuller(timeseries, autolag='AIC')
    dfoutput = pd.Series(dftest[0:4], index=['Test Statistic','p-value','#Lags Used','Number of Observations Used'])
    for key,value in dftest[4].items():
        dfoutput['Critical Value (%s)'%key] = value
    print(dfoutput)
test_stationarity(hts)
```


![png](/assets/fb_part3/output_12_0.png)


    Results of Dickey-Fuller Test:
    Test Statistic                  -3.743888
    p-value                          0.003537
    #Lags Used                       5.000000
    Number of Observations Used    330.000000
    Critical Value (5%)             -2.870338
    Critical Value (1%)             -3.450322
    Critical Value (10%)            -2.571458
    dtype: float64


The p-vale is less than 0.05 and test statistic -3.74 is smaller than the 1% Critical Value. So we know it's stationary with 99% confidence.

## Decompose the Time Series



```python
from statsmodels.tsa.seasonal import seasonal_decompose
decomposition = seasonal_decompose(hts.values, freq=24)

trend = decomposition.trend
seasonal = decomposition.seasonal
residual = decomposition.resid

plt.subplot(411)
plt.plot(hts, label='Original')
plt.legend(loc='upper right')
plt.subplot(412)
plt.plot(trend, label='Trend')
plt.legend(loc='upper right')
plt.subplot(413)
plt.plot(seasonal,label='Seasonality')
plt.legend(loc='upper right')
for i in range(14):
    plt.axvline(i*24, color='r', linestyle='dashed')
plt.subplot(414)
plt.plot(residual, label='Residuals')
plt.legend(loc='upper right')
plt.tight_layout()
```


![png](/assets/fb_part3/output_15_0.png)


From the plot, the daily seasonality is visible. So the assumption of minute unit is accept.

## FFT
Another way to check the time unit is through Fourier transform. This method is discussed [here](https://www.kaggle.com/leonlu/facebook-v-predicting-check-ins/another-way-to-know-the-time-definition).


```python
# FFT for most frequent place id
time = train[train['place_id']==8772469670]['time']
hist = np.histogram(time,5000)
hist_fft = np.absolute(np.fft.fft(hist[0]))
top2 = np.argsort(hist_fft)[-2:]
for i in top2:
    plt.axvline(i, color='r', linestyle='dashed')
plt.plot(hist_fft)
plt.xlim([0,2500])
plt.title('FFT of event time histogram')
plt.xlabel('1/T')
plt.grid(True)
plt.show()
```


![png](/assets/fb_part3/output_18_0.png)


Peak at 451 is highlighted.


```python
time.max()/451, 24*60
```




    (1440.6341463414635, 1440)



So we know the time unit is minute.

## Result 
Through time series analysis, we conclude that the time unit is in minute.

## Next Step
In next step, we will create new features based on time column and start to build model for this competition.
