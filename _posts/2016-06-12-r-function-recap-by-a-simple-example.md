---
layout: post
title: "R function recap by a simple example"
date: 2016-06-12 21:00:00
category: R
tags: R

---

## Overview
Since I move to python for data science, I haven't used R for a while. I tried to recap the R skill, here is the memo for recap the way I created a simple function.




## Problem
When I met the following piece of code, just wonder how I can practice the DRY(don't repeat your self).


```R
library(faraway)
pima$diastolic[pima$diastolic == 0] <- NA
pima$glucose[pima$glucose == 0] <- NA
pima$triceps[pima$triceps == 0] <- NA
pima$insulin[pima$insulin == 0] <- NA
pima$bmi[pima$bmi == 0] <- NA
na.counts <- sapply(pima, function(x) sum(is.na(x)))
print(na.counts)
```

     pregnant   glucose diastolic   triceps   insulin       bmi  diabetes       age 
            0         5        35       227       374        11         0         0 
         test 
            0 


## Test function
Before we create funtion to remove duplicates, write a test function for validation and reload the pima data.


```R
#make a test function
library(RUnit)
test_na <- function(ds){
   nas <- sapply(ds, function(x) sum(is.na(x)))
   if (checkEquals(na.counts, nas)){
       print("NA count is same as no function version.")
   }else{
       
       print("NA count is different from function version. Check the implementation!")
   }
}
#reload original data set
rm(pima)
data(pima)
origin.na.count <- sapply(pima, function(x) sum(is.na(x)))
print(origin.na.count)
```

     pregnant   glucose diastolic   triceps   insulin       bmi  diabetes       age 
            0         0         0         0         0         0         0         0 
         test 
            0 


## First function
My first try is to define a zero.to.na function with data frame and variable names as parameters.


```R
zero.to.na <- function(ds, vars) {
  for( var in vars){
    ds[[var]][ds[[var]]==0] <-NA
  }
  ds
}

vars <- c("diastolic","glucose","triceps","insulin","bmi")
ds <- zero.to.na(pima, vars)
test_na(ds)
```

    [1] "NA count is same as no function version."


## Second function
We can also remove the hard-code of zero to make this function more flexible.


```R
#reload original data set
rm(pima)
data(pima)
replace.na.value <- function(ds, vars, navalue){
  for( var in vars){
    ds[[var]][ds[[var]]==navalue] <-NA
  }
  ds
}

ds1 <- replace.na.value(pima,  vars, 0)
test_na(ds1)
```

    [1] "NA count is same as no function version."


## Third function
We can also use closure to create functions for different na values.


```R
#reload original data set
rm(pima)
data(pima)
replace.na.value <- function(navalue){
  function(ds, vars){
      for( var in vars){
        ds[[var]][ds[[var]]==navalue] <-NA
      }
      ds
  }
}


replace.na.zero <- replace.na.value(0)
ds2 <- replace.na.zero(pima,  vars)
test_na(ds2)
```

    [1] "NA count is same as no function version."


## Last function
We can remove the constraint of the assumption of a data frame input.


```R
#reload original data set
rm(pima)
data(pima)

replace.na.value <- function(navalue){
  function(x){
    x[x==navalue] <- NA
    x
  }
}
replace.na.zero <- replace.na.value(0)
pima[vars] <- replace.na.zero(pima[vars] )
test_na(pima)
```

    [1] "NA count is same as no function version."


## Next Step
There are lots of routine tasks for data wrangling, we can create our own utils to practice the DRY principle.
