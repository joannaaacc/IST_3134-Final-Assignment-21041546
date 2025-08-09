## 🛠️Full Execution Guide (Hadoop MapReduce Approach)
This section outlines how we executed the MapReduce workflow on AWS using Hadoop Streaming.

### 1️⃣ Setup Hadoop
```bash
sudo su hadoop #Switch to hadoop user

start-all.sh #Start Hadoop
```

### 2️⃣ Data Preparation
We have two separate data files: movies.csv and ratings.csv downloaded from Dropbox originally from https://grouplens.org/datasets/movielens/ under the section of MovieLens 32M
```bash
cd ~
wget "https://www.dropbox.com/scl/fi/4fx41j8ne773yhcz8exhe/movies.csv?rlkey=8k5ovbp7eaxf240d29w45c4vv&st=kufbpr0c&dl=1" -O movies.csv
wget "https://www.dropbox.com/scl/fi/krkdpnw9nrb3s5qyf1h0p/ratings.csv?rlkey=349sogyz8cvvxebks0wvzsoqt&st=l2leeqrp&dl=1" -O ratings.csv
```
Then, we upload the files to HDFS
```bash
hdfs dfs -rm -r /user/hadoop/moviedata #Remove any previous moviedata folder, this is to make sure the second run time do not create duplicate folder. Hence it is okay to have error for the first time
hdfs dfs -mkdir -p /user/hadoop/moviedata #Create new folder name moviedata to store the two data files 
hdfs dfs -put -f ratings.csv movies.csv /user/hadoop/moviedata/ # Put the ratings.csv and movies.csv into moviesdata folder
hdfs dfs -ls /user/hadoop/moviedata #List them out to check if both data files are in the correct folder
```
### 3️⃣ Mapper and Reducer Scripts

#### mapper.py : emit (movieId, rating)
```bash
cat > mapper.py << 'PY'
#!/usr/bin/env python3
import sys, csv

reader = csv.reader(sys.stdin)
for row in reader:
    try:
        if row[0] == "userId":
            continue
        movie_id = row[1].strip()
        rating = float(row[2].strip())
        print(f"{movie_id}\t{rating}")
    except:
        continue
PY
```
#### reducer.py : join to titles, compute averages, keep top 10 (≥50 ratings)
```bash
cat > reducer.py << 'PY'
#!/usr/bin/env python3
import sys, csv, heapq

# Load movieId -> title mapping (the movies.csv is shipped via -files and symlinked here)
movie_titles = {}
with open("movies.csv", encoding="utf-8") as f:
    r = csv.reader(f); next(r, None)
    for row in r:
        movie_titles[row[0].strip()] = row[1].strip()

current = None; total = 0.0; count = 0
topk, K, THRESH = [], 10, 50

def consider(mid, s, c):
    if c < THRESH: return
    avg = s / c
    title = movie_titles.get(mid, "Unknown Title")
    heapq.heappush(topk, (avg, c, title))
    if len(topk) > K:
        heapq.heappop(topk)

for line in sys.stdin:
    try:
        mid, val = line.rstrip("\n").split("\t", 1)
        rating = float(val)
    except:
        continue
    if current is None: current = mid
    if mid != current:
        consider(current, total, count)
        current, total, count = mid, 0.0, 0
    total += rating; count += 1

if current is not None:
    consider(current, total, count)

for avg, c, title in sorted(topk, key=lambda x: (x[0], x[1]), reverse=True):
    print(f"{title}\t{avg:.2f}\t{c}")
PY
```
Scripts were made executable using:
```bash
chmod +x mapper.py reducer.py
```

### 4️⃣ Hadoop Streaming Job Execution
The MapReduce job was executed using the Hadoop Streaming JAR:
Ships mapper, reducer, and the HDFS copy of movies.csv into the task runtime.

```bash
#Remove old output in movielens
hdfs dfs -rm -r -skipTrash /user/hadoop/movielens 

#Run Job
hadoop jar /home/hadoop/hadoop-3.3.6/share/hadoop/tools/lib/hadoop-streaming-3.3.6.jar \
  -D mapred.reduce.tasks=1 \
  -files mapper.py,reducer.py,hdfs:///user/hadoop/moviedata/movies.csv#movies.csv \
  -mapper mapper.py \
  -reducer reducer.py \
  -input /user/hadoop/moviedata/ratings.csv \
  -output /user/hadoop/movielens

```
### 5️⃣ Output & Sorting
Top‑10 movies, sorted by average rating
```bash
hdfs dfs -cat /user/hadoop/movielens/part-00000
```

### 6️⃣ Sample Output
```bash
Planet Earth II (2016)  4.45        1956
Planet Earth (2006)     4.44        2948
Band of Brothers (2001) 4.43        2811
Shawshank Redemption, The (1994)        4.41        102929
Cosmos  4.33        615
Godfather, The (1972)   4.32        66440
Parasite (2019) 4.31        11670
Blue Planet II (2017)   4.30        1163
Twin Peaks (1989)       4.30        1140
Twelve Angry Men (1954) 4.29        449
```
