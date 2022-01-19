# movielens-small-csv-analysis-with-pyspark
I used Pyspark in Jupyter Notebook for analyse movielens-small datas,
Çalışmada Hadoop üzerinden Jupyter Notebook aracılığıyla Apache Spark kullanılarak data analizi yapılmıştır.
## Setup
  ### JDK 8 Setup for Windows 
  Download Java SE Development Kit 8u321 for Windows.
  (Denetim Masası ->Sistem ve Güvenlik -> Sistem -> Gelişmiş Sistem Ayarları -> Ortam Değişkenleri) yeni değişken oluşturuyoruz. Değişken adını: “SPARK_HOME” değerini: “C:\spark”    olarak tanımlıyoruz.
  Yine ortam değişkenlerinde Path’i seçip düzenle diyoruz ve Path’e %SPARK_HOME%\bin ekliyoruz.
  ### Pyspark setup
  https://spark.apache.org/downloads.html
  1. Opening C:\spark file in PC
  2. Daha sonra conf klasörü içerisindeki log4j.properties.template dosyasının .template uzantısını siliyoruz ve herhangi bir text editör ile açıp log4j.rootCategory=INFO olan satırı   log4j.rootCategory=ERROR olarak değiştirip kaydediyoruz.
 3. Windows’un ortam değişkenlerine gelip (Denetim Masası ->Sistem ve Güvenlik -> Sistem -> Gelişmiş Sistem Ayarları -> Ortam Değişkenleri) yeni değişken oluşturuyoruz. Değişken adını: “SPARK_HOME” değerini: “C:\spark” olarak tanımlıyoruz.
Yine ortam değişkenlerinde Path’i seçip düzenle diyoruz ve Path’e %SPARK_HOME%\bin ekliyoruz.
  ### Hadoop Setup
  1. You must download "winutils.exe" for Hadoop
  2. Opening C:\spark file in PC Copy winutils.exe to Hadoop File
  3. C diskine gidip C:\tmp\hive dizinini oluşturuyoruz.
  Komut satırını yönetici olarak çalıştır seçeneği ile açıyoruz. winutils.exe’nin bulunduğu dizinde komutu yazıyoruz: `chmod -R 777 C:\tmp\hive`
  Son olarak komut satırına `spark-shell` komutunu yazıp çalıştırıyoruz.
  ### Anaconda Download
  1. Eğer pc'nizde kurulu değilse https://www.anaconda.com/products/individual sitesi üzerinden anaconda kurulumu yapılır.
  2. Apache Spark'ı Jupyter Notebook üzerinden kullanabilmek için, (Denetim Masası ->Sistem ve Güvenlik -> Sistem -> Gelişmiş Sistem Ayarları -> Ortam Değişkenleri) alanından Path'i düzenleye basarak, C:\Users\EnesA\anaconda3 Anaconda ' nın kurulu olduğu dizin eklenir. C:\Users\EnesA\anaconda3\Scripts anacondanın kurulu olduğu alandaki Scripts klasörünün dizini eklenir.

Yukarıdaki bütün kurulumlar tamamlandıktan sonra çalışmaya başlanabilecektir.

# Task-1 Exploratory Data Analysis.ipynb

1. Öncelikle jupyter notebook üzerinden File>New Notebook diyerek yeni bir sayfa açılır.
2. `pwd` komutunu yazarak jupyter' in kaydetme yerini öğrenip, kullanacağımız dataları buraya yüklüyoruz. Bu çalışmada https://grouplens.org/datasets/movielens/25m/ linki üzerinden "MovieLens 25M movie ratings" datası indirilir. İndirilen .csv uzantılı dosyaları **C:\Users\EnesA** dizinine koyulur.
3. Kullanacağımız Python kütüphanelerini import etmek için öncelikle `!pip install` komutunu kullanarak indirme işlemi yapılır. Example: `!pip install pyspark `
4. Kütüphaneler import edilir.
```import pyspark
import matplotlib.pyplot as plt
import pandas as pd
import seaborn as sns
from pyspark.sql import SparkSession 
```
5. Spark üzerinden bir session başlatmak için komut çalıştırılır. Dataları kullanmak için, daha önce indirmiş olduğumuz movielens datalarından gerekenler çekilir.
ml-25m klasorunu çağırdığınızda hata alırsanız, Ortam değişkenlerindeki  Path kısmına **C:\Users\EnesA\ml-25m** dizini eklemeniz gerekebilir. **.show** komutu dataların gelip gelmediğini görmeyi sağlar.
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
6. Session üzerinden direk sql kullanırken dataları çekmek için tanımlanan "movies" "ratings" değişkenleri sql tablosu haline getirilir.Artık datalar üzerinden çalışmaya başlanabilmektedir.
```
movies.createOrReplaceTempView("Movies")
ratings.createOrReplaceTempView("Ratings")
```
## 1.Question - Write a SQL query to create a dataframe with including userid, movieid, genre and rating

1. Bu soruda 2 farklı şekilde data çekilebilir.
Ayrı ayrı 2 tablo üzerinden data istendiği için join yapılması gerekmektedir. İlk olarak inner join kullanılarak **Movies** ve **Ratings** tablolaro birbirine bağlanılabilir. Ya da 2. örnekteki gibi **where** alanında tablolar birbirlerine bağlanılabilir.
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
In this question, we convert timestamp to date for using.
```
spark.sql("""select count(*) Frequentusers,userId from Movies m 
inner join Ratings r on r.movieId=m.movieId 
where CAST(TO_DATE(FROM_UNIXTIME(r.timestamp))as date)>CAST(TO_DATE(FROM_UNIXTIME(r.timestamp))as date) - interval '1' week 
group by userId 
order by count(*) desc   limit 5""").show()
```
## 5.Question -Calculate average ratings for each genre, and plot average ratings of top 10 genres with descending order

If anyone use the seaborn library with this data, you have to convert to Pandas.
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
