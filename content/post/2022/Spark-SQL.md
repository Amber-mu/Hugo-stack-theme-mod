---
title: "Spark SQL"
description: 
date: 2022-10-14T16:58:12+08:00
hidden: false
comments: true
draft: false

---

**Tasks 1**

Read  database from MySQL into Spark via MySQL connectors (e.g., JDBC or mysql-connector-python)



1.Install pyspark

Check python version: 

{{< highlight python >}}
import sys
sys.version
{{< /highlight >}}


Return: '3.9.13 (main, Aug 25 2022, 23:51:50) [MSC v.1916 64 bit (AMD64)]'

Install pyspark 3.3.0:

{{< highlight python >}}
pip install pyspark==3.3.0
{{< /highlight >}}


Install Java11: https://www.oracle.com/java/technologies/downloads/#java11

Try:

{{< highlight python >}}
from datetime import datetime, date
import pandas as pd
from pyspark.sql import Row

df = spark.createDataFrame([
    Row(a=1, b=2., c='string1', d=date(2000, 1, 1), e=datetime(2000, 1, 1, 12, 0)),
    Row(a=2, b=3., c='string2', d=date(2000, 2, 1), e=datetime(2000, 1, 2, 12, 0)),
    Row(a=4, b=5., c='string3', d=date(2000, 3, 1), e=datetime(2000, 1, 3, 12, 0))
])
df.show(5)
{{< /highlight >}}

2.Connect to SQL

Find spark home:

{{< highlight python >}}
import findspark
findspark.init()
findspark.find()
{{< /highlight >}}


Download jar( Platform Independent):https://dev.mysql.com/downloads/connector/j/

Move mysql-connector.jar to $SPARK_HOME/jars

Test:

{{< highlight python >}}
from pyspark import SparkContext,SparkConf
from pyspark.sql import SparkSession

spark = SparkSession. \
    Builder(). \
    appName('sql'). \
    master('local'). \
    getOrCreate()

prop = {'user':'username',
       'password':'password',
       'driver':'com.mysql.jdbc.Driver'}
url = 'jdbc:mysql://127.0.0.1:3306/database'
data = spark.read.jdbc(url=url,table='tablename',properties=prop)

data.createOrReplaceTempView('tablename')
spark.sql("select * \
from tablename \
where conditions \
            ").show()
{{< /highlight >}}

**Tasks 2**

Read data (xml file) into Spark

Download jar: spark-xml_version.jar
Move spark-xml_version.jar to $SPARK_HOME/jars

TEST:

{{< highlight python >}}
from pyspark import SparkContext, SparkConf
from pyspark.sql import SQLContext

df_test=spark.read \
        .format('com.databricks.spark.xml') \
        .option('rootTag', 'RootTagName') \
        .option('rowTag', 'RowTagName') \
        .load('sample.xml')
{{< /highlight >}}

**Tasks 3**

Conduct queries (listed below) over MAS database and DPLP data uniformly using Spark SQL.

> List all the papers (year and title) of “Divesh Srivastava”from DBLP that are published after his last paper in the MAS database






Refrence

https://www.cjavapy.com/article/81/

https://juejin.cn/post/6847902219627397127