# 🎬 IMDb Projects (SQL + BI)
This project analyzes a movie dataset using **PostgreSQL** for data processing and **Power BI** for visualization. It highlights trends in ratings, revenues, genres, directors, and more.
## 📁 Project Structure

| Folder        | Description                                  |
|---------------|----------------------------------------------|
| `Datasets/`   | Contains raw or cleaned CSV files            |
| `SQL/`        | SQL scripts for data analysis and views      |
| `PowerBI/`    | Power BI dashboard (.pbix + screenshots)     |
| `docs/`       | Data dictionary and additional notes         |

---

## 🎯 Objectives

- Explore patterns in ratings, genres, gross revenue
- Identify top-performing directors, actors, genres
- Visualize trends in the film industry over time
- Build reusable SQL queries for insights
- Perform windows functions for complex query.


---

## 🗃️ Dataset Schema

Table: `movies`

| Column         | Description                         |
|----------------|-------------------------------------|
| Series_Title   | Movie title                         |
| Released_Year  | Year the movie was released         |
| Certificate    | Movie certificate (e.g., PG, R)     |
| Runtime        | Runtime in hours                    |
| Genre          | Movie genre (e.g., Action,Drama)    |
| IMDB_Rating    | IMDb rating                         |
| Meta_score     | Metacritic score                    |
| Director       | Movie director names                |
| Star1–Star4    | Lead actors names                   |
| No_of_Votes    | Number of IMDb votes                |
| Gross          | Box office gross revenue (USD)      |

---

## 🧮 SQL Analysis




-- Create Database & Tables
```sql
DROP DATABASE IF EXISTS imdb;
CREATE DATABASE imdb;
```
```sql
DROP TABLE IF EXISTS movies;
CREATE TABLE movies(
  Series_Title TEXT,
  Released_Year TEXT,
  Certificate TEXT,
  Runtime FLOAT,
  Genre TEXT,
  IMDB_Rating FLOAT,
  Meta_score INT,
  Director TEXT,
  Star1 TEXT,
  Star2 TEXT,
  Star3 TEXT,
  Star4 TEXT,
  No_of_Votes INT,
  Gross FLOAT
);
```

------------------------------------------------------------------------------------------------------------------------
-- Business Queries
------------------------------------------------------------------------------------------------------------------------
```sql
-- 1. Preview First 10 Rows >>

SELECT * FROM movies
LIMIT 10;

-- 2. Get Table Schema (Column Names & Data Types)

SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name = 'movies';

-- 3. Count Total Rows

SELECT COUNT(*) FROM movies;
SELECT COUNT(*) AS TotalMovies FROM movies;

-- 4. Count of NULLs per COLUMN

SELECT 
  COUNT(*) FILTER (WHERE Series_Title IS NULL) AS null_series_title,
  COUNT(*) FILTER (WHERE Released_Year IS NULL) AS null_released_year,
  COUNT(*) FILTER (WHERE Certificate IS NULL) AS null_certificate,
  COUNT(*) FILTER (WHERE Runtime IS NULL) AS null_runtime,
  COUNT(*) FILTER (WHERE Genre IS NULL) AS null_genre,
  COUNT(*) FILTER (WHERE IMDB_Rating IS NULL) AS null_imdb_rating,
  COUNT(*) FILTER (WHERE Meta_score IS NULL) AS null_meta_score,
  COUNT(*) FILTER (WHERE Director IS NULL) AS null_director,
  COUNT(*) FILTER (WHERE Star1 IS NULL) AS null_star1,
  COUNT(*) FILTER (WHERE No_of_Votes IS NULL) AS null_votes,
  COUNT(*) FILTER (WHERE Gross IS NULL) AS null_gross
FROM movies;

-- 5. Summary Statistics for Numeric Columns

SELECT
  MIN(Runtime) AS min_runtime,
  MAX(Runtime) AS max_runtime,
  AVG(Runtime) AS avg_runtime,
  MIN(IMDB_Rating) AS min_rating,
  MAX(IMDB_Rating) AS max_rating,
  AVG(IMDB_Rating) AS avg_rating,
  MIN(Meta_score) AS min_meta,
  MAX(Meta_score) AS max_meta,
  AVG(Meta_score) AS avg_meta,
  MIN(Gross) AS min_gross,
  MAX(Gross) AS max_gross,
  AVG(Gross) AS avg_gross
FROM movies;

-- 6. Distribution of Movies by Certificate and YEAR

SELECT Certificate, COUNT(*) AS count
FROM movies
GROUP BY Certificate
ORDER BY count DESC;

SELECT Released_Year, COUNT(*) AS movie_count
FROM movies
GROUP BY Released_Year
ORDER BY Released_Year;

-- 7. Top 10 Most Frequent Genres / UNIQUE Genres

SELECT Genre, COUNT(*) AS genre_count
FROM movies
GROUP BY Genre
ORDER BY genre_count DESC
LIMIT 10;


-- 8. Count how many movies each director has directed

SELECT Director, COUNT(*) AS movie_count
FROM movies
GROUP BY Director
ORDER BY movie_count DESC
LIMIT 10;

-- 9. Most Frequent Lead Actor (Star1)

SELECT Star1, COUNT(*) AS count
FROM movies
GROUP BY Star1
ORDER BY count DESC
LIMIT 10;

-- 10. Correlation Between IMDB Rating and Meta Score

SELECT CORR(IMDB_Rating, Meta_score) AS correlation
FROM movies;

-- 11. Outliers in Gross Revenue (Top and Bottom)

SELECT Series_Title, Gross
FROM movies
ORDER BY Gross DESC NULLS LAST
LIMIT 5;

SELECT Series_Title, Gross
FROM movies
ORDER BY Gross ASC NULLS LAST
LIMIT 5;

-- 12. Distribution Buckets of IMDB Rating

SELECT 
  FLOOR(IMDB_Rating) AS rating_bucket,
  COUNT(*) AS count
FROM movies
WHERE IMDB_Rating IS NOT NULL
GROUP BY FLOOR(IMDB_Rating)
ORDER BY rating_bucket;

-- 13. Get the top 10 movies by IMDB rating

SELECT Series_Title, IMDB_Rating
FROM movies
ORDER BY IMDB_Rating DESC
LIMIT 10;

-- 14. Most Common Movie Runtimes

SELECT Runtime as Runtime_hours, COUNT(*) AS count
FROM movies
GROUP BY Runtime
ORDER BY count DESC
LIMIT 10;

-- 15. Duplicate Titles Check

SELECT Series_Title, COUNT(*) AS title_count
FROM movies
GROUP BY Series_Title
HAVING COUNT(*) > 1;

-- 16. Find the highest Grossing & Lowest Grossing movies.

SELECT Series_Title, Gross
FROM movies
ORDER BY Gross DESC
LIMIT 1;
SELECT Series_Title, Gross
FROM movies
ORDER BY Gross ASC
LIMIT 1;

-- 17. List all movies with a meta score below 50

SELECT Series_Title, Meta_score
FROM movies
WHERE Meta_score < 50
ORDER BY 2 ASC;

-- 18. List all movies with certificate 'PG-13'

SELECT Series_Title AS Movies_Name
FROM movies
WHERE Certificate = 'PG-13';
```
------------------------------------------------------------------------------------------------------------------------
--  Intermediate Queries --
------------------------------------------------------------------------------------------------------------------------
```sql
-- 1. Find top 5 directors by average IMDB rating

SELECT Director, AVG(IMDB_Rating) AS avg_rating
FROM movies
GROUP BY Director
ORDER BY avg_rating DESC
LIMIT 5;

-- 2. Find year-wise average gross

SELECT Released_Year, AVG(Gross) AS avg_gross
FROM movies
GROUP BY Released_Year
ORDER BY Released_Year;

-- 3. Calculate the percentage of movies with a Meta_score above 70

SELECT
  (COUNT(*) FILTER (WHERE Meta_score > 70) * 100.0) / COUNT(*) AS percentage_above_70
FROM movies;

-- 4. List all movies with IMDB rating above average

SELECT Series_Title, IMDB_Rating
FROM movies
WHERE IMDB_Rating > (SELECT AVG(IMDB_Rating) FROM movies);

-- 5. Top 3 genres with highest average IMDB rating

SELECT Genre, AVG(IMDB_Rating) AS avg_rating
FROM movies
GROUP BY Genre
ORDER BY avg_rating DESC
LIMIT 3;

-- 6. Find total gross revenue generated by each genre

SELECT Genre, SUM(Gross) AS total_gross
FROM movies
GROUP BY Genre
ORDER BY total_gross DESC;

-- 7. Find directors with more than 5 movies and average rating > 8

SELECT Director, COUNT(*) AS movie_count, AVG(IMDB_Rating) AS avg_rating
FROM movies
GROUP BY Director
HAVING COUNT(*) > 5 AND AVG(IMDB_Rating) > 8;
```

------------------------------------------------------------------------------------------------------------------------
-- Advance Queries
------------------------------------------------------------------------------------------------------------------------
```sql
-- 1. Create a star-cast frequency leaderboard (count how many times each actor appears across all star columns)

SELECT actor, COUNT(*) AS appearances
FROM (
  SELECT Star1 AS actor FROM movies
  UNION ALL
  SELECT Star2 FROM movies
  UNION ALL
  SELECT Star3 FROM movies
  UNION ALL
  SELECT Star4 FROM movies
) AS all_stars
GROUP BY actor
ORDER BY appearances DESC;

-- 2. Rank Movies by IMDB Rating

SELECT
  Series_Title,
  IMDB_Rating,
  RANK() OVER (ORDER BY IMDB_Rating DESC) AS imdb_rank
FROM movies;

-- 3. Dense Rank of Movies Within Each Genre by Gross

SELECT
  Genre,
  Series_Title,
  Gross,
  DENSE_RANK() OVER (PARTITION BY Genre ORDER BY Gross DESC) AS rank_within_genre
FROM movies;

-- 4. Running Total of Votes by Year

SELECT
  Released_Year,
  Series_Title,
  No_of_Votes,
  SUM(No_of_Votes) OVER (PARTITION BY Released_Year ORDER BY Series_Title) AS running_votes
FROM movies; 

-- 5. Lag/Lead to Compare Ratings

SELECT
  Series_Title,
  IMDB_Rating,
  LAG(IMDB_Rating) OVER (ORDER BY IMDB_Rating DESC) AS previous_rating,
  LEAD(IMDB_Rating) OVER (ORDER BY IMDB_Rating DESC) AS next_rating
FROM movies;



-- 6. Cumulative Gross, Avg Gross, Avg Votes Using Window Functions

SELECT
  Series_Title,
  Genre,
  IMDB_Rating,
  Gross,
  No_of_Votes,

  -- Running Total of Gross
  SUM(Gross) OVER (ORDER BY IMDB_Rating DESC) AS cumulative_gross,

  -- Rolling Average Gross
  AVG(Gross) OVER (ORDER BY IMDB_Rating DESC) AS avg_gross_so_far,

  -- Rolling Average Number of Votes
  ROUND(AVG(No_of_Votes) OVER (ORDER BY IMDB_Rating DESC),2) AS avg_votes_so_far

FROM movies
WHERE Gross IS NOT NULL AND No_of_Votes IS NOT NULL
ORDER BY IMDB_Rating DESC;


-- 7. Query with PRECEDING Window Frame

SELECT
  Series_Title,
  Genre,
  IMDB_Rating,
  Gross,
  No_of_Votes,

  -- Cumulative Gross from first row to current 
  SUM(Gross) OVER (
    ORDER BY IMDB_Rating DESC
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS cumulative_gross,

  -- Average Gross from first row to current
  AVG(Gross) OVER (
    ORDER BY IMDB_Rating DESC
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS avg_gross_so_far,

  -- Average Votes from first row to current
  ROUND(AVG(No_of_Votes) OVER (
    ORDER BY IMDB_Rating DESC
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ),2) AS avg_votes_so_far

FROM movies
WHERE Gross IS NOT NULL AND No_of_Votes IS NOT NULL
ORDER BY IMDB_Rating DESC;


-- 8. Cumulative Sum & Avg Over Top 5 Movies by IMDB Rating

WITH top_5_movies AS (
  SELECT *
  FROM movies
  WHERE Gross IS NOT NULL
  ORDER BY IMDB_Rating DESC
  LIMIT 5
)
SELECT
  Series_Title,
  IMDB_Rating,
  Gross,
  
  -- Cumulative gross for top 5 movies
  SUM(Gross) OVER (ORDER BY IMDB_Rating DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cumulative_gross_top_5,
  
  -- Average gross for top 5 movies
  AVG(Gross) OVER (
    ORDER BY IMDB_Rating DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS avg_gross_top_5

FROM top_5_movies
ORDER BY IMDB_Rating DESC;

```
------------------------------------------------------------------------------------------------------------------------
--Bonus--
------------------------------------------------------------------------------------------------------------------------
```sql
-- Creating View for this table

CREATE OR REPLACE VIEW movies_easy_view AS
SELECT
  Series_Title,
  Released_Year,
  Certificate,
  Runtime,
  Genre,
  IMDB_Rating,
  Meta_score,
  Director,
  Star1,
  Star2,
  Star3,
  Star4,
  No_of_Votes,
  Gross
FROM movies;

-- To call this VIEW
SELECT * FROM movies_easy_view;
SELECT AVG(IMDB_Rating) FROM movies_easy_view;
SELECT series_title,IMDB_Rating,Meta_score FROM movies_easy_view;

--------------------------------------------------------------------------------------------------------------------
```



## 📊 Power BI Dashboard
The Power BI report includes:

💡 KPI Cards: Avg IMDb rating, total gross, total movies, Total Votes, Meta Score.

📈 Line Chart: Ratings over time.

📊 Bar Charts: Top directors, genres, certificates.

🎥 Filterable Tables: Explore by actor, year, rating, genre.

## 🎬 Final Report Insights – IMDb(movies Table)
### 1. ⭐ IMDb Ratings Insights
Average IMDb rating across all movies: `7.94 `

* Top 10 highest-rated movies lists -
`The Godfather 9.2,
The Shawshank Redemption 9.1,
The Dark Knight 9,
The Godfather: Part II 9,
12 Angry Men 9,
The Lord of the Rings: The Return of the King 8.9 ,
Pulp Fiction 8.9,
Schindler's List 8.9,
Inception 8.8,
Forrest Gump 8.8`

* Genres with the highest average ratings: `Animation Drama War 8.5, Drama Musical 8.4, Action Sci-Fi 8.4, Drama Mystery War 8.35 ` and `Western 8.35` consistently show above-average ratings.

* Directors with the highest average ratings (with at least 3 movies): `Frank Darabont,
Irvin Kershner, Lana Wachowski` lead the list.

* Top 5 Highest rated meta score movies- `Rear Window, Soorarai Pottru, The Godfather, Casablanca, Lawrence of Arabia`
* 
### 2. 💰 Revenue & Gross Insights
* Average gross revenue: $680M (68016083.83)

* Top 5 highest-grossing movies earned over $678 million.

* Which movie generated the highest revenue is `Star Wars: Episode VII - The Force Awakens (936M)`

* A strong positive correlation observed between IMDb rating and gross revenue, especially among highly rated films.

### 3. 📆 Time Trends
* Number of movies released per year shows a noticeable increase between 2000–2020.

* Recent years (post-2010) contributed significantly to box office revenue, suggesting a growing film industry.

* Average movie runtime has slightly increased over the decades.

### 4. 🧑‍💼 Director & Actor Insights
* Most active directors in the dataset `Alfred Hitchcock(14), Steven Spielberg(13), and Hayao Miyazaki(11)`

* Top 5 actors by appearances across all movies:
`Robert De Niro (17),
Tom Hanks (14),
Al Pacino (13),
Brad Pitt (12),
Clint Eastwood (12)`

* These actors also often appear in high-grossing and highly rated films.

### 5. 🎭 Genre & Certificate Insights
* Most popular genres by votes: `Drama, Action Crime Drama, Action Adventure Sci-Fi`

* Highest-rated genres: ` Crime Drama(9.2), Drama(9.1), Crime Drama(9.1)`

* `U-rated` films dominate the dataset (259 Movies).

* PG-16 films earn more on average, likely due to broader audience appeal.

### 6. 🕒 Runtime & Votes Insights
* Average runtime: ~122 minutes.

* Top 5 longest movies run over ~228 minutes.

* Most voted movies (on IMDb) include blockbuster hits with high ratings and gross earnings.
  `Star Wars: Episode VII - The Force Awakens (7.9 - 936M)`

* High vote count often correlates with both box office success and IMDb popularity.

### 7. 🧠 Key Business Takeaways
🎯 Well-rated movies (IMDb > 7.0) tend to earn more gross revenue and attract more votes.

🎬 `Action Adventure Sci-Fi` genres dominate in terms of revenue but `Drama Musical` outperform in average rating.

👥 Actor-director collaborations ` [ Daisy Ridley, John Boyega, Oscar Isaac, Domhnall Gleeson - J.J.Abrams]` appear in top-rated/grossing movie.

🏆 For investment in movies, focus on: Directors with proven success Genres with high gross potential Runtime and certificate balance (PG-16 performs well).








