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


-- 15 Business Problems & Solutions

--1. Count the number of Movies vs TV Shows

SELECT * FROM netflix_db;

SELECT
   type , COUNT(type) AS total_type
	FROM netflix_db
GROUP BY 1


-- 2. Find the most common rating for movies and TV shows

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
--3. List all movies released in a specific year (e.g., 2020)

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


--4. Find the top 5 countries with the most content on Netflix

SELECT
		UNNEST(STRING_TO_ARRAY(country,',')) AS most_content_country,
     COUNT(show_id) AS total_content
	FROM netflix_db
	WHERE country IS NOT NULL
	GROUP BY 1 
	ORDER BY 2 DESC
	LIMIT 5



	
--5. Identify the longest movie

SELECT title, duration, type
FROM netflix_db
WHERE type = 'Movie'
  AND duration ~ '^[0-9]+ min$'
ORDER BY CAST(split_part(duration, ' ', 1) AS INTEGER) DESC
LIMIT 5;




--6. Find content added in the last 5 years

SELECT * FROM netflix_db;

SELECT 
 		*
   	FROM netflix_db
	   WHERE TO_DATE(date_added,'Month-DD-YYYY') >= CURRENT_DATE - INTERVAL '5 years'

--7. Find all the movies/TV shows by director 'Rajiv Chilaka'!

SELECT *
	 FROM netflix_db
	WHERE director ILIKE '%Rajiv Chilaka%'


--8. List all TV shows with more than 5 seasons


SELECT *
FROM (
		SELECT title, duration, type
				FROM netflix_db
			WHERE type = 'TV Show'
				ORDER BY CAST(split_part(duration, ' ', 1) AS INTEGER) DESC
		)
		WHERE CAST(split_part(duration, ' ', 1) AS INTEGER) > 5
	
--9. Count the number of content items in each genre

SELECT
	UNNEST(STRING_TO_ARRAY(listed_in,',')) AS most_content_genre,
    COUNT(show_id) AS total_content
FROM netflix_db
	GROUP BY 1 
	ORDER BY 2 DESC


--10.Find each year and the average numbers of content release in India on netflix. 
--    return top 5 year with highest avg content release!

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

--- 11. List all movies that are documentaries

SELECT *
FROM netflix_db
WHERE 
	listed_in LIKE '%Documentaries%'


-- 12. Find all content without a director

SELECT *
    FROM netflix_db
	WHERE director IS NULL

-- 13. Find how many movies actor 'Salman Khan' appeared in last 10 years!


SELECT 
* FROM netflix_db
WHERE cast_ LIKE '%Salman Khan%'
	AND
	 release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;

-- 14. Find the top 10 actors who have appeared in the highest number of movies produced in India.
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

-- 15.Categorize the content based on the presence of the keywords 'kill' and 'violence' in 
-- the description field. Label content containing these keywords as 'Bad' and all other 
-- content as 'Good'. Count how many items fall into each category.

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
