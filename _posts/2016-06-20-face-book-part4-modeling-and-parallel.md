---
layout: post
title: "Facebook Check Ins competition part4: Modeling and Parallel Processing"
date: 2016-06-20 21:00:00
category: R
tags: R kaggle parallel

---

## Overview
This is the fourth part of my kaggle competition [Facebook V: Predicting Check Ins](https://www.kaggle.com/c/facebook-v-predicting-check-ins).

In the step, I will process the data for modeling. The competition will be end soon. A common solution that discussed a lot in the forum is to split the data to small blocks and apply machine learning method on each block. In this post, I will try this method and check the multicore processing to accerlate the process time.



## Data Split
The following function is used to split the data set to small blocks.


```R
data_spliter <- function(xx, fb){
  function(yy){
    fb %>% filter(x > xx-span-eps, x < xx+eps, y > yy-span-eps, y < yy+eps) -> sb
    write.csv(sb, file = paste0("pdblocks/",ds, "_block_",xx, "x",yy,".csv"), row.names = FALSE)
  }
}
eps = 0.00001
span = 0.2
ds="train"
for(i in seq(span,10,by=span)){
  sapply(seq(span,10,by=span), data_spliter(i,fb))
}
ds="test"
fb <- fread(paste0("input/", ds, ".csv"), integer64 = "character")#, showProgress = FALSE)
for(i in seq(span,10,by=span)){
  sapply(seq(span,10,by=span), data_spliter(i,fb))
}
```

## Block Model
The following function process a small block and store the result to csv file.


```R
library(data.table) 
library(dplyr) 
library(ranger) 
library(tidyr) 
block_model <- function(xx){
    function(yy){
        train <- fread(paste0("pdblocks/train_block_", xx, "x", yy, ".csv"), integer64 = "character", showProgress = FALSE)
    test <- fread(paste0("pdblocks/test_block_", xx, "x", yy, ".csv"), integer64 = "character", showProgress = FALSE)

    train$hour = (train$time/60) %% 24
    train$weekday = (train$time/(60*24)) %% 7
    train$month = (train$time/(60*24*30)) %% 12 
    train$year = train$time/(60*24*365)
    train$day = train$time/(60*24) %% 365

    test$hour = (test$time/60) %% 24
    test$weekday = (test$time/(60*24)) %% 7
    test$month = (test$time/(60*24*30)) %% 12 
    test$year = test$time/(60*24*365)
    test$day = test$time/(60*24) %% 365

    train %>% count(place_id) %>% filter(n > 3) -> ids
    small_train = train[train$place_id %in% ids$place_id,]

    small_train$place_id <- as.factor(small_train$place_id) 
    model_rf <- ranger(place_id ~ x + y + accuracy + hour + weekday + month + year,
                       small_train,
                       num.trees = 100,
                       write.forest = TRUE,
                       probability = TRUE,
                       importance = "impurity")


    pred = predict(model_rf, test)

    pred = pred$predictions
    results <- sapply(seq(1:dim(test)[1]), function(x){ names(sort(pred[x,],decreasing=TRUE)[1:3])})
    top3 <- t(results)
    submissions <- data.frame(row_id=test$row_id, p1 = top3[,1], p2 = top3[,2], p3 = top3[, 3] )
    fs <- tidyr::unite(submissions,place_id, 2:4,sep=' ')
    write.csv(fs, file = paste0("pdblocks/result_", xx, "x", yy, ".csv"),quote = FALSE,row.names = FALSE)
    }
    
}
```

## Single process
Apply the block model to all the blocks.


```R
outlog <- function(x){
  cat(x , format(Sys.time(),"%Y-%m-%d %H:%M:%OS3"), "\n")
}
outlog('Generater start at ')
span = 0.2
for(i in seq(span,10,by=span)){
  sapply(seq(span,10,by=span), block_model(i))
}
outlog('Generater end at ')
```

    Generater start at  2016-06-20 13:24:02.008 
    Generater end at  2016-06-20 16:17:32.461 


It takes about 2 hours and 53 minutes without parallel processing.
## Parallel processing
R package foreach can be used to do parallel processing on multicore machine.
We modify the block_model function first to enable the usage in parallel processing. Then use foreach to do parallel processing.


```R
library(data.table) 
library(dplyr) 
library(ranger) 
library(tidyr) 
pblock_model <- function(xx, yy){
    train <- fread(paste0("pdblocks/train_block_", xx, "x", yy, ".csv"), integer64 = "character", showProgress = FALSE)
    test <- fread(paste0("pdblocks/test_block_", xx, "x", yy, ".csv"), integer64 = "character", showProgress = FALSE)

    train$hour = (train$time/60) %% 24
    train$weekday = (train$time/(60*24)) %% 7
    train$month = (train$time/(60*24*30)) %% 12 
    train$year = train$time/(60*24*365)
    train$day = train$time/(60*24) %% 365

    test$hour = (test$time/60) %% 24
    test$weekday = (test$time/(60*24)) %% 7
    test$month = (test$time/(60*24*30)) %% 12 
    test$year = test$time/(60*24*365)
    test$day = test$time/(60*24) %% 365

    train %>% count(place_id) %>% filter(n > 3) -> ids
    small_train = train[train$place_id %in% ids$place_id,]

    small_train$place_id <- as.factor(small_train$place_id) 
    model_rf <- ranger(place_id ~ x + y + accuracy + hour + weekday + month + year,
                       small_train,
                       num.trees = 100,
                       write.forest = TRUE,
                       probability = TRUE,
                       importance = "impurity")


    pred = predict(model_rf, test)

    pred = pred$predictions
    results <- sapply(seq(1:dim(test)[1]), function(x){ names(sort(pred[x,],decreasing=TRUE)[1:3])})
    top3 <- t(results)
    submissions <- data.frame(row_id=test$row_id, p1 = top3[,1], p2 = top3[,2], p3 = top3[, 3] )
    fs <- tidyr::unite(submissions,place_id, 2:4,sep=' ')
    write.csv(fs, file = paste0("pdblocks/presult_", xx, "x", yy, ".csv"),quote = FALSE,row.names = FALSE)
}

library(foreach)
library(doParallel)

span = 0.2
nodes <- detectCores()
registerDoParallel(nodes)
outlog(paste0("Nodes: ", nodes))
outlog("Parallel start at: ")
presults <- foreach(n = seq(span,10,by=span)) %:% foreach(m = seq(span,10,by=span), .packages = c('data.table','ranger','tidyr')) %dopar% { 
  pblock_model(n,m) 
}
outlog("Parallel end at: ")

```

    Loading required package: iterators
    Loading required package: parallel


    Nodes: 8 2016-06-20 16:42:42.212 
    Parallel start at:  2016-06-20 16:42:42.216 
    Parallel end at:  2016-06-20 18:21:30.135 




It takes about 1 hours and 39 minutes with parallel processing on a 8 core machine.

## Generate submission
Merge all the result files to get the submission.


```R
library(data.table) 
library(dplyr) 
library(tidyr) 
fdt <- fread(paste0("pdblocks/result_", 0.2, "x", 0.2, ".csv"), showProgress = FALSE)
span=0.2
for(n in seq(span, 10, by=span)){
    for(m in seq(span, 10, by=span)){
        sdt <- fread(paste0("pdblocks/result_", n, "x", m, ".csv"), showProgress = FALSE)
        fdt <- union(fdt,sdt)            
    }        
}
# remove duplicates introduced by epsilon
fdt %>% distinct(row_id) %>% arrange(row_id) -> offdt
write.csv(offdt, file = "offdt.csv",quote = FALSE,row.names = FALSE)
```

## Summary
In this step we tried the grid method and the parallel processing. Now we are ready to try other algorithm on these blocks.
