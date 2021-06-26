
--revenue budget ratio:
SELECT id, round(CONVERT(float,revenue)/CONVERT(float,budget),4)  AS rev_budg_ratio,
CASE when round(CONVERT(float,revenue)/CONVERT(float,budget),4)>3 then 1
ELSE 0 END AS y_hat
INTO yhat
FROM dbo.movies 
WHERE budget > 0


-----------------------------------------------------------------------------------------------------------------------------

--genre pivot:

SELECT id , [name],
1 AS [type]
INTO temp5
FROM dbo.genres_dim

SELECT *
INTO temp6
FROM temp5
PIVOT
(
count([type])
FOR [name]
IN ([Action],[Adventure], [Animation], [Comedy], [Crime],[Documentary], [Drama], [Family], [Fantasy],[Foreign], [History],[Horror], [Music],[Mystery], [Romance], [Science Fiction], [Thriller], [TV Movie], [War], [Western] )) AS PIVOT_LANG
ORDER BY (id)


select DISTINCT movie_id, sum(Action) as Action ,sum(Adventure) as Adventure, sum(Animation) as Animation,
sum(Comedy) as Comedy, sum(Crime) as Crime, sum(Documentary) as Documentary, sum(Drama) as Drama, sum(Family) as Family, sum(Fantasy) as Fantasy,
sum([Foreign]) as [Foreign], sum(History) as History, sum(Horror) as Horror, sum(Music) as Music, sum(Mystery) as Mystery, sum(Romance) as Romance,
sum([Science Fiction]) as [Science Fiction], sum(Thriller) as Thriller, sum([TV Movie]) as [TV Movie], sum(War) as War, sum(Western) as Western
INTO
GENDER01
FROM 
dbo.movies_genres as g1
LEFT JOIN
temp6 as g2
ON 
g1.id=g2.id
GROUP BY movie_id
ORDER BY movie_id

-----------------------------------------------------------------------------------------------------------------------------

--languages pivot:

SELECT movie_id, 1 AS counts, iso_639_1
INTO langs
FROM dbo.movie_languages
 
 
SELECT movie_id, [en], [fr], [es], [de], [ru], [it], [ja]
INTO langs_pivot
FROM langs
PIVOT (
   COUNT(counts)
   FOR iso_639_1
   IN ([en], [fr], [es], [de], [ru], [it], [ja])
) AS a
ORDER BY movie_id


select * from langs_pivot
order by movie_id



-----------------------------------------------------------------------------------------------------------------------------

-- department pivot:

SELECT * FROM dbo.movie_crew


SELECT movie_id, id, department
INTO crew
FROM dbo.movie_crew
 
SELECT department, COUNT(DISTINCT movie_id)
FROM crew
GROUP BY department
 
SELECT *
INTO department_pivot
FROM crew
PIVOT (
   COUNT(id)
   FOR department
   IN ([Art],[Camera],[Costume & Make-Up],[Crew],[Directing],[Editing],[Lighting],[Production],[Sound],[Visual Effects],[Writing])
) AS a
ORDER BY movie_id

SELECT * FROM department_pivot


-----------------------------------------------------------------------------------------------------------------------------

-- female/male counts:

SELECT * FROM dbo.movie_crew
SELECT * FROM dbo.crew_dim


SELECT movie_id,
      SUM(CASE WHEN (c.gender = 1) THEN (1) ELSE (0) END) AS female_cnt
INTO movie_females
FROM dbo.movie_crew as b
INNER JOIN dbo.crew_dim as c
   ON b.id = c.id
GROUP BY movie_id
ORDER BY movie_id
 
 
SELECT movie_id,
      SUM(CASE WHEN (c.gender = 2) THEN (1) ELSE (0) END) AS male_cnt
INTO movie_males
FROM dbo.movie_crew b
INNER JOIN dbo.crew_dim c
   ON b.id = c.id
GROUP BY movie_id
ORDER BY movie_id


SELECT * FROM movie_males
SELECT * FROM movie_females

select *
from movie_females

select * from movie_females

select * from movie_males
select * from movie_males

select * from movies_departments_v


-----------------------------------------------------------------------------------------------------------------------------

--FINAL TABLE:

SELECT
DISTINCT M.id,
M.budget,
CASE when homepage is not null then 1
ELSE 0 END AS homepage,
M.original_language,
CASE when M.original_language='en' then 1
ELSE 0 END AS original_is_en,
M.original_title,
popularity,
CASE WHEN M.popularity < 3.0 THEN 'Low'
   WHEN M.popularity BETWEEN 3.0 AND 5.99 THEN 'Medium'WHEN M.popularity BETWEEN 6.0 AND 8.99 THEN 'High'
   ELSE 'Superb' END AS 'popularity_cat',
M.release_date,
YEAR(CONVERT(DATE, M.release_date)) AS [year],
MONTH(CONVERT(DATE, M.release_date)) AS [month],
M.runtime,
M.revenue,
CASE WHEN (M.revenue < 1000000) THEN ('No') ELSE ('Yes') END AS best_seller,
DENSE_RANK() OVER(PARTITION BY YEAR(CONVERT(DATE, M.release_date)) ORDER BY M.revenue DESC) AS revenue_rank,
SUM(CAST(M.revenue AS FLOAT)) OVER(PARTITION BY YEAR(CONVERT(DATE, M.release_date))) AS yearly_revenue_sum,
round(M.revenue / SUM(CAST(M.revenue AS FLOAT)) OVER(PARTITION BY YEAR(CONVERT(DATE, M.release_date))),4) * 100 AS revenue_year_percent,
 T1.rev_budg_ratio,
Action,Adventure, Animation,Comedy, Crime, Documentary, Drama, Family, Fantasy,[Foreign], History,Horror, Music,Mystery, Romance, [Science Fiction], Thriller, [TV Movie], War, Western,
en, fr, es, de, ru, it, ja,
[Art],[Camera],[Costume & Make-Up],[Crew],[Directing],[Editing],[Lighting],[Production],[Sound],[Visual Effects],[Writing],
female_cnt, male_cnt
INTO final_table
FROM dbo.movies AS M
LEFT JOIN 
yhat AS T1
ON M.id=T1.id
LEFT JOIN 
dbo.movies_genres AS T2
ON M.id=T2.movie_id
LEFT JOIN 
dbo.genres_dim AS T3
ON T2.id=T3.id
LEFT JOIN 
GENDER01 AS T4
ON M.id=T4.movie_id
LEFT JOIN
langs_pivot AS T5
ON M.id=T5.movie_id
LEFT JOIN 
department_pivot as T6
ON M.id=T6.movie_id
LEFT JOIN 
movie_males AS T7
ON M.id=T7.movie_id
LEFT JOIN 
movie_females AS T8
ON M.id=T8.movie_id
--WHERE revenue is not NULL and budget > 0


select * from final_table
