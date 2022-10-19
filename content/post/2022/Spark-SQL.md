---
title: "Spark SQL"
description: 
date: 2022-10-14T16:58:12+08:00
hidden: false
comments: true
draft: false

---

**Task 1**

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

**Task 2**

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

**Task 3**

Conduct queries (listed below) over MAS database and DPLP data uniformly using Spark SQL.

> List all the papers (year and title) of “Divesh Srivastava”from DBLP that are published after his last paper in the MAS database

1.Find the last paper in the MAS:

Try:
{{< highlight python >}}
author.createOrReplaceTempView('author')
writes.createOrReplaceTempView('writes')
publication.createOrReplaceTempView('publication')
spark.sql("SELECT year FROM publication WHERE pid IN (SELECT pid FROM writes WHERE aid = (SELECT aid FROM author WHERE name = 'Divesh Srivastava') )ORDER BY year DESC LIMIT 1").show()
{{< /highlight >}}
Error：[WinError 10061] 由于目标计算机积极拒绝，无法连接

Try:
{{< highlight python >}}
author.createOrReplaceTempView('author')
writes.createOrReplaceTempView('writes')
publication.createOrReplaceTempView('publication')
aid=spark.sql("SELECT aid FROM author WHERE name = 'Divesh Srivastava'")
aid=aid.toPandas()
year=spark.sql("SELECT year FROM publication WHERE pid IN (SELECT pid FROM writes WHERE aid = "+str(aid.aid[0])+")ORDER BY year DESC LIMIT 1")
year=year.toPandas()
{{< /highlight >}}

2.Find papers in the DBLP:

Try:
{{< highlight python >}}
dblp=spark.read \
        .format('com.databricks.spark.xml') \
        .option('rootTag', 'dblp') \
        .option('rowTag', 'article') \
        .load('dblp.xml')
{{< /highlight >}}
Too many errors in data...

Try:
{{< highlight python >}}
import xml.etree.ElementTree as ET
import pandas as pd
import codecs

with codecs.open('dblp.xml', 'r', encoding='utf8') as f:
    tt = f.read()

def xml2df(xml_data):
    root = ET.XML(xml_data)
    all_records = []
    for i, child in enumerate(root):
        record = {}
        for sub_child in child:
            record[sub_child.tag] = sub_child.text
        all_records.append(record)
    return pd.DataFrame(all_records)

df_xml1 = xml2df(tt)
{{< /highlight >}}
OverflowError: size does not fit in an int

Try it using Sample.xml:
{{< highlight python >}}
import lxml
from lxml import etree as et
import io
import chardet
import pandas as pd

fn = 'sample.xml'
info=[]
info_list=[]
for event, elem in lxml.etree.iterparse(fn, load_dtd=True):
    if elem.tag not in ['article', 'inproceedings', 'proceedings']:
        continue
    info=[]
    author=[]
    for i in elem:
        if(i.tag=='author'):
            author.append(i.text)
            if(len(info) == 0):
                info.append(author)
            else:
                info[0]=author
        if(i.tag=='title' or i.tag=='year'):
            info.append(i.text)
    info_list.append(info)
    elem.clear()
	
from pandas.core.frame import DataFrame
data=DataFrame(info_list)
data.rename(columns={0:'author',1:'title',2:'year'},inplace=True)

sparkDF=spark.createDataFrame(data) 
sparkDF.printSchema()
sparkDF.show()
{{< /highlight >}}

Py4JJavaError: An error occurred while calling o54.showString.

...??? Seems like something wrong with enviorment. 

Fix that by setting environment variables as follows:
{{< highlight python >}}
PYSPARK_DRIVER_PYTHON=jupyter
PYSPARK_DRIVER_PYTHON_OPTS=notebook
PYSPARK_PYTHON=python
{{< /highlight >}}

It works well on Sample.xml

When it comes to DBLP.xml:
{{< highlight python >}}
sparkDF=spark.createDataFrame(data) 
{{< /highlight >}}
TypeError: field author: Can not merge type <class 'pyspark.sql.types.ArrayType'> and <class 'pyspark.sql.types.StringType'>

Try:
{{< highlight python >}}
import numpy as np
for i in data.columns:
    data[i].iloc[np.where(data[i].isna() == True)] = "Nan values"
from pyspark.sql.types import *
df_schema = StructType([StructField("author", ArrayType(StringType()), True)\
                       ,StructField("title", StringType(), True)\
                       ,StructField("year", StringType(), True)])
#rdd = spark.sparkContext.parallelize(data)
#Pandas dataframes can not direct convert to rdd. 
sparkDF=spark.createDataFrame(rdd,df_schema) 
sparkDF.printSchema()
{{< /highlight >}}

Something wrong with my data... Add an if with loading 'info':
{{< highlight python >}}
if(len(info)==3):
    info_list.append(info)
{{< /highlight >}}

Find the papers：
{{< highlight python >}}
from pyspark.sql import functions as F

df_result = sparkDF.filter(F.array_contains(F.col('author'), 'Divesh Srivastava') & (sparkDF.year>(int(year.year[0])))).show()
{{< /highlight >}}
Supper slow...
Set the PySpark executor memory:
{{< highlight python >}}
import os
memory = '30g'
pyspark_submit_args = ' --driver-memory ' + memory + ' pyspark-shell'
os.environ["PYSPARK_SUBMIT_ARGS"] = pyspark_submit_args
{{< /highlight >}}

> List the author(s) (names) who have collaborated most (in terms of number of publications) with “Divesh Srivastava”other than himself based on MAS database and DBLP data. The result may contain more than one author if they coauthored with him for same number of papers.

{{< highlight python >}}
query = "SELECT name,aid,pub\
         FROM author \
         INNER JOIN\
         (SELECT DISTINCT first(aid) AS aid,COUNT(pid) AS pub\
         FROM writes\
         INNER JOIN\
         (SELECT pid FROM writes WHERE aid = "+str(aid.aid[0])+")AS h\
         USING(pid)\
         GROUP BY aid) AS t USING(aid)\
         WHERE name!='Divesh Srivastava'"
		 
aids=spark.sql(query)
aids=aids.toPandas()
{{< /highlight >}}



{{< highlight python >}}
df= sparkDF.filter(F.array_contains(F.col('author'), 'Divesh Srivastava') & (sparkDF.year>(int(year.year[0]))))

from pyspark.sql import functions as F

a=df.withColumn("atr", F.expr("""transform(array_distinct(author),x->aggregate(author,0,(acc,y)->\
                               IF(y=x, acc+1,acc)))"""))\
  .withColumn("zip", F.explode(F.arrays_zip(F.array_distinct("author"),("atr"))))\
  .select("zip.*").withColumnRenamed("0","elements")\
  .groupBy("elements").agg(F.sum("atr").alias("sum"))\
  .collect()

dict={a[i][0]: a[i][1] for i in range(len(a))} 
df_aids = pd.DataFrame(columns=['name','pub']) 
for i in range(len(a)):
    df_aids.loc[len(df_aids.index)] = [a[i][0],a[i][1]]
{{< /highlight >}}

However there are so many duplicate data in these two databases:
{{< highlight python >}}
s1 = pd.merge(aids, df_aids, how='inner', on=['name'])

     name	          pub_x pub_y
0	Howard J. Karloff	4	1
1	Bojian Xu	1	5
2	Beng Chin Ooi	7	1
3	Albert Angel	2	1
4	Yannis Velegrakis	4	2
5	Vladislav Shkapenyuk	2	3
6	Songtao Guo	1	2
7	Flip Korn	22	8
8	Xin Luna Dong	3	24
9	Theodore Johnson	7	5
10	Gerhard Weikum	2	1
11	Lukasz Golab	7	24
12	Qing Zhang	4	2
13	Vassilis J. Tsotras	5	1
14	Nikos Sarkas	2	1
15	Nick Koudas	76	5
16	Monica Scannapieco	1	1
17	Entong Shen	3	2
18	Graham Cormode	21	19
19	Ninghui Li	1	1
20	Anish Das Sarma	2	2
21	Tamraparni Dasu	1	12
22	Barna Saha	1	13
{{< /highlight >}}

So I changed my method:
{{< highlight python >}}
query = "SELECT name,title\
         FROM publication\
         INNER JOIN\
         (SELECT name,pid\
         FROM author \
         INNER JOIN\
         (SELECT DISTINCT aid,pid\
         FROM writes\
         INNER JOIN\
         (SELECT pid FROM writes WHERE aid = "+str(aid.aid[0])+")AS h\
         USING(pid)\
         ) AS t USING(aid)\
         WHERE name!='Divesh Srivastava') AS r\
         USING (pid)"
aids=spark.sql(query)

df= sparkDF.filter(F.array_contains(F.col('author'), 'Divesh Srivastava'))
from pyspark.sql.functions import explode
df2 = df.select(df.title,explode(df.author))
df2 = df2.selectExpr("col as name","title as title")

from pyspark.sql import SparkSession
import functools
def unionAll(dfs):
    return functools.reduce(lambda df1, df2: df1.union(df2.select(df1.columns)), dfs)
    spark = SparkSession.builder.getOrCreate()
    
unioned_df = unionAll([aids, df2])

unioned_df = unioned_df.distinct()
unioned_df = unioned_df.groupBy("name").count()
unioned_df.show()
unioned_df=unioned_df.filter(unioned_df.name!='Divesh Srivastava')

import pyspark.sql.functions as f
unioned_df.join(unioned_df.agg(f.max('count').alias('count')),on='count',how='leftsemi').show()
{{< /highlight >}}


> List the number of publications of “Divesh Srivastava”each year based on MAS database and DBLP data. Duplicate papers in both MAS and DBLP should be counted only once in the result.

{{< highlight python >}}
df_all = sparkDF.filter(F.array_contains(F.col('author'), 'Divesh Srivastava'))
query_all = "SELECT title,year\
             FROM publication\
             WHERE pid IN\
             (SELECT pid FROM writes WHERE aid = "+str(aid.aid[0])+")"
pub_all=spark.sql(query_all)

df_all=df_all.drop("author")

from pyspark.sql import SparkSession
import functools
def unionAll(dfs):
    return functools.reduce(lambda df1, df2: df1.union(df2.select(df1.columns)), dfs)
    spark = SparkSession.builder.getOrCreate()
    
unioned_df = unionAll([df_all, pub_all])
unioned_df = unioned_df.distinct()

unioned_df = unioned_df.groupBy("year").count()
unioned_df.show()
{{< /highlight >}}

> Find papers published in 2021 that are relevant to keyword query ‘self attention transformer’ (or 'self-attention transformer'). Treat each paper title as one document and rank them using tf-idf. Return the top 10 relevant papers (title, authors, journal/conference and year).  



**Refrence**

https://www.cjavapy.com/article/81/

https://juejin.cn/post/6847902219627397127

https://stackoverflow.com/questions/52968877/read-xml-file-to-pandas-dataframe

https://stackoverflow.com/questions/49614725/entity-ouml-error-while-using-lxml-to-parse-dblp-data

https://stackoverflow.com/questions/56213955/python-worker-failed-to-connect-back-in-pyspark-or-spark-version-2-3-1

https://sparkbyexamples.com/pyspark/pyspark-structtype-and-structfield/

https://stackoverflow.com/questions/74106274/arraytypestringtype-can-not-accept-object-sql-data-system-for-vse-a-relati

https://stackoverflow.com/questions/51601478/setting-pyspark-executor-memory-and-executor-core-within-jupyter-notebook

https://stackoverflow.com/questions/61682833/counting-instaces-of-values-across-lists-within-a-single-column

https://sparkbyexamples.com/pyspark/pyspark-explode-array-and-map-columns-to-rows/

https://www.geeksforgeeks.org/merge-two-dataframes-in-pyspark/

https://sparkbyexamples.com/pyspark/pyspark-distinct-to-drop-duplicates/

https://sparkbyexamples.com/pyspark/pyspark-groupby-count-explained/
