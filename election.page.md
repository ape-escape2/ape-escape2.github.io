## US 2020 Election County Flip Map

[back to main page](README.md)

![election](flip_map1)

```SQL

DROP TABLE if EXISTS e2020;
CREATE TEMPORARY TABLE e2020 as
SELECT
state,
county_name,
county_fips,
party2020,
total_votes
FROM
  (SELECT
  state,
  county_name,
  county_fips,
  party as party2020,
  sum(candidatevotes) as total_votes,
  rank() OVER (PARTITION BY state, county_fips ORDER BY sum(candidatevotes) DESC) AS vote_rank
  FROM countypres
  WHERE year = 2020
  GROUP by state,
  county_fips,
  party2020)
WHERE vote_rank = 1;

DROP TABLE if EXISTS e2016;
CREATE TEMPORARY TABLE e2016 as
SELECT
state,
county_name,
county_fips,
party2016,
total_votes
FROM
  (SELECT
  state,
  county_name,
  county_fips,
  party as party2016,
  sum(candidatevotes) as total_votes,
  rank() OVER (PARTITION BY state, county_fips ORDER BY sum(candidatevotes) DESC) AS vote_rank
  FROM countypres
  WHERE year = 2016
  GROUP by state,
  county_fips,
  party2016)
WHERE vote_rank = 1;


DROP TABLE if EXISTS flip_table;
CREATE TABLE flip_table as
SELECT
state,
county_name,
county_fips,
party2016,
party2020,
total_votes,
	CASE
        WHEN party2016 = 'DEMOCRAT' AND party2020 = 'REPUBLICAN' THEN 'Flipped to Republican'
        WHEN party2016 = 'REPUBLICAN' AND party2020 = 'DEMOCRAT' THEN 'Flipped to Democrat'
        WHEN party2016 = party2020 THEN 'Stayed the Same'
        ELSE 'Other'
    END as flip_status
FROM
	(SELECT 
	e2016.state,
	e2016.county_name,
	e2016.county_fips,
	e2016.party2016,
	e2020.party2020,
	e2020.total_votes
	FROM e2016
	JOIN e2020 on e2016.county_fips = e2020.county_fips)
WHERE state != "ALASKA" and state != "HAWAII";



SELECT flip_status,
count(flip_status)
FROM flip_table
GROUP by	flip_status;

DROP TABLE if EXISTS state_table;
CREATE TABLE state_table as
SELECT
state,
party2020,
total_state_votes
FROM(
  SELECT
  state,
  party as party2020,
  sum(candidatevotes) as total_state_votes,
  rank() OVER (PARTITION BY state ORDER BY sum(candidatevotes) DESC) AS vote_rank
  FROM countypres
  WHERE year = 2020
  GROUP by state,
  party2020)
WHERE vote_rank = 1;
```
