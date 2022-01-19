# movielens-small-csv-analysis-with-pyspark
In the study, data analysis for Movielens-Small data was carried out using Apache Spark via Jupyter Notebook.
## Setup
  ### JDK 8 Setup for Windows 
  Download Java SE Development Kit 8u321 for Windows.
 (Control Panel -> System and Security -> System -> Advanced System Settings -> Environment Variables) create a new variable. We define variable name: “SPARK_HOME” value: 
We select Path in environment variables and say edit and add %SPARK_HOME%\bin to Path. 
  ### Pyspark setup
  https://spark.apache.org/downloads.html
  1. Opening C:\spark file in PC
  2. Then we delete the .template extension of the log4j.properties.template file in the conf folder and open it with any text editor, change the log4j.rootCategory=INFO line to log4j.rootCategory=ERROR and save it.
 3. We come to the environment variables of Windows (Control Panel -> System and Security -> System -> Advanced System Settings -> Environment Variables) and create a new variable. We define variable name: “SPARK_HOME” value: “C:\spark”.
Again, we select Path in the environment variables and say edit and add %SPARK_HOME%\bin to Path.
  ### Hadoop Setup
  1. Download "winutils.exe" for Hadoop
  2. Opening C:\spark file in PC Copy winutils.exe to Hadoop File
  3. Go to the C disk and create the C:\tmp\hive directory.
  It opens with the option to run command line as administrator. In the directory where winutils.exe is located, write the command: `chmod -R 777 C:\tmp\hive`
  Finally, write the `spark-shell` command on the command line and check if the installation is complete.
  ### Anaconda Download
  1. Eğer pc'nizde kurulu değilse https://www.anaconda.com/products/individual sitesi üzerinden anaconda kurulumu yapılır.
  2. Apache Spark'ı Jupyter Notebook üzerinden kullanabilmek için, (Denetim Masası ->Sistem ve Güvenlik -> Sistem -> Gelişmiş Sistem Ayarları -> Ortam Değişkenleri) alanından Path'i düzenleye basarak, C:\Users\EnesA\anaconda3 Anaconda ' nın kurulu olduğu dizin eklenir. C:\Users\EnesA\anaconda3\Scripts anacondanın kurulu olduğu alandaki Scripts klasörünün dizini eklenir.

Yukarıdaki bütün kurulumlar tamamlandıktan sonra çalışmaya başlanabilecektir.

# Task-1 Exploratory Data Analysis

1.  First of all, a new page is opened by saying File>New Notebook on jupyter notebook. 
2. By typing the `pwd` command, the save location of jupyter is learned and the data to be used is loaded here.  In this study, "MovieLens 25M movie ratings" data is downloaded via https://grouplens.org/datasets/movielens/25m/ link. The downloaded .csv files are placed in the **C:\Users\EnesA** directory. 
3. To import the Python libraries to be used, write the `!pip install` command for downloading. Example: `!pip install pyspark ` Example: `!pip install pyspark `
4.Libraries are imported. 
```import pyspark
import matplotlib.pyplot as plt
import pandas as pd
import seaborn as sns
from pyspark.sql import SparkSession 
```
5. The command is run to start a session over Spark. To use the data, the required movielens data is drawn from the movielens data that we have downloaded before. If you get an error when you call the ml-25m folder, you may need to add the **C:\Users\EnesA\ml-25m** directory to the Path in the Environment variables. The **.show** command allows you to see if the data is coming.
```
spark = SparkSession.builder.appName('MovieLens').getOrCreate()
ratings = spark.read.option("header", "true").csv("ml-25m/ratings.csv")
ratings.show(5)
+------+-------+------+----------+
|userId|movieId|rating| timestamp|
+------+-------+------+----------+
|     1|    296|   5.0|1147880044|
|     1|    306|   3.5|1147868817|
|     1|    307|   5.0|1147868828|
|     1|    665|   5.0|1147878820|
|     1|    899|   3.5|1147868510|
+------+-------+------+----------+
only showing top 5 rows
movies = spark.read.option("header", "true").csv("ml-25m/movies.csv")
movies.show(5)
+-------+--------------------+--------------------+
|movieId|               title|              genres|
+-------+--------------------+--------------------+
|      1|    Toy Story (1995)|Adventure|Animati...|
|      2|      Jumanji (1995)|Adventure|Childre...|
|      3|Grumpier Old Men ...|      Comedy|Romance|
|      4|Waiting to Exhale...|Comedy|Drama|Romance|
|      5|Father of the Bri...|              Comedy|
+-------+--------------------+--------------------+
only showing top 5 rows

```
6. When using sql directly over the session, the "movies" "ratings" variables defined to pull the data are converted into a sql table. Now you can start working on the data.
```
movies.createOrReplaceTempView("Movies")
ratings.createOrReplaceTempView("Ratings")
```
## 1.Question - Write a SQL query to create a dataframe with including userid, movieid, genre and rating

1. In this question, data can be retrieved in 2 different ways. Since data is requested from 2 separate tables, a join is required. First, **Movies** and **Ratings** tables can be linked to each other using inner join. Or tables can be linked to each other in the **where** field as in the 2nd example.
```
dataframe=spark.sql("select r.userId,m.movieId,m.genres,r.rating from Movies m inner join Ratings r on r.movieId=m.movieId").show()
+------+-------+--------------------+------+
|userId|movieId|              genres|rating|
+------+-------+--------------------+------+
|     1|    296|Comedy|Crime|Dram...|   5.0|
|     1|    306|               Drama|   3.5|
|     1|    307|               Drama|   5.0|
|     1|    665|    Comedy|Drama|War|   5.0|
|     1|    899|Comedy|Musical|Ro...|   3.5|
|     1|   1088|Drama|Musical|Rom...|   4.0|
|     1|   1175|Comedy|Drama|Romance|   3.5|
|     1|   1217|           Drama|War|   3.5|
|     1|   1237|               Drama|   5.0|
|     1|   1250| Adventure|Drama|War|   4.0|
|     1|   1260|Crime|Film-Noir|T...|   3.5|
|     1|   1653|Drama|Sci-Fi|Thri...|   4.0|
|     1|   2011|Adventure|Comedy|...|   2.5|
|     1|   2012|Adventure|Comedy|...|   2.5|
|     1|   2068|Drama|Fantasy|Mys...|   2.5|
|     1|   2161|Adventure|Childre...|   3.5|
|     1|   2351|               Drama|   4.5|
|     1|   2573|       Drama|Musical|   4.0|
|     1|   2632|Adventure|Drama|M...|   5.0|
|     1|   2692|        Action|Crime|   5.0|
+------+-------+--------------------+------+
only showing top 20 rows

dataframe=spark.sql("select r.userId,m.movieId,m.genres,r.rating from Movies m,Ratings r where r.movieId=m.movieId").show()
+------+-------+--------------------+------+
|userId|movieId|              genres|rating|
+------+-------+--------------------+------+
|     1|    296|Comedy|Crime|Dram...|   5.0|
|     1|    306|               Drama|   3.5|
|     1|    307|               Drama|   5.0|
|     1|    665|    Comedy|Drama|War|   5.0|
|     1|    899|Comedy|Musical|Ro...|   3.5|
|     1|   1088|Drama|Musical|Rom...|   4.0|
|     1|   1175|Comedy|Drama|Romance|   3.5|
|     1|   1217|           Drama|War|   3.5|
|     1|   1237|               Drama|   5.0|
|     1|   1250| Adventure|Drama|War|   4.0|
|     1|   1260|Crime|Film-Noir|T...|   3.5|
|     1|   1653|Drama|Sci-Fi|Thri...|   4.0|
|     1|   2011|Adventure|Comedy|...|   2.5|
|     1|   2012|Adventure|Comedy|...|   2.5|
|     1|   2068|Drama|Fantasy|Mys...|   2.5|
|     1|   2161|Adventure|Childre...|   3.5|
|     1|   2351|               Drama|   4.5|
|     1|   2573|       Drama|Musical|   4.0|
|     1|   2632|Adventure|Drama|M...|   5.0|
|     1|   2692|        Action|Crime|   5.0|
+------+-------+--------------------+------+
only showing top 20 rows
```

```
dataframe=spark.sql("select r.userId,m.movieId,m.genres,r.rating from Movies m,Ratings r where r.movieId=m.movieId").show()
```

## 2.Question - Count ratings for each movie, and list top 5 movies with the highest value

```
spark.sql("""select count(r.rating) CountRatings,m.title as MovieName from Movies m 
inner join Ratings r on r.movieId=m.movieId 
group by m.title order by count(r.rating) desc limit 5""").show()
+------------+--------------------+
|CountRatings|           MovieName|
+------------+--------------------+
|       81491| Forrest Gump (1994)|
|       81482|Shawshank Redempt...|
|       79672| Pulp Fiction (1994)|
|       74127|Silence of the La...|
|       72674|  Matrix, The (1999)|
+------------+--------------------+
```
## 3.Question - Find and list top 5 most rated genres
```
sspark.sql("""select r.rating MostRated,m.genres from Movies m 
inner join Ratings r on r.movieId=m.movieId  
order by r.rating desc limit 5""").show()

+---------+--------------------+
|MostRated|              genres|
+---------+--------------------+
|      5.0|Adventure|Drama|S...|
|      5.0|      Comedy|Romance|
|      5.0|      Horror|Mystery|
|      5.0|              Horror|
|      5.0|Drama|Horror|Thri...|
+---------+--------------------+

```

## 4.Question - By using timestamp from ratings table, provide top 5 most frequent users within a week
In this question, we convert timestamp to date for use.
```
spark.sql("""select count(*) Frequentusers,userId from Movies m 
inner join Ratings r on r.movieId=m.movieId 
where CAST(TO_DATE(FROM_UNIXTIME(r.timestamp))as date)>CAST(TO_DATE(FROM_UNIXTIME(r.timestamp))as date) - interval '1' week 
group by userId 
order by count(*) desc   limit 5""").show()
```
## 5.Question -Calculate average ratings for each genre, and plot average ratings of top 10 genres with descending order

If anyone uses the seaborn library with this data, has to convert to Pandas.
```
table= spark.sql("""

select m.genres,avg(r.rating) as AvgOfRatings  from movies m
inner join Ratings r on r.movieId=m.movieId
group by m.genres order by AvgOfRatings desc limit 10""")

table.show()
plot=table.toPandas()
+--------------------+------------+
|              genres|AvgOfRatings|
+--------------------+------------+
|Comedy|Crime|Dram...|         5.0|
|Action|Drama|Myst...|         5.0|
|Adventure|Drama|F...|         5.0|
|Children|Comedy|D...|         5.0|
|Fantasy|Horror|Ro...|         5.0|
|Adventure|Fantasy...|         5.0|
|Action|Comedy|Mys...|         5.0|
|Adventure|Drama|R...|         5.0|
|Animation|Crime|M...|       4.625|
|Action|Children|D...|         4.5|
+--------------------+------------+

```
```
sns.barplot(x="AvgOfRatings",y = plot.genres,data=plot)
```

# Task-2 Recommender Design Model

Here, the previously developed Lightfm library will be used. Since there is ready-made movielens-small database information in the Lightfm library, data can be drawn directly thanks to the library.


```
import numpy as np

from lightfm.datasets import fetch_movielens

movielens = fetch_movielens()

```
This gives us a dictionary with the following fields:
```
for key, value in movielens.items():
    print(key, type(value), value.shape)
    
('test', <class 'scipy.sparse.coo.coo_matrix'>, (943, 1682))
('item_features', <class 'scipy.sparse.csr.csr_matrix'>, (1682, 1682))
('train', <class 'scipy.sparse.coo.coo_matrix'>, (943, 1682))
('item_labels', <type 'numpy.ndarray'>, (1682,))
('item_feature_labels', <type 'numpy.ndarray'>, (1682,))

train = movielens['train']
test = movielens['test']
```
The train and test elements are the most important: they contain the raw rating data, split into a train and a test set. Each row represents a user, and each column an item. Entries are ratings from 1 to 5.

## Fitting models
Now let's train a BPR model and look at its accuracy.

We'll use two metrics of accuracy: precision@k and ROC AUC. Both are ranking metrics: to compute them, we'll be constructing recommendation lists for all of our users, and checking the ranking of known positive movies. For precision at k we'll be looking at whether they are within the first k results on the list; for AUC, we'll be calculating the probability that any known positive is higher on the list than a random negative example.
```
from lightfm import LightFM
from lightfm.evaluation import precision_at_k
from lightfm.evaluation import auc_score

model = LightFM(learning_rate=0.05, loss='bpr')
model.fit(train, epochs=10)

train_precision = precision_at_k(model, train, k=10).mean()
test_precision = precision_at_k(model, test, k=10).mean()

train_auc = auc_score(model, train).mean()
test_auc = auc_score(model, test).mean()

print('Precision: train %.2f, test %.2f.' % (train_precision, test_precision))
print('AUC: train %.2f, test %.2f.' % (train_auc, test_auc))
Precision: train 0.59, test 0.10.
AUC: train 0.90, test 0.86.

```

The WARP model, on the other hand, optimises for precision@k---we should expect its performance to be better on precision.
```
model = LightFM(learning_rate=0.05, loss='warp')

model.fit_partial(train, epochs=10)

train_precision = precision_at_k(model, train, k=10).mean()
test_precision = precision_at_k(model, test, k=10).mean()

train_auc = auc_score(model, train).mean()
test_auc = auc_score(model, test).mean()

print('Precision: train %.2f, test %.2f.' % (train_precision, test_precision))
print('AUC: train %.2f, test %.2f.' % (train_auc, test_auc))
```
Modelling result is :
```
Precision: train 0.61, test 0.11.
AUC: train 0.93, test 0.90.

```
# Task – 3 Text Analysis:

It also means sentiment analysis.
In order to perform this analysis, the following libraries must be imported.
If there are no libraries, the libraries are installed with the `!pip install` command as shown earlier. It is then imported. If the latest version of Visual Studio Code ++ is not installed on your PC, you will receive an error in the installation of some libraries.
The path to be followed for installation can be followed from the page https://docs.microsoft.com/tr-tr/cpp/build/vscpp-step-0-installation?view=msvc-170.

## Kütüphanelerin import edilmesi

```
import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
import nltk
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.preprocessing import LabelBinarizer
from nltk.corpus import stopwords
from nltk.stem.porter import PorterStemmer
from wordcloud import WordCloud,STOPWORDS
from nltk.stem import WordNetLemmatizer
from nltk.tokenize import word_tokenize,sent_tokenize
from bs4 import BeautifulSoup
import spacy
import re,string,unicodedata
from nltk.tokenize.toktok import ToktokTokenizer
from nltk.stem import LancasterStemmer,WordNetLemmatizer
from sklearn.linear_model import LogisticRegression,SGDClassifier
from sklearn.naive_bayes import MultinomialNB
from sklearn.svm import SVC
from textblob import TextBlob
from textblob import Word
from sklearn.metrics import classification_report,confusion_matrix,accuracy_score
import os
import warnings
```
The nltk library is an important tool in this analysis.
nltk: Natural Language Toolkit; It is an open-source library developed with the Python programming language to work with human language data and built with over 50 corpus and lexical resources under development.
The following command is run for this kit that needs to be installed extra on the PC.
```
import nltk
nltk.download() # it will open pop-up for installing nltk
showing info https://raw.githubusercontent.com/nltk/nltk_data/gh-pages/index.xml 
```
The process is completed by installing the setup that falls on the desktop.

## Importing Data and Data Analysis
The IMDB Dataset file is placed in the folder where Jupyter Notebook receives data. Then the following command is run and the data is retrieved.
```
imdb_data=pd.read_csv('IMDB Dataset.csv')
print(imdb_data.shape)
imdb_data.head(10)
review	sentiment
0	One of the other reviewers has mentioned that ...	positive
1	A wonderful little production. <br /><br />The...	positive
2	I thought this was a wonderful way to spend ti...	positive
3	Basically there's a family where a little boy ...	negative
4	Petter Mattei's "Love in the Time of Money" is...	positive
5	Probably my all-time favorite movie, a story o...	positive
6	I sure would like to see a resurrection of a u...	positive
7	This show was an amazing, fresh & innovative i...	negative
8	Encouraged by the positive comments about this...	negative
9	If you like original gut wrenching laughter yo...	positive
```
## Data Describe and Sentiment Count (Control) 
```
imdb_data.describe()
imdb_data['sentiment'].value.counts()
```
## Text normalization
Words are tokenized. To separate a statement into words, we utilise the word tokenize () method.
```
tokenizer=ToktokTokenizer()
stopword_list=nltk.corpus.stopwords.words('english')
```

## Removing the html strips
```
def strip_html(text):
    soup = BeautifulSoup(text, "html.parser")
    return soup.get_text()
#Removing the square brackets
def remove_between_square_brackets(text):
    return re.sub('\[[^]]*\]', '', text)
#Removing the noisy text
def denoise_text(text):
    text = strip_html(text)
    text = remove_between_square_brackets(text)
    return text
#Apply function on review column
imdb_data['review']=imdb_data['review'].apply(denoise_text)
```
## Removing special characters
Because we’re working with English-language evaluations in our dataset, we need to make sure that any special characters are deleted.
```
#Define function for removing special characters
def remove_special_characters(text, remove_digits=True):
    pattern=r'[^a-zA-z0-9\s]'
    text=re.sub(pattern,'',text)
    return text
#Apply function on review column
imdb_data['review']=imdb_data['review'].apply(remove_special_characters)
```

## Removing stopwords and normalization
Stop words are words that have little or no meaning, especially when synthesising meaningful aspects from the text.
Stop words are words that are filtered out of natural language data (text) before or after it is processed in computers. While “stop words” usually refers to a language’s most common terms, all-natural language processing algorithms don’t employ a single universal list.
Stopwords include words such as a, an, the, and others.
```
#set stopwords to english
stop=set(stopwords.words('english'))
print(stop)

#removing the stopwords
def remove_stopwords(text, is_lower_case=False):
    tokens = tokenizer.tokenize(text)
    tokens = [token.strip() for token in tokens]
    if is_lower_case:
        filtered_tokens = [token for token in tokens if token not in stopword_list]
    else:
        filtered_tokens = [token for token in tokens if token.lower() not in stopword_list]
    filtered_text = ' '.join(filtered_tokens)    
    return filtered_text
#Apply function on review column
imdb_data['review']=imdb_data['review'].apply(remove_stopwords)
{'ours', 'he', 'our', 'am', 'aren', 'each', 'yourselves', 'o', 'all', 'any', 'what', 'during', 'as', 'those', "you've", 'on', "shan't", "that'll", 'these', 'yours', 'hasn', 'who', 'of', 'doing', 'did', 'for', 'against', 'why', 'few', 'its', 's', 'will', 'weren', 'very', 'whom', 'can', 'further', 'she', 'because', 'me', 'be', "couldn't", 'than', 'being', 'down', 'should', 'isn', 'needn', 'over', "didn't", 'ain', 'had', 'd', 'again', 'some', 'been', 'where', 'm', "hadn't", "won't", 'more', "mustn't", 'once', 'does', 'and', 'under', 'myself', 'so', "it's", 'hadn', 'mustn', 'the', "shouldn't", 'their', 'while', 'were', 'himself', 'out', 'both', 'own', 'll', 'there', 'don', 'in', 'with', 'into', 'but', "aren't", 'just', 'this', 'yourself', 'here', 'nor', 'too', 'no', 'how', 'hers', 'below', 'when', 'before', 'are', 'after', "wasn't", 'couldn', 'do', 'such', 'ourselves', "wouldn't", 've', 'now', 'shan', 'itself', 'theirs', 'most', 'her', "you'd", 't', 'other', "you'll", 'y', 'won', 'your', 'having', "weren't", 'we', 'was', 'an', 'above', 'that', 'from', 'up', 'about', 'ma', 'same', "don't", "needn't", 'to', 'haven', 'not', 're', 'they', "isn't", 'off', 'themselves', 'at', 'didn', 'my', 'have', 'herself', 'a', 'by', 'or', "mightn't", 'has', 'it', 'you', "she's", 'wouldn', 'then', 'between', "you're", 'his', "haven't", 'until', "hasn't", 'through', 'mightn', 'shouldn', 'is', "should've", 'wasn', 'i', "doesn't", 'doesn', 'him', 'which', 'if', 'only', 'them'}
```
