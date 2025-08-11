# 🎥MovieLens Movie Average Ratings: A Comparison of Hadoop MapReduce and Apache Spark Approaches

This is the Final Project for IST3134 Big Data Analysis, completed by Chan Pui Ying Joanna (21041546) and Gan Yi Jean (22041321). 
The project uses the MovieLens 32M, which provide millions of movie ratings for thousands of movies. 
The analysis is conducted using two approaches: a Hadoop MapReduce pipeline (via Hadoop Streaming on AWS EC2) and a non-MapReduce method using Apache Spark. 
The objective is to compare both methods in terms of implementation complexity, scalability, and output while processing millions of movie ratings.


---

## 📁 Files in this Repo

- `README.md`: Project overview
- `MapReduce Approach.md`: Full execution guide and explanation of the Hadoop MapReduce implementation
- `Non-MapReduce Approach.md`: Full execution guide and explanation of the Apache Spark (non-MapReduce) implementation

---

## ⛁ Dataset

This project uses movies.csv and ratings.csv sourced from the MovieLens website:

Dataset: [MovieLens 32M](https://grouplens.org/datasets/movielens/32m/)

File 1: movies.csv
- **File Name**: movies.csv

- **File Size**: 4.14 MB

- **Total Records**: 87,586 movies

File 2: ratings.csv
- **File 1 Name**: ratings.csv
  
- **File Size**: 856.52 MB
  
- **Total Records**: 32 million ratings


No preprocessing was required as both datasets are already clean and analysis-ready from the MovieLens website. For this project, we focus on movie ratings analysis.

Due to GitHub's 25MB file size limit, the extracted files used in this project are hosted externally on Dropbox:
[movies.csv](https://www.dropbox.com/scl/fi/4fx41j8ne773yhcz8exhe/movies.csv?rlkey=8k5ovbp7eaxf240d29w45c4vv&st=2a55u5j0&dl=1)
[ratings.csv](https://www.dropbox.com/scl/fi/krkdpnw9nrb3s5qyf1h0p/ratings.csv?rlkey=349sogyz8cvvxebks0wvzsoqt&st=s1mnfnyp&dl=1)

---
## Full Report
[📄 View the PDF Report](https://github.com/joannaaacc/IST_3134-Final-Assignment-21041546/blob/main/IST3134%20%20%E2%80%93%20Group%20Assignment%20April%202025.pdf)
---

## 👥 Team Member

| Name                    | Student ID  |
|-------------------------|-------------|
| Chan Pui Ying Joanna    | 21041546    |
| Gan Yi Jean             | 22041321    |

---

