# Netflix Movies and TV Shows Data Analysis using SQL

## Overview

This project focuses on an in-depth analysis of Netflix's movie and TV show dataset using SQL. The primary aim is to uncover meaningful insights and address a variety of business questions based on the data.
This README outlines the project's objectives, key business challenges, proposed solutions, insights, and conclusions

## Objectives

* Analyze the distribution of content types (movies vs TV shows).
* Identify the most common ratings for movies and TV shows.
* List and analyze content based on release years, countries, and durations.
* Explore and categorize content based on specific criteria and keywords.

## Schema

```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```

## Business Problems and Solutions

### Most Popular Show Type in the Dataset

```sql
SELECT show_type,
       COUNT(*) AS count
FROM netflix
GROUP BY show_type
ORDER BY count DESC;
```

### Top Countries by Number of Shows

```sql
SELECT country, COUNT(*) AS show_count
FROM netflix
WHERE country IS NOT NULL
GROUP BY country
ORDER BY content_count DESC
LIMIT 10;
```

### Trend of Show Release Over the Years

```sql
SELECT release_year, COUNT(*) AS content_count
FROM netflix
GROUP BY release_year
ORDER BY release_year;
```

### Find the Most Common Rating for Movies and TV Shows

```sql
WITH CTE AS
(SELECT show_type, rating, COUNT(*),
DENSE_RANK () OVER(PARTITION BY show_type ORDER BY COUNT(*) DESC)
FROM netflix
GROUP BY 1, 2)

SELECT show_type, rating AS most_frequent_rating
FROM CTE
WHERE DENSE_RANK = 1
```

### Identify the Longest Duration for Movie and TV Show

```sql
With CTE AS
(SELECT show_type, title, MAX(SPLIT_PART(duration, ' ', 1)::INT) AS duration,
DENSE_RANK () OVER (PARTITION BY show_type ORDER BY MAX(split_part(duration, ' ', 1)::INT) DESC)
FROM netflix
WHERE duration IS NOT NULL
GROUP BY show_type, title)

SELECT show_type, title
FROM CTE
WHERE DENSE_RANK = 1;
```

## Top Directors with Most Content

```sql
SELECT director, COUNT(*) AS count
FROM netflix
WHERE director IS NOT NULL
GROUP BY director
ORDER BY count DESC
LIMIT 10;
```

## Find Each Year and the Average Numbers of Content Release in India on Netflix

```sql
SELECT country, release_year,
COUNT(show_id) AS total_release,
ROUND(COUNT(show_id)::numeric /
        (SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100, 2) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;
```

### Find Content Added in the Last 5 Years

```sql
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```

### Find All the Shows Featuring Robert Downey Jr

```sql
SELECT *
FROM netflix
WHERE casts like '%Robert Downey Jr.%';
```

### Find All the Movies Released After 2015 in the Action Genre

```sql
SELECT title, release_year, listed_in
FROM netflix
WHERE show_type = 'Movie' 
  AND release_year > 2015
  AND listed_in LIKE '%Action%';
```

### Find the Actors hho have Appeared in the Most Shows/Movies

```sql
SELECT UNNEST(STRING_TO_ARRAY(casts,',')) AS actors, COUNT(*) AS appearance_count
FROM netflix
GROUP BY actors
ORDER BY appearance_count DESC
LIMIT 10;
```

### List All Movies that are Thriller

```sql
SELECT *
FROM netflix
WHERE listed_in like '%Thriller%';
```

### Find All Content Without a Director

```sql
SELECT *
FROM netflix
WHERE director is null;
```

### Find How Many Movies Actor 'Vikrant Massey' Appeared in the Last 10 Years

```sql
SELECT *
FROM netflix
WHERE casts like '%Vikrant Massey%'
and release_year >= extract(year from CURRENT_DATE) - 10;
```

### List All TV Shows with More Than 5 Seasons

```sql
SELECT *
FROM netflix
WHERE show_type = 'TV Show' AND title is not null
AND SPLIT_PART(duration, ' ', 1):: int > 5;
```
