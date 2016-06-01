---
layout: post
title:  "SAS Log anaysis with pandas"
date: 2016-05-27 21:00:00
categories: pydata
tags: python pandas
---
## Overview
One of my routine tasks is to support the product system that executing critical calculation in SAS Platform daily.
We have customized a well defined log system to tracing and profiling the system.

I have done several scripts in python to analyze the logs and extract information about the status of the system.
One issue oberserved is the total time span for one calculation increased steadly.
To figure out the bottle neck of this issue I did an analysis in python and pandas.

## Why Python and Pandas
We have some built in performance report to show the system monitor result. But parsing the log for ad hoc analysis in SAS is feasible but not easy.
With python, its easy to parse the log and extract performance information from log files.

Pandas has built in time series and plotting functions that enable a quick analysis and result reporting.


The first result I got is below:
![Daily performance](/assets/Daily_performance_stacked.png)

The pattern is not obvious, we can get weekly trend with the following piece of code easily.

{% highlight python %} 
final_report_weekly = final_report_daily.resample('W')

ax = final_report_weekly.plot.area(figsize=(12,10),fontsize=12)
plt.ylabel("Seconds")
ax.set_xlabel('Weekly Performance of Reporting ')
fig = ax.get_figure()

fig.savefig('Weekly_performance_stacked.png', bbox_inches='tight')

{% endhighlight %}


![Weekly performance](/assets/Weekly_performance_stacked.png)

The pattern is visible now.
Let's check the substeps separately:

{% highlight python %} 
ax = final_report_weekly.plot(subplots=True, figsize=(10, 10));

fig = ax[0].get_figure()

fig.savefig('Weekly_performance.png', bbox_inches='tight')

{% endhighlight %}


![Weekly performance for substeps](/assets/Weekly_performance.png)

**Note** The **scale** of y axis it not the same.

## Result
From the plot, we know it's the position table step increased most fast.

With Pandas, it's a few lines of code to make it visible. Life is short, we can use Pandas to make the job quick and beauty.
