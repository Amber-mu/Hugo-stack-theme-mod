---
title: "Intern Coding Test"
description: 
date: 2022-10-14T12:01:27+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: false

---

This is a coding test for intern screening by the AIDF NLP-AD Team.

Task 1: Write a web crawler to crawl **all** news from [www.forbes.com/business](https://ddec1-0-en-ctp.trendmicro.com/wis/clicktime/v1/query?url=http%3a%2f%2fwww.forbes.com%2fbusiness&umid=40f11d39-a1fd-4d28-a9a3-79e4a7ad1bb1&auth=8d3ccd473d52f326e51c0f75cb32c9541898e5d5-1554c4bac75a019c3d69449e94ae1eaa026ff0ea). Save each piece of the article as one txt document.

​       Please note that by “all news”, it means that you are supposed to crawl those after you click the button “More Articles” as well. To save time, you may set the click number **30** times.

{{< highlight python >}}
import requests
import parsel
import time
from lxml import etree
import scrapy
from bs4 import BeautifulSoup
import urllib.request
import re
import random

import sys
import io

import string

from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By

import pandas as pd
import numpy as np
from requests.compat import urljoin
{{< /highlight >}}

Get one article:
{{< highlight python >}}
def getUrlText(url):

    headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:92.0) Gecko/20100101 Firefox/92.0',
        }

    html_text = requests.get(url,headers = headers)


    selector = parsel.Selector(html_text.text)
    html_title = selector.css('.fs-headline')

    title_text = html_title.xpath('.//text()').get()

    result = ''.join(random.sample(string.digits, 8))
    file_name = result
    file_name = str(file_name) + ".txt"

    #create file and write
    try:
        with open(r'{}'.format(file_name), 'a+') as f:
            #f.write(str(url) + '\n')
            f.write(str(title_text) + '\n')
            print(file_name+" write title successfully!")
            f.close()
    except:
        pass
    body_p_texts = selector.css('p')
    for body_p_text in body_p_texts:
        text = body_p_text.xpath('.//text()').getall()

        text = list(map(str,text))
        text=' '.join(text)

        try:
            with open(r'{}'.format(file_name), 'a+',encoding='utf-8') as f:
                f.write(text + '\n')
                f.close()
        except:
            pass
{{< /highlight >}}

Get one page of articles:
{{< highlight python >}}
def getUrl(date,start,count):

    headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:92.0) Gecko/20100101 Firefox/92.0',
        }

    params = {
        'date': date,
        'start': start,
        'ids': 'content_63498a475fc47600011afb73',
        'limit': '25',
        'sourceValue': 'channel_1',
        'swimLane': '',
        'specialSlot': '',
        'streamSourceType': 'channelsection'
        }

    url = "https://www.forbes.com/simple-data/chansec/stream/"
    html_text = requests.get(url,headers = headers,params = params)
    for i in range(0,10):
        url_base = html_text.json()["blocks"]["items"][i]["url"]
        getUrlText(url_base)
        print(url_base)
    print(f"{count} executed")
{{< /highlight >}}


{{< highlight python >}}
def get_Date():
    list_dates = ["1665780202342","1665773443009","1665769827727","1665764665783","1665759600000","1665756506107","1665781780480","1665774032000","1665770879938","1665765682411","1665759636000","1665757059227","1665754565662","1665749267965","1665738000000","1665709889708","1665699033834","1665691608142","1665687624000","1665682799169","1665677281428","1665675322663","1665670049585","1665666580107","1665662469015","1665659495182","1665650031439","1665616682929","1665609153975","1665608220044","1665603038606"]
    j = 0
    for date_l in list_dates:
        j = j+1
        count = j
        getUrl(date_l,16,count)
{{< /highlight >}}

 Task 2: Choose 1 article from step 1 output, which talks about any company. Write codes to do the following pre-processing steps:

{{< highlight python >}}
import re
import nltk
from nltk.corpus import stopwords
from nltk.stem import WordNetLemmatizer
from nltk import pos_tag
from nltk import RegexpParser
from nltk.corpus import wordnet
import spacy
{{< /highlight >}}

1   Remove all company names shown in the article (Please do **not** use hardcode to specify the company name)

{{< highlight python >}}
nlp = spacy.load('en_core_web_sm')
def ORG_remove(text):
    document=nlp(text)
    text_no_namedentities = []
    org_stopwords = [ent.text for ent in document.ents if ent.label_ == 'ORG']
    print(org_stopwords)
    regex = re.compile('|'.join(org_stopwords))
    text = re.sub(regex, '', text) 
    return text
{{< /highlight >}}

2    Regular Expression/Normalization — lowercase the words, remove punctuation and remove numbers

{{< highlight python >}}

def Normalization(text):
    text = text.lower()  #lowercase
    punctuation = r"~!@#$%^&*()_+`{}|\[\]\:\";\-\\\='<>?,./，。、《》？；：‘“{【】}|、！@#￥%……&*（）——+=-"
    text = re.sub(r'[{}]+'.format(punctuation), '', text)
    text = re.sub(r'[0-9]+', '', text)
    return text
{{< /highlight >}}

3    Tokenization

{{< highlight python >}}
import nltk

def Tokenization(text):
    text = nltk.word_tokenize(text)
    return text
{{< /highlight >}}

4    Remove stop words 

{{< highlight python >}}
from nltk.corpus import stopwords

def Stopword_Remove(text):
    stop_words = set(stopwords.words("english"))  
    text = [word for word in text if word not in stop_words]   # removing stop words
    return text    
{{< /highlight >}}

5    Lemmatization

{{< highlight python >}}

def get_wordnet_pos(treebank_tag):

    if treebank_tag.startswith('J'):
        return wordnet.ADJ
    elif treebank_tag.startswith('V'):
        return wordnet.VERB
    elif treebank_tag.startswith('N'):
        return wordnet.NOUN
    elif treebank_tag.startswith('R'):
        return wordnet.ADV
    else:
        return wordnet.NOUN #return None or '' not working

def get_tags(text):
    tags = pos_tag(text)
    return get_wordnet_pos(tags[0][1])

def Lemmatization(text):
    lemmatiser = WordNetLemmatizer() 
    text = [lemmatiser.lemmatize(t, get_tags([t])) for t in text]
    return text
{{< /highlight >}}

6    Any other pre-processing steps you think are necessary to prepare this input for topic modelling.





**Refrence**

https://regex101.com/

https://ithelp.ithome.com.tw/m/articles/10262664

https://stackoverflow.com/questions/15586721/wordnet-lemmatization-and-pos-tagging-in-python





