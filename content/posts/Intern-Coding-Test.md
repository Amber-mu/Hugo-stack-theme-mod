---
title: "Intern Coding Test"
description: 
date: 2022-10-14T12:01:27+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: true

---

This is a coding test for intern screening by the AIDF NLP-AD Team.

Task 1: Write a web crawler to crawl **all** news from [www.forbes.com/business](https://ddec1-0-en-ctp.trendmicro.com/wis/clicktime/v1/query?url=http%3a%2f%2fwww.forbes.com%2fbusiness&umid=40f11d39-a1fd-4d28-a9a3-79e4a7ad1bb1&auth=8d3ccd473d52f326e51c0f75cb32c9541898e5d5-1554c4bac75a019c3d69449e94ae1eaa026ff0ea). Save each piece of the article as one txt document.

​       Please note that by “all news”, it means that you are supposed to crawl those after you click the button “More Articles” as well. To save time, you may set the click number **30** times.





 Task 2: Choose 1 article from step 1 output, which talks about any company. Write codes to do the following pre-processing steps:

1   Remove all company names shown in the article (Please do **not** use hardcode to specify the company name)

~~~markdown
    ```python

    ```
~~~

2    Regular Expression/Normalization — lowercase the words, remove punctuation and remove numbers

~~~markdown
    ```python
import re

def Normalization(text):
    text = text.lower()  #lowercase
    punctuation = r"~!@#$%^&*()_+`{}|\[\]\:\";\-\\\='<>?,./，。、《》？；：‘“{【】}|、！@#￥%……&*（）——+=-"
    text = re.sub(r'[{}]+'.format(punctuation), '', text)
    text = re.sub(r'[0-9]+', '', text)
    return text
    
    ```
~~~

3    Tokenization

~~~markdown
    ```python
import nltk

def Tokenization(text):
    text = nltk.word_tokenize(text)
    return text
    ```
~~~

4    Remove stop words 

~~~markdown
    ```python
from nltk.corpus import stopwords

def Stopword_Remove(text):
    stop_words = set(stopwords.words("english"))  
    text = [word for word in text if word not in stop_words]   # removing stop words
    return text    
    ```
~~~

5    Lemmatization

~~~markdown
    ```python
from nltk.stem import WordNetLemmatizer
from nltk import pos_tag
from nltk import RegexpParser
from nltk.corpus import wordnet
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
        return ''

def get_tags(text):
    tags = pos_tag(text)
    return get_wordnet_pos(tags[0][1])

def Lemmatization(text):
    lemmatiser = WordNetLemmatizer() 
    text = [lemmatiser.lemmatize(t, get_tags([t])) for t in text]
    return text
    ```
~~~

6    Any other pre-processing steps you think are necessary to prepare this input for topic modelling.





Reference

https://regex101.com/

https://ithelp.ithome.com.tw/m/articles/10262664

https://stackoverflow.com/questions/15586721/wordnet-lemmatization-and-pos-tagging-in-python





