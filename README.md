# SQL-Based Analytical Study of ZEPTO Products


<img width="1280" height="720" alt="LOGO" src="https://github.com/user-attachments/assets/a6b09ce0-1f04-4522-ba28-15f8a363bc81" />



### Overview

This project presents an in-depth SQL-driven analysis of ZEPTO’s product catalog dataset. The objective is to extract meaningful insights into product pricing, stock availability, discount patterns, inventory management, and category-wise performance. By leveraging advanced SQL queries in pgAdmin, the project addresses real business problems and provides actionable insights for e-commerce operations.

The analysis explores various dimensions such as pricing trends, discount analysis, stock availability, inventory valuation, product categorization, and revenue contributions.

### Objectives

Examine product distribution and availability across categories.

Identify high-value and premium products and analyze discounts.

Explore inventory weight, stock status, and slow-moving items.

Calculate category-wise revenue, inventory value, and revenue contribution percentages.

Detect pricing anomalies and underpriced/overpriced products.

Categorize products based on weight, price, or discount for better decision-making.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

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

### 1.Find the total number of titles added each year.

```sql
SELECT 
    EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) AS year,
    COUNT(*) AS total_titles
FROM netflix
GROUP BY 1
ORDER BY year;

```
**Objective:** To understand annual content growth on Netflix and identify trends in how content additions have increased or decreased over time.

### 2.Identify the month with the highest content additions
```sql
SELECT 
    TO_CHAR(TO_DATE(date_added, 'Month DD, YYYY'), 'Month') AS month,
    COUNT(*) AS total_added
FROM netflix
GROUP BY month
ORDER BY total_added DESC;

```
**Objective:**  To analyze seasonal content upload patterns and determine which months see the most activity on the platform.

### 3.Top 5 directors with the most content
```sql
SELECT 
    director,
    COUNT(*) AS total_content
FROM netflix
WHERE director IS NOT NULL AND director <> ''
GROUP BY director
ORDER BY total_content DESC
LIMIT 5;
```
**Objective:**  To identify the most influential and frequently featured directors, revealing patterns in Netflix’s collaboration and content acquisition strategy.

### 4.Actors who appear together most often
```sql
WITH actor_list AS (
    SELECT 
        show_id,
        UNNEST(STRING_TO_ARRAY(casts, ', ')) AS actor
    FROM netflix
    WHERE casts IS NOT NULL AND casts <> ''
),
actor_pairs AS (
    SELECT 
        a1.actor AS actor1,
        a2.actor AS actor2
    FROM actor_list a1
    JOIN actor_list a2
       ON a1.show_id = a2.show_id
      AND a1.actor < a2.actor   -- avoid duplicates (A,B) and (B,A)
)
SELECT 
    actor1,
    actor2,
    COUNT(*) AS appearances_together
FROM actor_pairs
GROUP BY actor1, actor2
ORDER BY appearances_together DESC
LIMIT 10;

)
```
**Objective:**  To discover common actor pairings and frequent collaborations, which can help understand casting trends and viewer-favorite combination

### 5.List all titles containing the word "Love
```sql
SELECT *
FROM netflix
WHERE title ILIKE '%love%';
```
**Objective:** To filter and analyze content based on keywords, helping detect theme-based trends or the popularity of specific subjects (such as romance).

### 6. Count the Number of Movies vs TV Shows

```sql
SELECT 
    type,
    COUNT(*)
FROM netflix
GROUP BY 1;
```

**Objective:** Determine the distribution of content types on Netflix.

### 7. Find the Most Common Rating for Movies and TV Shows

```sql
WITH RatingCounts AS (
    SELECT 
        type,
        rating,
        COUNT(*) AS rating_count
    FROM netflix
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT 
        type,
        rating,
        rating_count,
        RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCounts
)
SELECT 
    type,
    rating AS most_frequent_rating
FROM RankedRatings
WHERE rank = 1;
```

**Objective:** Identify the most frequently occurring rating for each type of content.

### 8. List All Movies Released in a Specific Year (e.g., 2020)

```sql
SELECT * 
FROM netflix
WHERE release_year = 2020;
```

**Objective:** Retrieve all movies released in a specific year.

### 9. Find the Top 5 Countries with the Most Content on Netflix

```sql
SELECT * 
FROM
(
    SELECT 
        UNNEST(STRING_TO_ARRAY(country, ',')) AS country,
        COUNT(*) AS total_content
    FROM netflix
    GROUP BY 1
) AS t1
WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5;
```

**Objective:** Identify the top 5 countries with the highest number of content items.

### 10. Identify the Longest Movie

```sql
SELECT 
    *
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC;
```

**Objective:** Find the movie with the longest duration.

### 11. Find Content Added in the Last 5 Years

```sql
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```

**Objective:** Retrieve content added to Netflix in the last 5 years.

### 12. Find All Movies/TV Shows by Director 'Rajiv Chilaka'

```sql
SELECT *
FROM (
    SELECT 
        *,
        UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name
    FROM netflix
) AS t
WHERE director_name = 'Rajiv Chilaka';
```

**Objective:** List all content directed by 'Rajiv Chilaka'.

### 13. List All TV Shows with More Than 5 Seasons

```sql
SELECT *
FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```

**Objective:** Identify TV shows with more than 5 seasons.

### 14. Count the Number of Content Items in Each Genre

```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre,
    COUNT(*) AS total_content
FROM netflix
GROUP BY 1;
```

**Objective:** Count the number of content items in each genre.

### 15.Find each year and the average numbers of content release in India on netflix. 
return top 5 year with highest avg content release!

```sql
SELECT 
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::numeric /
        (SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100, 2
    ) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;
```

**Objective:** Calculate and rank years by the average number of content releases by India.

### 16. List All Movies that are Documentaries

```sql
SELECT * 
FROM netflix
WHERE listed_in LIKE '%Documentaries';
```

**Objective:** Retrieve all movies classified as documentaries.

### 17. Find All Content Without a Director

```sql
SELECT * 
FROM netflix
WHERE director IS NULL;
```

**Objective:** List content that does not have a director.

### 18. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years

```sql
SELECT * 
FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```

**Objective:** Count the number of movies featuring 'Salman Khan' in the last 10 years.

### 19. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India

```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
    COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY actor
ORDER BY COUNT(*) DESC
LIMIT 10;
```

**Objective:** Identify the top 10 actors with the most appearances in Indian-produced movies.

### 20. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords

```sql
SELECT 
    category,
    COUNT(*) AS content_count
FROM (
    SELECT 
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY category;
```

**Objective:** Categorize content as 'Bad' if it contains 'kill' or 'violence' and 'Good' otherwise. Count the number of items in each category.

## Findings and Conclusion

- **Content Distribution:** The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
- **Common Ratings:** Insights into the most common ratings provide an understanding of the content's target audience.
- **Geographical Insights:** The top countries and the average content releases by India highlight regional content distribution.
- **Content Categorization:** Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.

