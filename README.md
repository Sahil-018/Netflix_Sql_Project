# Netflix_Sql_Project Movies & TV Show Analysis using SQL

![Netflix logo](https://github.com/Sahil-018/Netflix_Sql_Project/blob/main/Netflix_logo.png)

## Objective
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

Objectives
Analyze the distribution of content types (movies vs TV shows).
Identify the most common ratings for movies and TV shows.
List and analyze content based on release years, countries, and durations.
Explore and categorize content based on specific criteria and keywords.
Dataset
The data for this project is sourced from the Kaggle dataset:

Dataset Link: https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download

---- NETFLIX DATABASE PROJECT 

```sql
CREATE TABLE netflix_db
(
  	  show_id   VARCHAR(20),
	  type	VARCHAR(20),
	  title	VARCHAR(150),
	  director	VARCHAR(250),
	  cast_	VARCHAR(800),
	  country	VARCHAR(150),
	  date_added	VARCHAR(25),
	  release_year	INT,
	  rating	VARCHAR(10),
	  duration	VARCHAR(15),
	  listed_in	VARCHAR(100),
	  description VARCHAR(300)
)
```
```sql
SELECT * FROM netflix_db;

SELECT 
	 COUNT(*) as total_content
FROM netflix_db;

SELECT 
	DISTINCT type
FROM netflix_db;

SELECT 
     DISTINCT release_year
	FROM netflix_db
ORDER BY 1;

SELECT *
FROM (
		SELECT 
		    DISTINCT director AS director_name ,
			COUNT(title) AS count_title
		FROM netflix_db
		GROUP BY 1
		ORDER BY 2 desc
)
WHERE count_title > 1;
```

-- 15 Business Problems & Solutions

### 1. Count the Number of Movies vs TV Shows

```sql
SELECT * FROM netflix_db;

SELECT
   type , COUNT(type) AS total_type
	FROM netflix_db
GROUP BY 1

```

**Objective:** Determine the distribution of content types on Netflix.

### 2. Find the Most Common Rating for Movies and TV Shows

```sql
SELECT 
      type , 
	  rating ,
	  common_rating 
	FROM
   (
		SELECT 
				type,
				rating , 
				COUNT(rating) AS common_rating , 
				RANK() OVER(PARTITION BY type ORDER BY COUNT(rating)  DESC ) AS ranking
			FROM netflix_db
			GROUP BY 1 , 2
      ) t
	  WHERE ranking = 1;
```

**Objective:** Identify the most frequently occurring rating for each type of content.

### 3. List All Movies Released in a Specific Year (e.g., 2020)

```sql
SELECT 
	*
FROM netflix_db
WHERE release_year = 2020 AND type = 'Movie'
ORDER BY 1, 2

SELECT 
	title , release_year
FROM netflix_db
WHERE release_year = 2020 AND type = 'TV Show'
ORDER BY 1, 2

```

**Objective:** Retrieve all movies released in a specific year.

### 4. Find the Top 5 Countries with the Most Content on Netflix

```sql
SELECT
		UNNEST(STRING_TO_ARRAY(country,',')) AS most_content_country,
     COUNT(show_id) AS total_content
	FROM netflix_db
	WHERE country IS NOT NULL
	GROUP BY 1 
	ORDER BY 2 DESC
	LIMIT 5

```

**Objective:** Identify the top 5 countries with the highest number of content items.

### 5. Identify the Longest Movie

```sql
SELECT title, duration, type
FROM netflix_db
WHERE type = 'Movie'
  AND duration ~ '^[0-9]+ min$'
ORDER BY CAST(split_part(duration, ' ', 1) AS INTEGER) DESC
LIMIT 5;
```

**Objective:** Find the movie with the longest duration.

### 6. Find Content Added in the Last 5 Years

```sql

SELECT * FROM netflix_db;
SELECT 
 		*
   	FROM netflix_db
	   WHERE TO_DATE(date_added,'Month-DD-YYYY') >= CURRENT_DATE - INTERVAL '5 years'

```

**Objective:** Retrieve content added to Netflix in the last 5 years.

### 7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'

```sql

SELECT *
	 FROM netflix_db
	WHERE director ILIKE '%Rajiv Chilaka%'
```

**Objective:** List all content directed by 'Rajiv Chilaka'.

### 8. List All TV Shows with More Than 5 Seasons

```sql
SELECT *
FROM (
		SELECT title, duration, type
				FROM netflix_db
			WHERE type = 'TV Show'
				ORDER BY CAST(split_part(duration, ' ', 1) AS INTEGER) DESC
		)
		WHERE CAST(split_part(duration, ' ', 1) AS INTEGER) > 5
```

**Objective:** Identify TV shows with more than 5 seasons.

### 9. Count the Number of Content Items in Each Genre

```sql
SELECT
	UNNEST(STRING_TO_ARRAY(listed_in,',')) AS most_content_genre,
    COUNT(show_id) AS total_content
FROM netflix_db
	GROUP BY 1 
	ORDER BY 2 DESC
```

**Objective:** Count the number of content items in each genre.

### 10.Find each year and the average numbers of content release in India on netflix. 
return top 5 year with highest avg content release!

```sql
SELECT 
     EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD,YYYY')) AS year,
	 COUNT(*) AS yearly_content,

     ROUND(
	     COUNT(*)::numeric /(SELECT
					    COUNT(*) FROM netflix_db
						WHERE country = 'India')::numeric * 100 , 2)
						AS avg_content_year
	 FROM netflix_db
	WHERE country = 'India'
   GROUP BY 1
```

**Objective:** Calculate and rank years by the average number of content releases by India.

### 11. List All Movies that are Documentaries

```sql
SELECT *
FROM netflix_db
WHERE 
	listed_in LIKE '%Documentaries%'

```

**Objective:** Retrieve all movies classified as documentaries.

### 12. Find All Content Without a Director

```sql
SELECT * 
FROM netflix
WHERE director IS NULL;
```

**Objective:** List content that does not have a director.

### 13. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years

```sql
SELECT 
* FROM netflix_db
WHERE cast_ LIKE '%Salman Khan%'
	AND
	 release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```

**Objective:** Count the number of movies featuring 'Salman Khan' in the last 10 years.

### 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India

```sql
SELECT
*
FROM
(
		SELECT 
		 UNNEST(STRING_TO_ARRAY(cast_,',')) as Cast_appeared,
		 COUNT(*) as total_content,
		 DENSE_RANK() OVER(ORDER BY COUNT(*) DESC) as total_actors
		   FROM netflix_db
		   WHERE country ILIKE '%India'
			GROUP BY 1
			ORDER BY 2 DESC
)
WHERE 
total_actors <= 10;
```

**Objective:** Identify the top 10 actors with the most appearances in Indian-produced movies.

### 15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords

```sql
WITH CTE_content
AS
     (		SELECT 
		   	*,
			   CASE 
			   	WHEN 
				   description ILIKE '%kill%' OR 
				   description ILIKE '%violence%' THEN 'Bad_Content'
				   ELSE
				   'Good Content'
				 END content_category
		FROM netflix_db
    )

SELECT 
   content_category ,
   COUNT(*) AS total_content
FROM CTE_content
GROUP BY 1
```

**Objective:** Categorize content as 'Bad' if it contains 'kill' or 'violence' and 'Good' otherwise. Count the number of items in each category.

## Findings and Conclusion

- **Content Distribution:** The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
- **Common Ratings:** Insights into the most common ratings provide an understanding of the content's target audience.
- **Geographical Insights:** The top countries and the average content releases by India highlight regional content distribution.
- **Content Categorization:** Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.





Thank you for your support, and I look forward to connecting with you!
GROUP BY 1
