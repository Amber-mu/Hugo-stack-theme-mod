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


Try:
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
Schema：
root
 |-- author: array (nullable = true)
 |    |-- element: string (containsNull = true)
 |-- title: string (nullable = true)
 |-- year: string (nullable = true)
 
Py4JJavaError: An error occurred while calling o54.showString.
...??? Seems like something wrong with enviorment. 
Fix that by setting environment variables as follows:
{{< /highlight >}}
PYSPARK_DRIVER_PYTHON=jupyter
PYSPARK_DRIVER_PYTHON_OPTS=notebook
PYSPARK_PYTHON=python
{{< /highlight >}}

**Refrence**

https://www.cjavapy.com/article/81/

https://juejin.cn/post/6847902219627397127

https://stackoverflow.com/questions/52968877/read-xml-file-to-pandas-dataframe

https://stackoverflow.com/questions/49614725/entity-ouml-error-while-using-lxml-to-parse-dblp-data

https://stackoverflow.com/questions/56213955/python-worker-failed-to-connect-back-in-pyspark-or-spark-version-2-3-1