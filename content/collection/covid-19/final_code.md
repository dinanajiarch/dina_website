---
title: "Big Data Analytics Course Project"
date: 2020-12-11
author: 'Dina Arch'
subtitle: A collaborative project for Big Data Analytics at UC Santa Barbara using COVID-19 data.
excerpt: This project used Natural Language Processing (NLP) techniques to analyze the text from the abstracts within the CORD-19 dataset and used Machine Learning methods to train and fit models. Specifically, Latent Dirichlet Allocation was used to see what topics are described in the abstracts of these papers. Logistic Regression and Naïve Bayes were used in conjunction with some NLP and machine learning techniques to predict Journal based on the text from the abstracts.
weight: 4
draft: false
images: 
series: 
tags:
- python
- machine learning
- visualization
- classification
- spark
categories:
- python
editor_options:
  markdown:
    wrap: sentence
links:
- icon: github
  icon_pack: fab
  name: code and dataset
  url: https://github.com/dinanajiarch/covid19_project   
---



# Final Project: Big Data Analytics

## Dina Arch, Jeremy Knox, Theo Velikov, Roupen Khanjian

### Data Import


```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.getOrCreate()
```

[CORD-19: The COVID-19 Open Research Dataset](https://arxiv.org/abs/2004.10706)


```python
#Import COVID-19 dataset
filename = '/home/jovyan/Project/metadata.csv'
df = spark.read.csv(filename,  inferSchema=True, header = True) 
```


```python
df.printSchema()
```

    root
     |-- cord_uid: string (nullable = true)
     |-- sha: string (nullable = true)
     |-- source_x: string (nullable = true)
     |-- title: string (nullable = true)
     |-- doi: string (nullable = true)
     |-- pmcid: string (nullable = true)
     |-- pubmed_id: string (nullable = true)
     |-- license: string (nullable = true)
     |-- abstract: string (nullable = true)
     |-- publish_time: string (nullable = true)
     |-- authors: string (nullable = true)
     |-- journal: string (nullable = true)
     |-- mag_id: string (nullable = true)
     |-- who_covidence_id: string (nullable = true)
     |-- arxiv_id: string (nullable = true)
     |-- pdf_json_files: string (nullable = true)
     |-- pmc_json_files: string (nullable = true)
     |-- url: string (nullable = true)
     |-- s2_id: string (nullable = true)
    
    

Source: https://github.com/allenai/cord19


```python
df.show(1)
```

    +--------+--------------------+--------+--------------------+--------------------+--------+---------+-------+--------------------+------------+--------------------+--------------+------+----------------+--------+--------------------+--------------------+--------------------+-----+
    |cord_uid|                 sha|source_x|               title|                 doi|   pmcid|pubmed_id|license|            abstract|publish_time|             authors|       journal|mag_id|who_covidence_id|arxiv_id|      pdf_json_files|      pmc_json_files|                 url|s2_id|
    +--------+--------------------+--------+--------------------+--------------------+--------+---------+-------+--------------------+------------+--------------------+--------------+------+----------------+--------+--------------------+--------------------+--------------------+-----+
    |ug7v899j|d1aafb70c066a2068...|     PMC|Clinical features...|10.1186/1471-2334...|PMC35282| 11472636|  no-cc|OBJECTIVE: This r...|  2001-07-04|Madani, Tariq A; ...|BMC Infect Dis|  null|            null|    null|document_parses/p...|document_parses/p...|https://www.ncbi....| null|
    +--------+--------------------+--------+--------------------+--------------------+--------+---------+-------+--------------------+------------+--------------------+--------------+------+----------------+--------+--------------------+--------------------+--------------------+-----+
    only showing top 1 row
    
    

### Data Cleaning

#### Duplicates


```python
#Check for duplicates
```


```python
print('Count of rows: {0}'.format(df.count()))
print('Count of distinct rows:{0}'.format(df.distinct().count()))
```

    Count of rows: 136067
    Count of distinct rows:136064
    


```python
#Check for duplicates in the data irrespective of ID
```


```python
print('Count of ids: {0}'.format(df.count()))
print('Count of distinct ids: {0}'.format(
df.select([
c for c in df.columns if c != 'cord_uid'
]).distinct().count())
)
```

    Count of ids: 136067
    Count of distinct ids: 136053
    


```python
# Check for duplicated ID
```


```python
import pyspark.sql.functions as fn
df.agg(
fn.count('cord_uid').alias('count'),
fn.countDistinct('cord_uid').alias('distinct')
).show()
```

    +------+--------+
    | count|distinct|
    +------+--------+
    |136067|  135124|
    +------+--------+
    
    

Source: https://github.com/allenai/cord19/blob/master/README.md

##### Why can the same cord_uid appear in multiple rows?
This is a very tricky issue, and we have not decided on the best way forward. To explain, let’s take example cord_uid=hox2xwjg. Examining their respective rows in the metadata file, we see that they are the same paper, but sent from different sources (Elsevier, PMC). The Elsevier row has DOI and PDF, but the PMC row doesn’t. Furthermore, the PMC ID, publication date, and URL for each of these rows is different.

Technically all of this data is representative of paper hox2xwjg so we don’t want to remove any of it. But combining them into one cluster would require a schema change to the data, which would break a lot of people’s code. Hopefully this is not too big an issue because there are only a small percentage of papers affected, but know that this issue exists and we’re debating what’s the best way forward.

#### Missing


```python
# Percent missing for each column
```


```python
import pyspark.sql.functions as fn
df.agg(*[
(1 - (fn.count(c) / fn.count('*'))).alias(c + '_missing')
for c in df.columns
]).show()
```

    +----------------+-----------------+----------------+--------------------+-------------------+------------------+------------------+--------------------+-------------------+--------------------+-------------------+-------------------+------------------+------------------------+------------------+----------------------+----------------------+-------------------+-------------------+
    |cord_uid_missing|      sha_missing|source_x_missing|       title_missing|        doi_missing|     pmcid_missing| pubmed_id_missing|     license_missing|   abstract_missing|publish_time_missing|    authors_missing|    journal_missing|    mag_id_missing|who_covidence_id_missing|  arxiv_id_missing|pdf_json_files_missing|pmc_json_files_missing|        url_missing|      s2_id_missing|
    +----------------+-----------------+----------------+--------------------+-------------------+------------------+------------------+--------------------+-------------------+--------------------+-------------------+-------------------+------------------+------------------------+------------------+----------------------+----------------------+-------------------+-------------------+
    |             0.0|0.567859951347498|             0.0|2.204796166593858...|0.21650363423901464|0.5229703013956359|0.2409842210087677|3.013221427679013...|0.21533509227071956|3.233701044338399...|0.03670985617379674|0.04888033101339784|0.9796644300234443|        0.83416993098988|0.9731749799731015|    0.5558732095217798|     0.643256630924471|0.08417176832001882|0.20808866220317934|
    +----------------+-----------------+----------------+--------------------+-------------------+------------------+------------------+--------------------+-------------------+--------------------+-------------------+-------------------+------------------+------------------------+------------------+----------------------+----------------------+-------------------+-------------------+
    
    

#### Remove Missing


```python
#Remove null values from abstract, journal, authors
from pyspark.sql.functions import col 
df_noNull = df.filter(col("abstract").isNotNull()).filter(col("journal").isNotNull()).filter(col("authors").isNotNull())
df_noNull.show(2)
```

    +--------+--------------------+--------+--------------------+--------------------+--------+---------+-------+--------------------+------------+--------------------+--------------+------+----------------+--------+--------------------+--------------------+--------------------+-----+
    |cord_uid|                 sha|source_x|               title|                 doi|   pmcid|pubmed_id|license|            abstract|publish_time|             authors|       journal|mag_id|who_covidence_id|arxiv_id|      pdf_json_files|      pmc_json_files|                 url|s2_id|
    +--------+--------------------+--------+--------------------+--------------------+--------+---------+-------+--------------------+------------+--------------------+--------------+------+----------------+--------+--------------------+--------------------+--------------------+-----+
    |ug7v899j|d1aafb70c066a2068...|     PMC|Clinical features...|10.1186/1471-2334...|PMC35282| 11472636|  no-cc|OBJECTIVE: This r...|  2001-07-04|Madani, Tariq A; ...|BMC Infect Dis|  null|            null|    null|document_parses/p...|document_parses/p...|https://www.ncbi....| null|
    |02tnwd4m|6b0567729c2143a66...|     PMC|Nitric oxide: a p...|        10.1186/rr14|PMC59543| 11667967|  no-cc|Inflammatory dise...|  2000-08-15|Vliet, Albert van...|    Respir Res|  null|            null|    null|document_parses/p...|document_parses/p...|https://www.ncbi....| null|
    +--------+--------------------+--------+--------------------+--------------------+--------+---------+-------+--------------------+------------+--------------------+--------------+------+----------------+--------+--------------------+--------------------+--------------------+-----+
    only showing top 2 rows
    
    

Imputation methods and outlier removals (at this time in the analyses) are not applicable as this is qualitative data

### Exploratory Data

#### Top Journal Pulications


```python
# Top Journals

from pyspark.sql.functions import desc
df_noNull.groupby('journal').count().orderBy(desc('count')).show()
```

    +--------------------+-----+
    |             journal|count|
    +--------------------+-----+
    |            PLoS One| 1682|
    | Journal of virology| 1542|
    |            Virology| 1273|
    |             bioRxiv| 1202|
    |  Surgical endoscopy|  823|
    |             Sci Rep|  758|
    |             Viruses|  678|
    |          Arch Virol|  633|
    |    Emerg Infect Dis|  604|
    |The Journal of ge...|  574|
    |             Vaccine|  517|
    |         J Med Virol|  404|
    |             Virol J|  398|
    |      Virus Research|  379|
    |      BMC Infect Dis|  377|
    |Veterinary Microb...|  362|
    |         PLoS Pathog|  348|
    |Emerging infectio...|  329|
    |Journal of clinic...|  324|
    |              Nature|  287|
    +--------------------+-----+
    only showing top 20 rows
    
    

#### Top Authors


```python
### Authors

from pyspark.sql.functions import desc
df_noNull.groupby('authors').count().orderBy(desc('count')).show()
```

    +--------------------+-----+
    |             authors|count|
    +--------------------+-----+
    |                2020|   68|
    |   Mahase, Elisabeth|   30|
    |Andersen, Bjørg M...|   20|
    |   Iacobucci, Gareth|   20|
    |Modrow, Susanne; ...|   19|
    |         Rimmer, Abi|   18|
    |       Knopf, Alison|   17|
    |    Domingo, Esteban|   17|
    |      Pearson, Helen|   17|
    |                2017|   14|
    |    Le Page, Michael|   13|
    |                2005|   13|
    |    Cyranoski, David|   12|
    |                2016|   12|
    |Burrell, Christop...|   12|
    |                2015|   12|
    |FENNER, FRANK; BA...|   12|
    |         Fong, I. W.|   11|
    |Tulchinsky, Theod...|   11|
    |                2018|   11|
    +--------------------+-----+
    only showing top 20 rows
    
    

#### Number of Papers in Top 10 Journals Per Month and Year


```python
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np
%matplotlib inline
np.random.seed(42)
import pyspark.sql.functions as F
from pyspark.sql.functions import col, lower
```


```python
df_2 = spark.read.format('csv')\
          .option('header','true')\
          .option('inferSchema', 'true')\
          .option('timestamp', 'true')\
          .load('metadata.csv')

data = df_2.select(lower(col('journal')),lower(col('abstract')), lower(col('publish_time')))\
        .withColumnRenamed('lower(journal)','Journal')\
        .withColumnRenamed('lower(publish_time)','Publish_Time')\
        .withColumnRenamed('lower(abstract)', 'Abstract')
data.cache()
data_clean = data.na.drop()

top_10 = ["plos one",
"journal of virology",
"virology",
"biorxiv",
"surgical endoscopy",
"sci rep",
"viruses",
"arch virol",
"emerg infect dis",
"virus research"]

data_top10 = data_clean.filter(F.col("Journal").isin(top_10))
```


```python
df_pd = data_top10.toPandas()
```


```python
import pandas as pd
import datetime
df_pd['Publish_Time']= pd.to_datetime(df_pd['Publish_Time'])
```


```python
df_pd['month'] = pd.DatetimeIndex(df_pd['Publish_Time']).month
df_pd['year'] = pd.DatetimeIndex(df_pd['Publish_Time']).year
df_2020 = df_pd[(df_pd.year == 2020)]
```

### Latent Dirichlet Allocation model

#### Evaluation using filtered dataset


```python
from pyspark.sql.functions import rand
#Filtered dataset with n=1000
df_demo =df_noNull.orderBy(rand()).limit(1000)
```

#### Model Construction

Stop word list copied from https://www.jmir.org/2020/11/e21559#app1 to supplement StopWordRemover's dictionary and account for some of the missed out on some stop words


```python
stopwordList = ['shall', 'accordance', 'yourself', 'successfully', 'each', 'showed', 'ok', 'eighty', 'welcome', 'their', ' those', 'been',
'whoever', 'ask', 'mean', 'thank', 'apparently', ' almost', 'cov', 'obtain', 'theyd', 'becomes', 'largely', 'date', 'except', 'n',
'herself', 'be', 'out', 'usefulness', 'itself', 'none', 'for', 'plus', 'went', 'coronavirus', 'your', 'studies', 'somewhere', 'make',
'give', 'means', 'etc', 'look', 'hereafter', 'nine', ' would', 'an', 'first', 'selves', 'vol', 'kept', 'im', 'yes', ' due', "shouldn't",
'awfully', 'mainly', 'therein', 'affects', 'patient', 'section', 'whereas', 'his', ' make', 'other', ' do', 'unto', 'probably',
"doesn't", ' ml', 'together', 'fifth', 'some', 'when', 'anywhere', 'beginning', 'werent', ' therefore', 'value', ' that',
'previously', 'rather', 'ord', 'might', ' mg', 'due', ' what', 'home', 'he', 'found', 'after', 'its', 'et-al', 'na', 'ran', 'run', 'similar',
'readily', 'rd', 'over', 'whomever', ' several', 'a', 'thats', 'moreover', 'mg', 'myself', 'least', 'miss', 'r', 'thereupon', 'care',
'begins', 'sup', ' used', 'during', ' show', ' really', 'last', 'd', 'able', 'outside', 'fix', 'results', 'that', 'if', 'thereto', 'covid',
'behind', 'by', ' mainly', 'h', 'certainly', ' they', 'several', 'causes', 'mrs', 'qv', 'noone', 'say', 'shed', ' pmid', 'kg', 'following',
'different', 'thence', ' using', 'four', 'amongst', "it'll", 'otherwise', 'y', 'did', 'hereupon', 'themselves', 'nd', ' done', 'into', 
'its', 'ex', 'himself', 'zero', 'see', 'ah', "you've", 'and', 'next', 'thanx', 'both', 'anybody', 'would', 'anyways', 'too', 'age', 'still',
'mug', 'patients', 'less', 'actually', 'everyone', 'regards', "they'll", 'name', 'oh', 'until', 'usually', 'afterwards', 'hid', 'one',
'more', 'specify', 'beyond', 'shows', 'whole', 'omitted', 'known', 'try', 'especially', 'itd', 'suggest', ' must', 'km', 'study', 'z',
' theirs', 'already', 'between', 'to', 'ie', 'yourselves', 'quite', 'thus', 'gone', 'think', 'whatever', 'you', 'up', ' seen', 'm', ' any',
'th', ' always', 'irr', ' etc', 'anyone', 'invention', 'past', 'suret', 'right', 'we', 'were', 'related', ' but', ' there', 'gotten', ' our',
"there'll", 'sufficiently', 'various', 'obviously', 'ff', 'former', 'poorly', ' further', ' been', 'useful', 'besides', 'said', 'seemed',
' it', "don't", 'pp', "you'll", 'others', 'makes', 'hundred', 'because', 'nearly', "haven't", 'l', 'viz', 'lets', 'seeming', 'present',
' overall', 'important', 'somewhat', ' most', 'whod', 'slightly', 'ups', 'nobody', 'possibly', 'significant', 'at', 'brief',
'somehow', 'while', 'o', 'ed', 'whether', 'can', 'was', 'what', 'abst', 'everybody', 'ci', 'with', 'resulting', 're', 'cause', 'hes',
'quickly', 'unlike', 'namely', 'contain', "hasn't", 'should', 'ending', 'just', 'doing', 'mr', 'where', 'part', 'everywhere',
'corona', 'my', ' neither', 'much', 'obtained', ' we', 'sorry', ' another', 'use', 'coronaviruse', 'ninety', ' again', 'heres', 'per',
'theres', 'herein', ' during', 'become', 'seem', 'they', 'now', "that've", 'words', 'on', 'particular', 'are', 'away', 'such', 'but',
'indeed', ' nor', 'whereafter', 'ought', 'possible', 'wont', 'believe', 'elsewhere', 'here', 'which', 'trying', 'hardly', 'once',
'couldnt', ' their', 'similarly', 'particularly', 'nevertheless', "there've", 'comes', 'even', 'immediate', 'affecting', 'non', 
'when', ' without', 'cannot','c-bound', 'else', 'seen', 'then', 'throug', 'nor', 'under', 'although', 'or', 'stop', 'thanks', 'must', 'enough',
'old', 'sec', 'taking', 'having', 'she', 'us', 'maybe', 'co', 'importance', 'upon', 'against', 'further', 'neither', 'nos', 'nothing',
'not', 'in', 'little', "i'll", 'whither','rso2', 'thou', 'has', 'adj','f8', '2b8', 'either', ' use', 'v', 'regarding', 'respectively', 'please', ' could', 'gave',
'page', 'alone', 'seven', 'seems', 'mostly', ' itself', 'whereupon', 'unlikely', 'shes', ' especially', 'hereby', 'j', 'sars', 'wasnt',
'along', 'forth', 'take', 'x', 'come', 'sometime', 'all', 'really', 'everything', 'substantially', 'never', 'as', ' shows', 'since',
'beforehand', 'u', 'towards', 'usefully', ' because', 'til', "they've", 'sometimes', 'beginnings', 'whence', ' is', ' on', 'okay',
'edu', 'me', 'knows', 'willing', 'used', 'happens', 'median', 'i', 'pages', "we'll", 'around', 'own', 'un', 'than', 'uses', 'owing',
'thereby', 'end', 'says', 'vs', 'biol', 'who', 'becoming', 'way', ' being', 'without', 'wherein', 'getting', ' often', ' if', 'seeing',
' regarding', ' while', 'inward', 'got', 'specifying', 'down', 'predominantly', 'almost', 'anymore', 'nowhere', 'com',
'provides', 'whereby', 'two', 'every', 'ref', 'q', 'ones', 'liked', 'any', 'above', 'tries', 'sent', 'again', 'year', ' how', 'et', 'que',
'am', 'instead', 'new', 'nay', "that'll", ' since', 'later', 'merely', 'saying', ' of', 'lest', ' are', 'may', 'is', 'specifically', 'youd', 
'should', 'proud', 'twice', 'tends', 'hospital', ' from', 'eg', ' into', 'using', 'made', ' the', 'get', 'e', 'needs', ' to', 'characteristic',
'ml', 'act', 'wheres', 'briefly','sirnas','(95%', 'las', 'para', ' by', 'recent', 'latterly', 'effect', 'beside', ' although', 'her', 'www', 'know', 'need', 'announce',
'it', ' through', 'overall', 'million', "what'll", 'far', 'have', ' which', 'line', 'gets', 'normally', 'affected', 'w', 'k', ' all', 'hence',
' have', 'very', 'vols', 'wish', 'f', 'svcv-induced', '[formula:', ' among', 'thereafter', 'taken', 'became', 'accordingly', 'begin', 'back', ' about',
'immediately', 'about', 'anyway', ' km', 'how', 'many', 'only', ' were', 'followed', 'no', "didn't", 'off', ' no', 'ca', 'few',
'show', "we've", 'whim', 'ltd', 'asking', 'noted', 'ourselves', ' significantly', ' mostly', 'whose', 'approximately', 'does', 
'has', 's', 'sub', 'being','être','text]', 'auf', 'mit', 'thoughh', 'strongly', ' thus', 'had', 'thru', 'nonetheless', ' within', 'aren', 'truly', 'meanwhile',
'whenever', 'necessary', ' shown', 'according', 'specified', 'hither', 'could', 'arent', 'want', 'of', 'looking', 'goes',
'unfortunately', 'across', 'index', 'available','il-13', 'il-13/tgf-β(1)', 'tell', ' mm', 'id', 'however', 'gives', 'hers', 'so', 'g', 'resulted', 'throughout',
'p', 'from', 'done', "isn't", 'let','bei', 'von', 'los' 'ours', 'do', ' then', ' might', 'go', 'primarily', ' in', "can't", 'those', 'six', 'within', 'him',
'day', ' may', 'before', ' them', 'necessarily', 'wants', ' also', 'our', 'theyre', 'containing', 'auth', 'yet', 'regardless', ' p',
'somethan', 'given', 'five','2015', 'information', "'ll", 'toward', 'there', ' having', "i've", 'like', ' so', 'them', 'took', ' very', 'same',
'the', 'most', 'always', 'keepkeeps', 'respectively),', 'fah', 'promptly', 'this', 'hi', ' an', 'significantly', 'research', 'put', 'anything', 'thousand',
'potentially', 'shown', 'ts','(or', "'ve", 'relatively', 'howbeit', 'perhaps', ' between', 'tip', ' such', 'thereof', ' enough', ' here',
'theirs', 'often', 'also', 'self', 'though', 'wouldnt','ssp.', ' does', 'eight', 'refs', 'ncov', ' before', 'likely', 'unless', ' showed', 'thered',
'aside', ' either', 'whom', ' these', 'came', 'ever', "who'll", 'anyhow', 'onto', 'among', 'formerly', 'certain', 'hed', 'latter',
'looks', 'via', 'inc', 'downwards','1a5','sr','rsv', 'near', 'below', 'therere', 'youre', 'yours', ' however', 'whos', 'b', 'something', ' and',
'added', 'another', 'lately', "she'll",'tgf-β(1)', 'tried', 'wed', ' found', 'giving', 'arise', 'someone', 'these', 'widely', ' with', 'recently',
'meantime', 'furthermore', 'why', 'c', 'follows', 'soon','-', 'placed', 'somebody', ' both', 'showns', 'whats', 'wherever', 'saw',
'world', 'arg', 'svcv','30','il-13rα(2)','und','il','svcv.', '2019-ncov','-','p=','ps', '(i.e.,', '(md:', '95%ci:', '(arr:', 'el', 'del','contains','lamp', 'di', 'le', 'bvdv', 'therefore', 'through','http','amp','rt','t','(p','m.','c','the','y','la', 'en', 'de', '=', '+/-', 'ct', 'der', 'par', 'des','±']
```


```python
##Feature Utilities
from pyspark.ml.feature import Tokenizer, StopWordsRemover, CountVectorizer

#Tokenize
tokenizer = Tokenizer(inputCol='abstract',outputCol='words')

tokenized = tokenizer.transform(df_demo)

#StopWords
stopwordList.extend(StopWordsRemover().getStopWords())
stopwordList2 = list(set(stopwordList))
stopwords = StopWordsRemover(inputCol=tokenizer.getOutputCol(), outputCol="input_stop",stopWords=stopwordList2)

removed = stopwords.transform(tokenized)

#CountVectorizer
cv = CountVectorizer(inputCol=stopwords.getOutputCol(), outputCol="features")
cv_model = cv.fit(removed)
result = cv_model.transform(removed)
```

#### Model Evaluation


```python
#model evaluation using log perplexity and logliklihood
#you want the lowest perplexity value (not sure about logliklihood, I don't think you use that as an actual fit index)
from pyspark.ml.clustering import LDA
from pyspark.ml import Pipeline
for i in range(2,13):
    lda = LDA(k=i, featuresCol='features')
    lda_model = lda.fit(result)
    
    ll = lda_model.logLikelihood(result)
    lp = lda_model.logPerplexity(result)
    print("k="+ str(i) + "The lower bound on the log likelihood of the entire corpus: " + str(ll))    
    print("k="+ str(i) + "The upper bound on perplexity: " + str(lp))
```

    k=2The lower bound on the log likelihood of the entire corpus: -1232033.8796788356
    k=2The upper bound on perplexity: 9.5939344926633
    k=3The lower bound on the log likelihood of the entire corpus: -1250885.4843992703
    k=3The upper bound on perplexity: 9.740733264801431
    k=4The lower bound on the log likelihood of the entire corpus: -1264830.0362694783
    k=4The upper bound on perplexity: 9.849320471191565
    k=5The lower bound on the log likelihood of the entire corpus: -1289385.4159899985
    k=5The upper bound on perplexity: 10.04053494050677
    k=6The lower bound on the log likelihood of the entire corpus: -1323642.3012117401
    k=6The upper bound on perplexity: 10.307295715645315
    k=7The lower bound on the log likelihood of the entire corpus: -1355257.1486839715
    k=7The upper bound on perplexity: 10.553482756965312
    k=8The lower bound on the log likelihood of the entire corpus: -1392679.0231465516
    k=8The upper bound on perplexity: 10.844889525974175
    k=9The lower bound on the log likelihood of the entire corpus: -1435046.0331835593
    k=9The upper bound on perplexity: 11.174804413583448
    k=10The lower bound on the log likelihood of the entire corpus: -1479877.496895546
    k=10The upper bound on perplexity: 11.52391017533014
    k=11The lower bound on the log likelihood of the entire corpus: -1525509.2196417032
    k=11The upper bound on perplexity: 11.879247610472856
    k=12The lower bound on the log likelihood of the entire corpus: -1579501.9189508231
    k=12The upper bound on perplexity: 12.299692558292632
    

#### Topic Clusters


```python
#Clustering 2-12 models

from pyspark.ml.clustering import LDA
from pyspark.ml import Pipeline
for i in range(2,13):
#Train LDA model
    lda = LDA(k=i, featuresCol='features')

    pipeline = Pipeline(stages=[tokenizer, stopwords, cv, lda])
    pipelineModel = pipeline.fit(df_demo)

    topivocab = pipelineModel.stages[2].vocabulary
    topics = pipelineModel.stages[3].describeTopics(5)
    topics_rdd = topics.rdd
    topics_words = topics_rdd\
       .map(lambda row: row['termIndices'])\
       .map(lambda idx_list: [topivocab[idx] for idx in idx_list])\
       .collect()

    print("k="+ str(i) + str(topics_words))
```

    k=2[['clinical', 'respiratory', 'disease', 'health', 'virus'], ['kim,', 'lee,', 'park,', 'allergic', 'young']]
    k=3[['viral', 'respiratory', 'virus', 'viruses', 'samples'], ['kim,', 'lee,', 'park,', 'allergic', 'young'], ['clinical', 'health', 'disease', 'infection', 'acute']]
    k=4[['kim,', 'lee,', 'park,', 'nasal', 'pig'], ['kim,', 'lee,', 'park,', 'allergic', 'young'], ['frameshifting', 'protein', 'half', 'mitochondrial', 'hbx'], ['clinical', 'respiratory', 'disease', 'health', 'virus']]
    k=5[['pig', 'innovation', 'samples', 'nasal', 'processes'], ['kim,', 'lee,', 'park,', 'allergic', 'young'], ['protein', 'frameshifting', 'mrna', 'half', 'mitochondrial'], ['clinical', 'respiratory', 'disease', 'health', 'virus'], ['kim,', 'lee,', 'park,', 'van', 'les']]
    k=6[['pig', 'innovation', 'samples', 'nasal', 'processes'], ['kim,', 'lee,', 'park,', 'allergic', 'young'], ['protein', 'hbx', 'mitochondrial', 'aromatic', 'membrane'], ['clinical', 'respiratory', 'viral', 'virus', 'disease'], ['kim,', 'lee,', 'park,', 'van', 'ji'], ['health', 'covid-19', 'disease', 'pandemic', 'response']]
    k=7[['pig', 'samples', 'innovation', 'domain', 'viruses'], ['kim,', 'lee,', 'park,', 'allergic', 'young'], ['aromatic', "2'-o", 'herbal', 'cancer', 'protein'], ['clinical', 'respiratory', 'disease', 'health', 'virus'], ['kim,', 'lee,', 'park,', 'van', 'ji'], ['kim,', 'health', 'lee,', 'covid-19', 'extract'], ['les', 'delle', 'dei', 'lesions', 'brain']]
    k=8[['pig', 'innovation', 'nasal', 'samples', 'processes'], ['kim,', 'lee,', 'park,', 'allergic', 'young'], ['aromatic', 'cancer', "2'-o", 'herbal', 'quality'], ['health', 'clinical', 'data', 'disease', 'covid-19'], ['kim,', 'van', 'lee,', 'les', 'park,'], ['kim,', 'lee,', 'allergic', 'health', 'psoriasis'], ['delle', 'dei', 'lesions', 'loro', 'più'], ['respiratory', 'virus', 'viral', 'clinical', 'human']]
    k=9[['pig', 'innovation', 'samples', 'nasal', 'processes'], ['kim,', 'lee,', 'park,', 'allergic', 'young'], ['aromatic', 'quality', "2'-o", 'herbal', 'cancer'], ['laparoscopic', 'surgery', 'angle', 'femoral', 'surgical'], ['kim,', 'van', 'lee,', 'les', 'park,'], ['kim,', 'lee,', 'allergic', 'park,', 'choi,'], ['delle', 'dei', 'lesions', 'loro', 'più'], ['clinical', 'respiratory', 'disease', 'virus', 'health'], ['cerebral', 'map', '95%', 'cpb', 'autoregulation']]
    k=10[['pig', 'innovation', 'nasal', 'processes', 'samples'], ['kim,', 'lee,', 'park,', 'allergic', 'young'], ['frameshifting', 'half', 'stem-loop', 'aromatic', 'top'], ['kim,', 'lee,', 'park,', 'cardiac', 'allergic'], ['les', 'kim,', 'te', 'sie', 'van'], ['kim,', 'lee,', 'allergic', 'park,', 'psoriasis'], ['delle', 'dei', 'lesions', 'loro', 'più'], ['ultrasound', 'respiratory', 'virus', 'human', 'carotid'], ['cerebral', 'map', '95%', 'autoregulation', 'cpb'], ['clinical', 'respiratory', 'disease', 'health', 'virus']]
    k=11[['pig', 'innovation', 'nasal', 'processes', 'samples'], ['kim,', 'lee,', 'park,', 'allergic', 'young'], ['aromatic', "2'-o", 'herbal', 'cancer', 'quality'], ['kim,', 'lee,', 'park,', 'allergic', 'atopic'], ['les', 'liver', 'sie', 'die', 'acute'], ['kim,', 'lee,', 'allergic', 'park,', 'choi,'], ['delle', 'dei', 'lesions', 'loro', 'più'], ['carotid', 'endovascular', 'occluded', 'ultrasound', 'infections'], ['plastic', 'pressure', 'cerebral', 'main', '95%'], ['respiratory', 'disease', 'clinical', 'virus', 'health'], ['clinical', 'methods', 'laparoscopic', 'performed', 'associated']]
    k=12[['pig', 'innovation', 'nasal', 'processes', 'samples'], ['kim,', 'lee,', 'park,', 'allergic', 'young'], ['aromatic', "2'-o", 'region', 'conserved', 'mrna'], ['kim,', 'lee,', 'park,', 'allergic', 'atopic'], ['les', 'sie', 'die', 'liver', 'acute'], ['kim,', 'lee,', 'allergic', 'park,', 'choi,'], ['delle', 'dei', 'lesions', 'loro', 'più'], ['carotid', 'endovascular', 'ultrasound', 'occluded', 'guidance'], ['plastic', 'sars-cov-2', 'reaction', 'main', 'inhibitors'], ['virus', 'viral', 'disease', 'cell', 'human'], ['laparoscopic', 'methods', 'surgery', 'treatment', 'surgical'], ['health', 'clinical', 'respiratory', 'severe', 'covid-19']]
    

#### Evaluation using complete dataset


```python
#Evaluation of the chosen LDA model using the df_noNull dataset
```

#### Model Construction

#### Tokenize


```python
#Tokenize using FULL DATASET
tokenizer = Tokenizer(inputCol='abstract',outputCol='words')

tokenized = tokenizer.transform(df_noNull)
tokenized.show(2)
```

    +--------+--------------------+--------+--------------------+--------------------+--------+---------+-------+--------------------+------------+--------------------+--------------+------+----------------+--------+--------------------+--------------------+--------------------+-----+--------------------+
    |cord_uid|                 sha|source_x|               title|                 doi|   pmcid|pubmed_id|license|            abstract|publish_time|             authors|       journal|mag_id|who_covidence_id|arxiv_id|      pdf_json_files|      pmc_json_files|                 url|s2_id|               words|
    +--------+--------------------+--------+--------------------+--------------------+--------+---------+-------+--------------------+------------+--------------------+--------------+------+----------------+--------+--------------------+--------------------+--------------------+-----+--------------------+
    |ug7v899j|d1aafb70c066a2068...|     PMC|Clinical features...|10.1186/1471-2334...|PMC35282| 11472636|  no-cc|OBJECTIVE: This r...|  2001-07-04|Madani, Tariq A; ...|BMC Infect Dis|  null|            null|    null|document_parses/p...|document_parses/p...|https://www.ncbi....| null|[objective:, this...|
    |02tnwd4m|6b0567729c2143a66...|     PMC|Nitric oxide: a p...|        10.1186/rr14|PMC59543| 11667967|  no-cc|Inflammatory dise...|  2000-08-15|Vliet, Albert van...|    Respir Res|  null|            null|    null|document_parses/p...|document_parses/p...|https://www.ncbi....| null|[inflammatory, di...|
    +--------+--------------------+--------+--------------------+--------------------+--------+---------+-------+--------------------+------------+--------------------+--------------+------+----------------+--------+--------------------+--------------------+--------------------+-----+--------------------+
    only showing top 2 rows
    
    

#### StopWord Remover


```python
from pyspark.ml.feature import StopWordsRemover

#Remove stopwords
stopwordList.extend(StopWordsRemover().getStopWords())
stopwordList2 = list(set(stopwordList))
stopwords = StopWordsRemover(inputCol=tokenizer.getOutputCol(), outputCol="input_stop",stopWords=stopwordList2)

removed = stopwords.transform(tokenized)
removed.show(2)
```

    +--------+--------------------+--------+--------------------+--------------------+--------+---------+-------+--------------------+------------+--------------------+--------------+------+----------------+--------+--------------------+--------------------+--------------------+-----+--------------------+--------------------+
    |cord_uid|                 sha|source_x|               title|                 doi|   pmcid|pubmed_id|license|            abstract|publish_time|             authors|       journal|mag_id|who_covidence_id|arxiv_id|      pdf_json_files|      pmc_json_files|                 url|s2_id|               words|          input_stop|
    +--------+--------------------+--------+--------------------+--------------------+--------+---------+-------+--------------------+------------+--------------------+--------------+------+----------------+--------+--------------------+--------------------+--------------------+-----+--------------------+--------------------+
    |ug7v899j|d1aafb70c066a2068...|     PMC|Clinical features...|10.1186/1471-2334...|PMC35282| 11472636|  no-cc|OBJECTIVE: This r...|  2001-07-04|Madani, Tariq A; ...|BMC Infect Dis|  null|            null|    null|document_parses/p...|document_parses/p...|https://www.ncbi....| null|[objective:, this...|[objective:, retr...|
    |02tnwd4m|6b0567729c2143a66...|     PMC|Nitric oxide: a p...|        10.1186/rr14|PMC59543| 11667967|  no-cc|Inflammatory dise...|  2000-08-15|Vliet, Albert van...|    Respir Res|  null|            null|    null|document_parses/p...|document_parses/p...|https://www.ncbi....| null|[inflammatory, di...|[inflammatory, di...|
    +--------+--------------------+--------+--------------------+--------------------+--------+---------+-------+--------------------+------------+--------------------+--------------+------+----------------+--------+--------------------+--------------------+--------------------+-----+--------------------+--------------------+
    only showing top 2 rows
    
    

#### CountVectorizer


```python
from pyspark.ml.feature import CountVectorizer
cv = CountVectorizer(inputCol=stopwords.getOutputCol(), outputCol="features")
cv_model = cv.fit(removed)
result = cv_model.transform(removed)
result.show(2)
```

    +--------+--------------------+--------+--------------------+--------------------+--------+---------+-------+--------------------+------------+--------------------+--------------+------+----------------+--------+--------------------+--------------------+--------------------+-----+--------------------+--------------------+--------------------+
    |cord_uid|                 sha|source_x|               title|                 doi|   pmcid|pubmed_id|license|            abstract|publish_time|             authors|       journal|mag_id|who_covidence_id|arxiv_id|      pdf_json_files|      pmc_json_files|                 url|s2_id|               words|          input_stop|            features|
    +--------+--------------------+--------+--------------------+--------------------+--------+---------+-------+--------------------+------------+--------------------+--------------+------+----------------+--------+--------------------+--------------------+--------------------+-----+--------------------+--------------------+--------------------+
    |ug7v899j|d1aafb70c066a2068...|     PMC|Clinical features...|10.1186/1471-2334...|PMC35282| 11472636|  no-cc|OBJECTIVE: This r...|  2001-07-04|Madani, Tariq A; ...|BMC Infect Dis|  null|            null|    null|document_parses/p...|document_parses/p...|https://www.ncbi....| null|[objective:, this...|[objective:, retr...|(262144,[1,3,5,8,...|
    |02tnwd4m|6b0567729c2143a66...|     PMC|Nitric oxide: a p...|        10.1186/rr14|PMC59543| 11667967|  no-cc|Inflammatory dise...|  2000-08-15|Vliet, Albert van...|    Respir Res|  null|            null|    null|document_parses/p...|document_parses/p...|https://www.ncbi....| null|[inflammatory, di...|[inflammatory, di...|(262144,[1,4,13,3...|
    +--------+--------------------+--------+--------------------+--------------------+--------+---------+-------+--------------------+------------+--------------------+--------------+------+----------------+--------+--------------------+--------------------+--------------------+-----+--------------------+--------------------+--------------------+
    only showing top 2 rows
    
    

#### Model Evaluation


```python
#Train LDA model
lda = LDA(k=2, featuresCol='features')
```


```python
#Pipeline
pipeline = Pipeline(stages=[tokenizer, stopwords, cv, lda])
pipelineModel = pipeline.fit(df_noNull)
```

#### Topic Clusters


```python
topivocab = pipelineModel.stages[2].vocabulary
topics = pipelineModel.stages[3].describeTopics(5)
topics_rdd = topics.rdd
topics_words = topics_rdd\
    .map(lambda row: row['termIndices'])\
    .map(lambda idx_list: [topivocab[idx] for idx in idx_list])\
    .collect()
for idx, topic in enumerate(topics_words):
    print("topic: ", idx)
    print("----------")
    for word in topic:
        print(word)
        print("----------")
```

    topic:  0
    ----------
    respiratory
    ----------
    clinical
    ----------
    disease
    ----------
    health
    ----------
    infection
    ----------
    topic:  1
    ----------
    virus
    ----------
    viral
    ----------
    protein
    ----------
    rna
    ----------
    cells
    ----------
    

## Classification Modeling and Prediction

### Data Cleaning and Extraction


```python
print('Dataframe Structure')
print('----------------------------------')
print(data_clean.printSchema())
print(' ')
print('Dataframe preview')
print(data_clean.show(5))
print(' ')
print('----------------------------------')
print('Total number of rows', data_clean.count())
```

    Dataframe Structure
    ----------------------------------
    root
     |-- Journal: string (nullable = true)
     |-- Abstract: string (nullable = true)
     |-- Publish_Time: string (nullable = true)
    
    None
     
    Dataframe preview
    +--------------+--------------------+------------+
    |       Journal|            Abstract|Publish_Time|
    +--------------+--------------------+------------+
    |bmc infect dis|objective: this r...|  2001-07-04|
    |    respir res|inflammatory dise...|  2000-08-15|
    |    respir res|surfactant protei...|  2000-08-25|
    |    respir res|endothelin-1 (et-...|  2001-02-22|
    |    respir res|respiratory syncy...|  2001-05-11|
    +--------------+--------------------+------------+
    only showing top 5 rows
    
    None
     
    ----------------------------------
    Total number of rows 100998
    


```python
def top_n_list(df,var, N):
    '''
    This function determine the top N numbers of the list
    '''
    print("Total number of unique value of"+' '+var+''+':'+' '+str(df.select(var).distinct().count()))
    print(' ')
    print('Top'+' '+str(N)+' '+' '+var)
    df.groupBy(var).count().withColumnRenamed('count','totalValue')\
    .orderBy(col('totalValue').desc()).show(N)
    return (df.groupBy(var).count().withColumnRenamed('count','totalValue')\
    .orderBy(col('totalValue').desc()))
   
   
top_journals = top_n_list(data_clean, 'Journal',100)
print(' ')
```

    Total number of unique value of Journal: 15400
     
    Top 100  Journal
    +--------------------+----------+
    |             Journal|totalValue|
    +--------------------+----------+
    |            plos one|      1974|
    | journal of virology|      1567|
    |            virology|      1273|
    |             biorxiv|      1202|
    |  surgical endoscopy|       823|
    |             sci rep|       758|
    |             viruses|       678|
    |          arch virol|       633|
    |    emerg infect dis|       606|
    |      virus research|       580|
    |the journal of ge...|       574|
    |             vaccine|       517|
    |veterinary microb...|       500|
    |journal of virolo...|       446|
    |         j med virol|       404|
    |             virol j|       398|
    |      bmc infect dis|       377|
    |         plos pathog|       349|
    |emerging infectio...|       334|
    |journal of clinic...|       325|
    |              nature|       312|
    |  antiviral research|       311|
    |proceedings of th...|       278|
    |                 bmj|       277|
    |     obesity surgery|       275|
    |        j clin virol|       271|
    |        j infect dis|       265|
    |journal of neuroi...|       255|
    |     clin infect dis|       255|
    |journal of medica...|       252|
    |ajnr. american jo...|       250|
    |american journal ...|       241|
    |       front immunol|       237|
    |archives of virology|       229|
    |biochemical and b...|       228|
    |    int j infect dis|       226|
    |            bmj open|       219|
    |         virus genes|       218|
    |     front microbiol|       211|
    |   bmc public health|       206|
    |              stroke|       201|
    |hernia : the jour...|       197|
    |surgical laparosc...|       197|
    |   nucleic acids res|       194|
    |             science|       193|
    |mmwr. morbidity a...|       191|
    |journal of veteri...|       189|
    |           crit care|       185|
    |       j. med. virol|       185|
    |  intensive care med|       181|
    |  scientific reports|       181|
    |    respiratory care|       179|
    |         bmc vet res|       178|
    |influenza other r...|       166|
    |advances in exper...|       164|
    |jsls : journal of...|       164|
    |            j infect|       163|
    |  systematic reviews|       159|
    |veterinary immuno...|       158|
    |the pediatric inf...|       158|
    |               chest|       157|
    |              lancet|       157|
    |the journal of bi...|       156|
    |the journal of in...|       155|
    |      avian diseases|       153|
    |                mbio|       149|
    |cardiovascular an...|       149|
    |  plos negl trop dis|       148|
    |clinical infectio...|       147|
    |advances in infor...|       144|
    |emerg microbes in...|       144|
    |     j virol methods|       143|
    |       int j mol sci|       139|
    |                cell|       138|
    |eur j clin microb...|       136|
    |advances in knowl...|       135|
    |journal of feline...|       132|
    |   sci total environ|       132|
    |open forum infect...|       130|
    |          the lancet|       130|
    |journal of laparo...|       129|
    |           molecules|       129|
    |int j environ res...|       128|
    |world journal of ...|       127|
    | surgical innovation|       121|
    |the cochrane data...|       117|
    |          nat commun|       116|
    |           virus res|       111|
    |american journal ...|       109|
    |journal of gastro...|       108|
    |journal of hospit...|       107|
    |journal of wildli...|       107|
    |interventional ne...|       105|
    |             vet res|       105|
    |       critical care|       104|
    |            medicine|       102|
    |critical care med...|       102|
    |           radiology|       101|
    |revue scientifiqu...|       100|
    |         res vet sci|        98|
    +--------------------+----------+
    only showing top 100 rows
    
     
    


```python
top_10 = ["plos one",
"journal of virology",
"virology",
"biorxiv",
"surgical endoscopy",
"sci rep",
"viruses",
"arch virol",
"emerg infect dis",
"virus research"]
```


```python
top_100 = top_journals.select('Journal').rdd.map(lambda entry : entry[0]).collect()[:100]
```


```python
import pyspark.sql.functions as F
data_top100 = data_clean.filter(F.col("Journal").isin(top_100))
```

### Partition the dataset into Training and Test dataset


```python
training, test = data_top100.randomSplit([0.7,0.3], seed=42)
print("Training Dataset Count:", training.count())
print("Test Dataset Count:", test.count())
```

    Training Dataset Count: 19698
    Test Dataset Count: 8319
    


```python
from pyspark.ml.feature import RegexTokenizer, StopWordsRemover, CountVectorizer, OneHotEncoder, StringIndexer, VectorAssembler, HashingTF, IDF, Word2Vec
from pyspark.ml import Pipeline
from pyspark.ml.classification import LogisticRegression, NaiveBayes 

#----------------Define tokenizer with regextokenizer()------------------
regex_tokenizer = RegexTokenizer(pattern='\\W')\
                  .setInputCol("Abstract")\
                  .setOutputCol("tokens")

#----------------Define stopwords with stopwordsremover()---------------------
extra_stopwords = ['http','amp','rt','t','c','the']
stopwords_remover = StopWordsRemover()\
                    .setInputCol('tokens')\
                    .setOutputCol('filtered_words')\
                    .setStopWords(extra_stopwords)
                    

#----------Define bags of words using countVectorizer()---------------------------
count_vectors = CountVectorizer(vocabSize=10000, minDF=5)\
               .setInputCol("filtered_words")\
               .setOutputCol("features")


#-----------Using TF-IDF to vectorise features instead of countVectoriser-----------------
hashingTf = HashingTF(numFeatures=10000)\
            .setInputCol("filtered_words")\
            .setOutputCol("raw_features")
            
#Use minDocFreq to remove sparse terms
idf = IDF(minDocFreq=5)\
        .setInputCol("raw_features")\
        .setOutputCol("features")

#---------------Define bag of words using Word2Vec---------------------------
word2Vec = Word2Vec(vectorSize=1000, minCount=0)\
           .setInputCol("filtered_words")\
           .setOutputCol("features")

#-----------Encode the Category variable into label using StringIndexer-----------
label_string_idx = StringIndexer()\
                  .setInputCol("Journal")\
                  .setOutputCol("label")

#-----------Define classifier structure for logistic Regression--------------
lr = LogisticRegression(maxIter=20, regParam=0.3, elasticNetParam=0)

#---------Define classifier structure for Naive Bayes----------
nb = NaiveBayes(smoothing=1)


```

* Define tokenization function using RegexTokenizer: RegexTokenizer allows more advanced tokenization based on regular expression (regex) matching. By  default, the parameter “pattern” (regex,  default: “\s+”) is used as delimiters to split the input text. Alternatively, users can set parameter “gaps” to false indicating the regex “pattern” denotes “tokens” rather than splitting gaps, and find all matching occurrences as the tokenization result.

* Define stop remover function using StopWordsRemover: StopWordsRemover takes as input a sequence of strings (e.g. the output of a Tokenizer) and drops all the stop words from the input sequences. The list of stopwords is specified by the stopWords parameter.

* Define bag of words function for Descript variable using CountVectorizer: CountVectorizer can be used as an estimator to extract the vocabulary, and generates a CountVectorizerModel. The model produces sparse representations for the documents over the vocabulary, which can then be passed to other algorithms like LDA. During the fitting process, CountVectorizer will select the top vocabSize words ordered by term frequency across the corpus. An optional parameter minDF also affects the fitting process by specifying the minimum number (or fraction if < 1.0) of documents a term must appear in to be included in the vocabulary.

* Define function to Encode the values of category variable using StringIndexer: StringIndexer encodes a string column of labels to a column of label indices. The indices are in (0, numLabels), ordered by label frequencies, so the most frequent label gets index 0. In our case, the label colum(Category) will be encoded to label indices, from 0 to 38; the most frequent label (LARCENY/THEFT) will be indexed as 0.

* Define a pipeline to call these functions: ML Pipelines provide a uniform set of high-level APIs built on top of DataFrames that help users create and tune practical machine learning pipelines.

### Distribution of Number of Papers in the Top 10 Journals


```python
from pyspark.sql.functions import desc
data_viz = data_top10.groupby('journal').count().orderBy(desc('count'))

%matplotlib inline
import matplotlib.pyplot as plt
plt.style.use('ggplot')

df_pd = data_viz.toPandas()
```


```python
df_pd.plot.bar(x='journal', y='count', rot=0, stacked=True)
plt.xticks(rotation=90)
```




    (array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]), <a list of 10 Text xticklabel objects>)




    
![png](Group3_FinalCode_files/Group3_FinalCode_73_1.png)
    



```python
data_top10 = data_clean.filter(F.col("Journal").isin(top_10))
```

### BASELINE MODEL Logistic Regression with Count Vector Features


```python
pipeline_cv_lr = Pipeline().setStages([regex_tokenizer,stopwords_remover,count_vectors,label_string_idx, lr])
model_cv_lr = pipeline_cv_lr.fit(training)
predictions_cv_lr = model_cv_lr.transform(test)
```


```python
print('-----------------------------Check Top 5 predictions----------------------------------')
print(' ')
predictions_cv_lr.select('Abstract','Journal',"probability","label","prediction")\
                                        .orderBy("probability", ascending=False)\
                                        .show(n=5, truncate=30)
```

    -----------------------------Check Top 5 predictions----------------------------------
     
    +------------------------------+--------+------------------------------+-----+----------+
    |                      Abstract| Journal|                   probability|label|prediction|
    +------------------------------+--------+------------------------------+-----+----------+
    |background: the white backe...|plos one|[0.8865895210202326,0.00232...|  0.0|       0.0|
    |the epithelial-mesenchymal ...|plos one|[0.8811483028847181,0.00190...|  0.0|       0.0|
    |background: it is a commonl...|plos one|[0.8291103648226354,0.00362...|  0.0|       0.0|
    |transcription factor 7-like...| biorxiv|[0.8091226923933745,0.01012...|  3.0|       0.0|
    |andalusia (southern spain) ...|plos one|[0.8018967425747101,0.00217...|  0.0|       0.0|
    +------------------------------+--------+------------------------------+-----+----------+
    only showing top 5 rows
    
    


```python
from pyspark.ml.evaluation import MulticlassClassificationEvaluator 
evaluator_cv_lr = MulticlassClassificationEvaluator().setPredictionCol("prediction").evaluate(predictions_cv_lr)
print(' ')
print('------------------------------Accuracy----------------------------------')
print(' ')
print('                       accuracy:{}:'.format(evaluator_cv_lr))
```

     
    ------------------------------Accuracy----------------------------------
     
                           accuracy:0.3049739185075633:
    

### BASELINE MODEL Naive Bayes with Count Vector Features


```python
pipeline_cv_nb = Pipeline().setStages([regex_tokenizer,stopwords_remover,count_vectors,label_string_idx, nb])
model_cv_nb = pipeline_cv_nb.fit(training)
predictions_cv_nb = model_cv_nb.transform(test)
```


```python
evaluator_cv_nb = MulticlassClassificationEvaluator().setPredictionCol("prediction").evaluate(predictions_cv_nb)
print(' ')
print('--------------------------Accuracy-----------------------------')
print(' ')
print('                      accuracy:{}:'.format(evaluator_cv_nb))
```

     
    --------------------------Accuracy-----------------------------
     
                          accuracy:0.3206948342902863:
    

### Logistic Regression Using TF-IDF Features


```python
pipeline_idf_lr = Pipeline().setStages([regex_tokenizer,stopwords_remover,hashingTf, idf, label_string_idx, lr])
model_idf_lr = pipeline_idf_lr.fit(training)
predictions_idf_lr = model_idf_lr.transform(test)
```


```python
evaluator_idf_lr = MulticlassClassificationEvaluator().setPredictionCol("prediction").evaluate(predictions_idf_lr)
print(' ')
print('-------------------------------Accuracy---------------------------------')
print(' ')
print('                        accuracy:{}:'.format(evaluator_idf_lr))
```

     
    -------------------------------Accuracy---------------------------------
     
                            accuracy:0.26977707536164813:
    

### Naive Bayes with TF-IDF Features


```python
pipeline_idf_nb = Pipeline().setStages([regex_tokenizer,stopwords_remover,hashingTf, idf, label_string_idx, nb])
model_idf_nb = pipeline_idf_nb.fit(training)
predictions_idf_nb = model_idf_nb.transform(test)
```


```python
evaluator_idf_nb = MulticlassClassificationEvaluator().setPredictionCol("prediction").evaluate(predictions_idf_nb)
print(' ')
print('-----------------------------Accuracy-----------------------------')
print(' ')
print('                          accuracy:{}:'.format(evaluator_idf_nb))
```

     
    -----------------------------Accuracy-----------------------------
     
                              accuracy:0.32809008909122866:
    


```python

```
