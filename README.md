# Netflix Movies and TV Shows Data Analysis using SQL
![Netflix Logo](https://github.com/sathvika77777/netflix_sql_project/blob/main/BrandAssets_Logos_01-Wordmark.jpg)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset

The data for this project is sourced from the Kaggle dataset:
- **Dataset Link:** [Movies Dataset](https://github.com/sathvika77777/netflix_sql_project/blob/main/netflix_titles%202.csv)

## Schema

drop table if exists  netflix;
```sql
-- create table
Create table netflix(
show_id varchar(6),
type varchar(10),
title varchar(150),
director varchar(210),
casts varchar(1000),
country varchar(150),
date_added varchar(50),
release_year int,
rating varchar(10),
duration varchar(15),
listed_in varchar(100),
description varchar(250)
);
```

select * from netflix;

select distinct(type) from netflix;

## -- 15 business problems

## -- 1.Count the number of Movies vs TV Shows
```sql
select type ,count(title) from netflix
group by type
```


## -- 2.Find the most common rating for movies and TV shows
```sql
with common_rating as(select type,rating,count(rating),
rank() over(partition by type order by count(rating) desc) as rank
from netflix
group by type,rating)

select type,rating from  common_rating
where rank = 1;

```

## -- 3.List all movies released in a specific year (e.g., 2020)
```sql
select release_year,title from netflix
where type  = 'Movie' and release_year = '2020'

```

## -- 4.Find the top 5 countries with the most content on Netflix
```sql
select * from netflix;

select unnest(string_to_array(country,',')) as new_country,count(show_id) as most_content 
from netflix
group by new_country
order by most_content desc
limit 5;

select show_id,title,unnest(string_to_array(country,',')) as new_country
from netflix;

```

## -- 5.Identify the longest movie or TV show duration
```sql
select * from netflix
where type = 'Movie'
and
duration = (select max(duration) from netflix)
;
```

## -- 6.Find all content added in the last 5 years
```sql
select * FROM NETFLIX
WHERE to_date(date_added,'Month DD,YYYY') >= CURRENT_DATE - INTERVAL '5 YEARS'
```

## -- 7.Find all the movies/TV shows by director 'Rajiv Chilaka'
```sql
Select * from netflix 
where director Like '%Rajiv Chilaka%'
```

## -- 8.List all TV shows with more than 5 seasons
```sql
select *
from netflix
where type = 'TV Show'
and split_part(duration,' ',1)::numeric > 5
```


## -- 9.Count the number of content items in each genre
```sql
select trim(unnest(string_to_array(listed_in,','))) as genre,count(show_id) as total
from netflix
group by 1
order by total desc
```


## -- 10.Find each year and the average numbers of content release in india on netflix
```sql
select  extract(year from to_date(date_added,'Month DD, YYYY')) as year,
count(*),
round(count(*) ::numeric/(select count(*) from netflix where country = 'India')::numeric * 100,2) as avg_content_per_year
from netflix
where country = 'India'
group by 1
```


## -- 11.Find all content without a director
```sql
select * from netflix
where director is null
```
## -- 12.Find how many movies actor 'Salman Khan' appeared in the last 10 years
```sql
select * from netflix
where casts like '%Salman Khan%'
and release_year > extract(year from current_date) - 10
```


## -- 13.Find the top 10 actors who have appeared in the highest number of movies produced in India
```sql
with highest as(select *,unnest(string_to_array(casts,',')) as names from netflix)

select names,count(*) as total from highest
where country like '%India%'
group by names 
order by total desc
limit 10


```
## -- 14.List all movies that are documentaries
```sql
select title,listed_in from netflix
where listed_in like '%Documentaries%'
```

## -- 15.Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field.
## Label content containing these keywords as 'Bad'.
## Label all other content as 'Good'.
## Count how many items fall into each category.
```sql

WITH category AS (
  SELECT *,
    CASE 
      WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
      ELSE 'Good'
    END AS violence_rating 
  FROM netflix
)

SELECT violence_rating, COUNT(*) 
FROM category
GROUP BY violence_rating;
```

