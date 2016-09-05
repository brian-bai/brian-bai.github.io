---
layout: post
title: "Pyspark in Jupyter Notebook Setup"
date: 2016-09-04 21:00:00
category: Spark
tags: Spark

---

## Overview
Spark is hot now. Though Spark is a big data technology, it can be used local with small spec requirement. The notebook is an excellent tool to do exploratory analysis. These two can be setup together.

## Env
This is the memo of my own setup. Jupyter and Python 3.5 already installed.
Jupyter 4.1.0
Python 3.5

## Spark download
Spark can be downloaded from the [Apache Spark download page](https://spark.apache.org/downloads.html). I have downloaded the latest 2.0 version.

```
$wget http://d3kbcqa49mib13.cloudfront.net/spark-2.0.0-bin-hadoop2.7.tgz
```

## Spark Setup

```
>tar -xvf spark-2.0.0-bin-hadoop2.7.tgz
>cd spark-2.0.0-bin-hadoop2.7/
```

## Test pyspark

```
>.bin/spark-submit examples/src/main/python/pi.py 10

```

## Spark Juypter notebook setup

```
>vi ~/.bashrc

export SPARK_HOME="/path/to/spark-2.0.0-bin-hadoop2.7"
export PATH=$SPARK_HOME/bin:$PATH
export PATH=$SPARK_HOME/sbin:$PATH
alias sparknb='PYSPARK_PYTHON=/path/to/python PYSPARK_DRIVER_PYTHON=jupyter PYSPARK_DRIVER_PYTHON_OPTS="notebook" pyspark'    

>source ~/.bashrc
```
## Notebook start

```
>sparknb
```

## Notebook test



```python
spark.version
```




    '2.0.0'




```python
from pyspark.sql.types import StructType
from pyspark.sql.types import StructField
from pyspark.sql.types import IntegerType
from pyspark.sql.types import StringType
from pyspark.sql.types import DoubleType

struct = StructType([StructField("PassengerId", IntegerType(),True),
                    StructField("Survived", StringType(),True),
                    StructField("Pclass", StringType(),True),
                    StructField("Name", StringType(),True),
                    StructField("Gender", StringType(),True),
                    StructField("Age", DoubleType(),True),
                    StructField("SibSp", IntegerType(),True),
                    StructField("Parch", IntegerType(),True),
                    StructField("Ticket", StringType(),True),
                    StructField("Fare", DoubleType(),True),
                    StructField("Cabin", StringType(),True),
                    StructField("Embarked", StringType(),True)])
```


```python
raw_train = spark.read.csv('data/titanic/train.csv',header=True, schema=struct)
```


```python
raw_train.show()
```

    +-----------+--------+------+--------------------+------+----+-----+-----+----------------+-------+-----+--------+
    |PassengerId|Survived|Pclass|                Name|Gender| Age|SibSp|Parch|          Ticket|   Fare|Cabin|Embarked|
    +-----------+--------+------+--------------------+------+----+-----+-----+----------------+-------+-----+--------+
    |          1|       0|     3|Braund, Mr. Owen ...|  male|22.0|    1|    0|       A/5 21171|   7.25|     |       S|
    |          2|       1|     1|Cumings, Mrs. Joh...|female|38.0|    1|    0|        PC 17599|71.2833|  C85|       C|
    |          3|       1|     3|Heikkinen, Miss. ...|female|26.0|    0|    0|STON/O2. 3101282|  7.925|     |       S|
    |          4|       1|     1|Futrelle, Mrs. Ja...|female|35.0|    1|    0|          113803|   53.1| C123|       S|
    |          5|       0|     3|Allen, Mr. Willia...|  male|35.0|    0|    0|          373450|   8.05|     |       S|
    |          6|       0|     3|    Moran, Mr. James|  male|null|    0|    0|          330877| 8.4583|     |       Q|
    |          7|       0|     1|McCarthy, Mr. Tim...|  male|54.0|    0|    0|           17463|51.8625|  E46|       S|
    |          8|       0|     3|Palsson, Master. ...|  male| 2.0|    3|    1|          349909| 21.075|     |       S|
    |          9|       1|     3|Johnson, Mrs. Osc...|female|27.0|    0|    2|          347742|11.1333|     |       S|
    |         10|       1|     2|Nasser, Mrs. Nich...|female|14.0|    1|    0|          237736|30.0708|     |       C|
    |         11|       1|     3|Sandstrom, Miss. ...|female| 4.0|    1|    1|         PP 9549|   16.7|   G6|       S|
    |         12|       1|     1|Bonnell, Miss. El...|female|58.0|    0|    0|          113783|  26.55| C103|       S|
    |         13|       0|     3|Saundercock, Mr. ...|  male|20.0|    0|    0|       A/5. 2151|   8.05|     |       S|
    |         14|       0|     3|Andersson, Mr. An...|  male|39.0|    1|    5|          347082| 31.275|     |       S|
    |         15|       0|     3|Vestrom, Miss. Hu...|female|14.0|    0|    0|          350406| 7.8542|     |       S|
    |         16|       1|     2|Hewlett, Mrs. (Ma...|female|55.0|    0|    0|          248706|   16.0|     |       S|
    |         17|       0|     3|Rice, Master. Eugene|  male| 2.0|    4|    1|          382652| 29.125|     |       Q|
    |         18|       1|     2|Williams, Mr. Cha...|  male|null|    0|    0|          244373|   13.0|     |       S|
    |         19|       0|     3|Vander Planke, Mr...|female|31.0|    1|    0|          345763|   18.0|     |       S|
    |         20|       1|     3|Masselmani, Mrs. ...|female|null|    0|    0|            2649|  7.225|     |       C|
    +-----------+--------+------+--------------------+------+----+-----+-----+----------------+-------+-----+--------+
    only showing top 20 rows
    



```python
del raw_train
```

Now we can start to use Spark on Jupyter notebook.


