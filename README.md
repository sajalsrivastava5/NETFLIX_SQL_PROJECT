# NETFLIX SQL PROJECT: Movies and TV Shows Data Analysis Using SQL

![Netflix Logo](https://github.com/sajalsrivastava5/NETFLIX_SQL_PROJECT/raw/main/logo.png)

## Project Overview
This project involves analyzing a dataset of Netflix movies and TV shows containing **8,809 records** to extract key insights about genre, release patterns, content ratings, and other factors influencing Netflix's catalog. The dataset provides details on titles, genres, release dates, ratings, and more, offering a comprehensive view of the Netflix library over time.

## Objectives
- **Understand content distribution** across different genres, regions, and ratings.
- **Analyze release patterns** to identify trends over the years.
- **Compare performance metrics** across various content types, including movies and TV shows.

## Data Sources
The dataset for this project was obtained from [Kaggle's Netflix Movies and TV Shows dataset](https://www.kaggle.com/shivamb/netflix-shows).

## Tools and Technologies
- **SQL**: MySQL Server was used for data loading and analysis.
- **Data Visualization**: Key insights were visualized using Canva.

## Key Insights
- Distribution of content types (Movies vs. TV Shows)
- Trends in genre popularity over time
- Insights into age ratings and target audiences
- Regional availability and trends across different countries

## Project Files
-- Netflix Project

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

select * from netflix;


-- Count the number of Movies vs TV Shows.

select 
  type,
  count(*) as Movies_Types 
from netflix 
group by type;


-- Find the most common rating for movies and TV shows.

select type, 
rating from (
select type, 
  rating, 
  count(*),
  rank() over(partition by type order by count(*)desc) as ranking
from netflix 
group by 1,2
) as t1
where ranking= 1;


-- List all movies released in a specific year (e.g., 2020).

select * from netflix
where type= 'Movie' and release_year= 2020;


-- Find the top 5 countries with the most content on Netflix.

SELECT
  unnest(string_to_array(country, ',')) AS new_country,
  COUNT(show_id) AS most_content_created
FROM
  netflix
GROUP BY
  new_country
ORDER BY
  most_content_created DESC
LIMIT 5;


-- Identify the longest movie.

SELECT *
FROM netflix
WHERE duration = (SELECT MAX(duration) FROM netflix WHERE type = 'Movie');


-- Find content added in the last 5 years.

SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 YEAR';


-- Find all the movies/TV shows by director 'Rajiv Chilaka'.

select * from netflix 
where director like '%Rajiv Chilaka%';


-- List all TV shows with more than 5 seasons.

SELECT *
FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;


-- Count the number of content items in each genre.

select unnest(string_to_array(listed_in, ',')) AS GENRE,
COUNT(show_id) AS TOTAL_CONTENT
from netflix
GROUP BY GENRE;  


-- Find each year and the average numbers of content release in India on netflix. 
-- return top 5 year with highest avg content release.

WITH TotalIndiaContent AS (
  SELECT COUNT(*) AS total_content
  FROM netflix
  WHERE country = 'India'
)
SELECT
  EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) AS year,
  COUNT(*) AS yearly_content_count,
  ROUND(100.0 * COUNT(*) / (SELECT total_content FROM TotalIndiaContent), 2) AS avg_content_per_year
FROM
  netflix
WHERE
  country = 'India'
GROUP BY
  1;

-- List all movies that are documentaries.

select * from netflix
where listed_in like ('%Documentaries%');


-- Find all content without a director.

SELECT * FROM NETFLIX 
WHERE director is null;


-- Find how many movies actor 'Salman Khan' appeared in last 10 years.

select * from netflix 
where casts like ('%Salman Khan%') 
and release_year > extract(year from current_date) - 10;


-- Find the top 10 actors who have appeared in the highest number of movies produced in India.

select unnest(String_to_array(casts, ',')) as actors, 
count(*) as total_content 
from netflix 
where country ILIKE '%India%'
group by actors 
order by total_content desc
LIMIT 10;


-- Categorize the content based on the presence of the keywords 'kill' and 'violence' in 
-- the description field. Label content containing these keywords as 'Bad' and all other 
-- content as 'Good'. Count how many items fall into each category.

WITH new_table AS (
  SELECT *,
         CASE
           WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad_Content'
           ELSE 'Good_Content'
         END AS category
  FROM netflix
)
SELECT
  category,
  COUNT(*) AS total_content
FROM
  new_table
GROUP BY
  category;

## Results
- Detailed SQL analysis revealed that **Drama and Comedy** were the most common genres.
- Content targeting **mature audiences (TV-MA)** is highly prevalent in Netflix's library.
- There is a strong trend toward **increasing international content** availability over the years.

## Repository Structure
- `data/`: Contains raw and processed data files.
- `scripts/`: SQL queries for data analysis.
- `visualizations/`: Images and visual content summarizing the project findings.

## Conclusion
Through SQL-based analysis, this project provides a clear view of Netflix's evolving content strategy and audience targeting. The findings from this project could potentially help in predicting future content trends on the platform.
