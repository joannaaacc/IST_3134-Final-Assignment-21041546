## 🛠️Full Execution Guide (Hadoop MapReduce Approach)
As an alternative to the MapReduce pipeline, we implemented the same logic using Apache Spark’s DataFrame API. This approach required fewer lines of code and provided in-memory processing advantages.

### 1️⃣ Setup PySpark
```bash
pyspark #Switch PySpark Shell

```

### 2️⃣ Import Required Library and Spark Session
```bash
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, when, count, avg, round as spark_round

spark = (SparkSession.builder
         .appName("MovieRatings-NonMR")
         .getOrCreate())
```
### 3️⃣ Load datasets from HDFS
```bash
ratings = (spark.read.csv("hdfs:///user/hadoop/moviedata/ratings.csv",
                          header=True, inferSchema=True))
# columns: userId,movieId,rating,timestamp

movies  = (spark.read.csv("hdfs:///user/hadoop/moviedata/movies.csv",
                          header=True, inferSchema=True))
# columns: movieId,title,genres
```

### 4️⃣ Aggregate: average rating + count per movie
```bash
ratings_clean = ratings 

agg = (ratings_clean
       .groupBy("movieId")
       .agg(
           spark_round(avg("rating"), 2).alias("avg_rating"),
           count("*").alias("rating_count")
       ))

```
### 5️⃣ Apply minimum rating count
Top‑10 movies, sorted by average rating
```bash
MIN_RATINGS = 50  
agg_f = agg.filter(col("rating_count") >= MIN_RATINGS)
```

### 6️⃣ Join with movie titles
```bash
final_df = (agg_f.join(movies, on="movieId", how="inner")
                 .select("movieId", "title", "avg_rating", "rating_count"))
```

### 7️⃣ Sort + take Top‑10
```bash
top10 = (final_df
         .orderBy(col("avg_rating").desc(),
                  col("rating_count").desc(),
                  col("title").asc())
         .limit(10))
```

### 8️⃣ Show results
```bash
top10.show(truncate=False)
```

### 9️⃣ Sample Results
```python
+-------+--------------------------------+----------+------------+              
|movieId|title                           |avg_rating|rating_count|
+-------+--------------------------------+----------+------------+
|171011 |Planet Earth II (2016)          |4.45      |1956        |
|159817 |Planet Earth (2006)             |4.44      |2948        |
|170705 |Band of Brothers (2001)         |4.43      |2811        |
|318    |Shawshank Redemption, The (1994)|4.4       |102929      |
|171495 |Cosmos                          |4.33      |615         |
|858    |Godfather, The (1972)           |4.32      |66440       |
|202439 |Parasite (2019)                 |4.31      |11670       |
|179135 |Blue Planet II (2017)           |4.3       |1163        |
|198185 |Twin Peaks (1989)               |4.3       |1140        |
|220528 |Twelve Angry Men (1954)         |4.29      |449         |
+-------+--------------------------------+----------+------------+
```
