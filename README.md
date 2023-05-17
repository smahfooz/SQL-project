# SQL-project

Question #1: What is the movie with the highest average_rating of the year 2022- in case of same average_rating give the one with more num_votes - with more than 100,000 num_votes ?
SELECT 
primary_title,
num_votes,
MAX(average_rating) as avg_rating
FROM 
`awesome-chess-369408.imdb.title_basics` tb
LEFT JOIN 
`awesome-chess-369408.imdb.title_ratings` tr
ON 
tr.tconst=tb.tconst
WHERE 
(tr.num_votes>100000)
AND (tb.start_year = 2022)
AND (tb.title_type= 'movie')
GROUP BY primary_title, num_votes
ORDER BY MAX(average_rating) DESC, num_votes DESC
LIMIT 1

 

Question #2: In order to have 1 metric to find the best movies, we will use average_rating * num_votes that we will call rating_score - What are the top 5 movies with the highest rating_score ever ?

SELECT 
primary_title, (average_rating *num_votes) as rating_score
FROM 
`awesome-chess-369408.imdb.title_basics` tb
LEFT JOIN 
`awesome-chess-369408.imdb.title_ratings` tr
ON 
tr.tconst=tb.tconst
WHERE title_type = 'movie'
ORDER BY rating_score DESC
LIMIT 5

 
Question #3: Get a query to have for each year, the number of movies released, the highest rating_score for the year and the average of average_rating for the year as well as the sum of num_votes - order by year descending

SELECT 
start_year, 
COUNT(primary_title) as number_of_movies, 
MAX(average_rating *num_votes) as highest_rating_score, 
AVG(average_rating) as avg_rating, 
SUM(num_votes) as sum_of_votes
FROM 
`awesome-chess-369408.imdb.title_basics` tb
JOIN 
`awesome-chess-369408.imdb.title_ratings` tr
ON 
tr.tconst=tb.tconst
WHERE title_type= 'movie'
GROUP BY start_year
ORDER BY start_year DESC

 



Question #4: Who is the actor who played in movies that has the biggest sum of rating_score - provide also his average of average_rating and his number of movies ??

SELECT  
primary_name, COUNT (primary_title) as count_titles, SUM(num_votes*average_rating) as rating_score, AVG(average_rating) as avg_of_avg
FROM 
`awesome-chess-369408.imdb.name_basics` nb
JOIN
`awesome-chess-369408.imdb.title_principals` tp
ON 
nb.nconst = tp.nconst
JOIN 
`awesome-chess-369408.imdb.title_ratings` tr
ON
tp.tconst = tr.tconst
JOIN 
`awesome-chess-369408.imdb.title_basics` tb
ON
tb.tconst = tr.tconst
WHERE 
category = 'actor'
AND
title_type = 'movie'
GROUP BY primary_name
ORDER BY SUM(num_votes*average_rating) DESC 
LIMIT 1

 
Question #5: What are the top 3 movies with highest rating_score for the actor found above

SELECT  
primary_title, (num_votes*average_rating) as rating_score 
FROM 
`awesome-chess-369408.imdb.name_basics` nb
LEFT JOIN
`awesome-chess-369408.imdb.title_principals` tp
ON 
nb.nconst = tp.nconst
LEFT JOIN 
`awesome-chess-369408.imdb.title_ratings` tr
ON
tp.tconst = tr.tconst
LEFT JOIN 
`awesome-chess-369408.imdb.title_basics` tb
ON
tb.tconst = tr.tconst
WHERE 
primary_name = 'Brad Pitt' 
AND
title_type = 'movie'
ORDER BY (num_votes*average_rating) DESC 
LIMIT 3

--Complex version

SELECT 
primary_title
FROM
`awesome-chess-369408.imdb.name_basics` nb
JOIN
`awesome-chess-369408.imdb.title_principals` tp
ON nb.nconst= tp.nconst
JOIN 
`awesome-chess-369408.imdb.title_basics` tb
ON tb.tconst = tp.tconst
JOIN 
`awesome-chess-369408.imdb.title_ratings` tr
ON tr.tconst = tb.tconst
WHERE
primary_name = (SELECT  
primary_name
FROM 
`awesome-chess-369408.imdb.name_basics` nb
JOIN
`awesome-chess-369408.imdb.title_principals` tp
ON 
nb.nconst = tp.nconst
JOIN 
`awesome-chess-369408.imdb.title_ratings` tr
ON
tp.tconst = tr.tconst
JOIN 
`awesome-chess-369408.imdb.title_basics` tb
ON
tb.tconst = tr.tconst
WHERE 
category = 'actor'
AND
title_type = 'movie'
GROUP BY primary_name
ORDER BY SUM(num_votes*average_rating) DESC 
LIMIT 1)
ORDER BY (num_votes*average_rating) DESC
LIMIT 3
 
Question #6: Who is the actor who played in at least 5 movies with the highest average rating_score per movie (what is his average rating_score )?

SELECT
primary_name, COUNT(DISTINCT primary_title) AS title_count, AVG(num_votes*average_rating) 
AS avg_rating_score
FROM 
`awesome-chess-369408.imdb.name_basics` nb
JOIN
`awesome-chess-369408.imdb.title_principals` tp
ON 
nb.nconst = tp.nconst
JOIN 
`awesome-chess-369408.imdb.title_ratings` tr
ON
tp.tconst = tr.tconst
JOIN 
`awesome-chess-369408.imdb.title_basics` tb
ON
tb.tconst = tr.tconst
WHERE 
title_type = 'movie' 
AND
category = 'actor'
GROUP BY primary_name
HAVING COUNT(DISTINCT(primary_title))>=5
ORDER BY AVG(num_votes*average_rating) DESC
LIMIT 1

 
ANOTHER METHOD (changing the type of joins- as the count of movies is different) 

SELECT
primary_name, COUNT(DISTINCT primary_title) AS title_count, AVG(num_votes*average_rating) as avg_rating_score
FROM 
`awesome-chess-369408.imdb.title_basics` tb
LEFT JOIN
`awesome-chess-369408.imdb.title_principals` tp
ON 
tb.tconst = tp.tconst
LEFT JOIN 
`awesome-chess-369408.imdb.name_basics` nb
ON
nb.nconst = tp.nconst
LEFT JOIN 
`awesome-chess-369408.imdb.title_ratings` tr
ON
tp.tconst = tr.tconst
WHERE 
title_type = 'movie' 
AND
category = 'actor'
GROUP BY primary_name
HAVING COUNT(primary_title)>=5
ORDER BY AVG(num_votes*average_rating) DESC
LIMIT 1


 

Question #7: Create a Query to get the top movie (highest rating_score) for each year

SELECT 
primary_title,
(average_rating*num_votes) AS rating_score,
X.start_year
FROM 
`awesome-chess-369408.imdb.title_basics` tb
JOIN 
`awesome-chess-369408.imdb.title_ratings` tr
ON tb.tconst = tr.tconst
JOIN 
(SELECT 
start_year,  
MAX(average_rating *num_votes) as highest_rating_score
FROM 
`awesome-chess-369408.imdb.title_basics` tb
JOIN 
`awesome-chess-369408.imdb.title_ratings` tr
ON 
tr.tconst=tb.tconst
WHERE title_type ='movie'
GROUP BY start_year
ORDER BY start_year DESC) X
ON tb.start_year = X.start_year
WHERE
(average_rating*num_votes) = X.highest_rating_score
ORDER BY 3 DESC

 
Question #8: For each Movie Genre, for the release since 2000, give the movie title with the highest rating_score 

SELECT
tb.genres,
primary_title,
start_year,
(average_rating*num_votes) AS rating_score
FROM 
`awesome-chess-369408.imdb.title_basics` tb
LEFT JOIN 
`awesome-chess-369408.imdb.title_ratings` tr
ON
tb.tconst=tr.tconst
LEFT JOIN
(SELECT 
genres,  
MAX(average_rating *num_votes) as highest_rating_score
FROM 
`awesome-chess-369408.imdb.title_basics` tb
LEFT JOIN 
`awesome-chess-369408.imdb.title_ratings` tr
ON 
tr.tconst=tb.tconst
GROUP BY genres
ORDER BY genres) X
ON tb.genres = X.genres
WHERE
(average_rating*num_votes) = X.highest_rating_score
AND
start_year>=2000
AND
title_type = 'movie'
ORDER BY X.highest_rating_score DESC

 

Question #9: Duo? Find the actors duo that get the highest average rating_score per movie together in ordering 1 or 2, with at least 4 movies together

WITH actor_duo AS
(SELECT 
primary_name, tb.primary_title, tp.tconst
FROM 
`awesome-chess-369408.imdb.name_basics` nb
JOIN
`awesome-chess-369408.imdb.title_principals` tp
ON nb.nconst = tp.nconst
JOIN 
`awesome-chess-369408.imdb.title_basics` tb
ON tp.tconst = tb.tconst
WHERE tp.ordering<=2
AND title_type LIKE '%movie%'
AND category = 'actor')
SELECT 
ad.primary_name,
ad1.primary_name,
AVG(average_rating*num_votes) AS rating_score,
COUNT(ad.primary_title) AS movie_count
FROM actor_duo ad
JOIN actor_duo ad1
ON ad.primary_name > ad1.primary_name
AND
ad.primary_title = ad1.primary_title
JOIN 
`awesome-chess-369408.imdb.title_ratings` tr
ON ad.tconst = tr.tconst
GROUP BY 1,2
HAVING COUNT(ad.primary_title) >=4
ORDER BY rating_score DESC
LIMIT 1

 
Question #10: Extra - Bonus question if you manage a smart way to show 2 example of their movies together and then to limit the cases where the movies titles very similar looking like they belong to a sequel…

WITH actor_duo AS
(SELECT primary_name, 
primary_title
FROM `awesome-chess-369408.imdb.name_basics` nb
JOIN `awesome-chess-369408.imdb.title_principals` tp
ON nb.nconst = tp.nconst
JOIN `awesome-chess-369408.imdb.title_basics` tb
ON tp.tconst = tb.tconst
WHERE
title_type LIKE '%movie%'
AND
category = 'actor')
SELECT 
ad.primary_title,
FROM actor_duo ad
JOIN actor_duo ad1
ON ad.primary_title = ad1.primary_title
WHERE 
ad.primary_name = 'Robert Downey Jr.'
AND
ad1.primary_name = 'Chris Evans'
AND 
ad.primary_title LIKE '%Avengers%'
LIMIT 2


 

