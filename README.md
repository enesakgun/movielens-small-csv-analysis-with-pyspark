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

# Task-1 Exploratory Data Analysis.ipynb

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
