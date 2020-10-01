# SQL: Structured Query Language Exercise

### Getting Started

1. Go to BigQuery UI https://console.cloud.google.com/bigquery
2. Add in the public data sets. 3. Click the Add Data icon 4. Add any dataset 5. `bigquery-public-data` should become visible and populate in the BigQuery UI.
3. Add your queries where it says [YOUR QUERY HERE].
4. Make sure you add your query in between the triple tick marks.

---

For this section of the exercise we will be using the `bigquery-public-data.austin_311.311_service_requests` table.

5. Write a query that tells us how many rows are in the table.

   ```
   with T as (SELECT
   unique_key,
   count(unique_key) as key_count
   FROM
   `bigquery-public-data.austin_311.311_service_requests`
   group by
   unique_key
   )
   select
   key_count,
   sum(key_count) as rows_total
   from
   T
   group by
   key_count
   ```

6. Write a query that tells us how many _distinct_ values there are in the complaint_description column.

   ```
   SELECT
   complaint_description,
   count(distinct complaint_description) as count_unq_descriptions
   FROM
   `bigquery-public-data.austin_311.311_service_requests`
   Group by
   complaint_description
   LIMIT 1000
   ```

7. Write a query that counts how many times each owning_department appears in the table and orders them from highest to lowest.

   ```
     SELECT
   owning_department,
   count(owning_department) as nowning_department
   FROM
   `bigquery-public-data.austin_311.311_service_requests`
   Group by
   owning_department
   order by
   nowning_department DESC
   LIMIT
   1000

   ```

8. Write a query that lists the top 5 complaint_description that appear most and the amount of times they appear in this table. (hint... limit)
   ```
   SELECT
   complaint_description,
   count(complaint_description) as count_description
   FROM
   `bigquery-public-data.austin_311.311_service_requests`
   Group by
   complaint_description
   order by
   count_description DESC
   LIMIT
   5
   ```
9. Write a query that lists and counts all the complaint_description, just for the where the owning_department is 'Animal Services Office'.

   ```
   SELECT
   complaint_description,
   count(complaint_description) as count_description
   FROM
   `bigquery-public-data.austin_311.311_service_requests`
   Where
   owning_department = "Animal Services Office"
   Group by
   complaint_description
   order by
   count_description DESC
   ```

10. Write a query to check if there are any duplicate values in the unique_key column (hint.. There are two was to do this, one is to use a temporary table for the groupby, then filter for values that have more than one count, or, using just one table but including the `having` function).

    ```
    WITH T AS (
    	 SELECT
    	 unique_key,
    	 COUNT(unique_key) AS count
    	 FROM
    	 `bigquery-public-data.austin_311.311_service_requests`
    	 GROUP BY
    	 unique_key
    	 )
    	 SELECT
    	 count,
    	 FROM T
    	 WHERE
    	 count>1
    ```

### For the next question, use the `census_bureau_usa` tables.

1. Write a query that returns each zipcode and their population for 2000 and 2010.
   ```
   WITH
   T2000 AS (
   SELECT
    zipcode,
    SUM(population) AS populationOf2000
   FROM
    `bigquery-public-data.census_bureau_usa.population_by_zip_2000`
   GROUP BY
    zipcode ),
   T2010 AS (
   SELECT
    zipcode,
    SUM(population) AS populationOf2010
   FROM
    `bigquery-public-data.census_bureau_usa.population_by_zip_2010`
   GROUP BY
    zipcode )
   SELECT
   T2000.zipcode,
   T2000.populationOf2000,
   T2010.populationOf2010
   FROM
   T2000
   JOIN
   T2010
   ON
   T2000.zipcode = T2010.zipcode
   order by
   zipcode
   ```

### For the next section, use the `bigquery-public-data.google_political_ads.advertiser_weekly_spend` table.

1. Using the `advertiser_weekly_spend` table, write a query that finds the advertiser_name that spent the most in usd.

```
   SELECT
   advertiser_name,
   max(spend_usd) as most_spent
   FROM
   `bigquery-public-data.google_political_ads.advertiser_weekly_spend`
   group by advertiser_name
   order by
   most_spent desc
   limit 1

```

2. Who was the 6th highest spender? (No need to insert query here, just type in the answer.)

```

Biden for President

```

3. What week_start_date had the highest spend? (No need to insert query here, just type in the answer.)

```

2020-02-23

```

4. Using the `advertiser_weekly_spend` table, write a query that returns the sum of spend by week (using week_start_date) in usd for the month of August only.

```

   SELECT
   week_start_date,
   sum(spend_usd) as spent_weekly,
   FROM
   `bigquery-public-data.google_political_ads.advertiser_weekly_spend`
   where
    week_start_date BETWEEN '2020-08-01' AND '2020-08-31'
    group by
   week_start_date
   order by
   spent_weekly desc

```

5. How many ads did the 'TOM STEYER 2020' campaign run? (No need to insert query here, just type in the answer.)

```

1386

```

6. Write a query that has, in the US region only, the total spend in usd for each advertiser_name and how many ads they ran. (Hint, you're going to have to join tables for this one).

```

   WITH
   regions_table AS (
   SELECT
      advertiser_name,
      SUM(spend_usd) AS amount_spent,
   FROM
      `bigquery-public-data.google_political_ads.advertiser_stats`
   WHERE
      regions = "US"
   GROUP BY
      advertiser_name ),
   ads_table AS (
   SELECT
      advertiser_name,
      COUNT(ad_id) AS num_ads
   FROM
      `bigquery-public-data.google_political_ads.creative_stats`
   GROUP BY
      advertiser_name )
   SELECT
   A.advertiser_name,
   A.amount_spent,
   B.num_ads
   FROM
   regions_table AS A
   JOIN
   ads_table AS B
   ON
   A.advertiser_name = B.advertiser_name
   ORDER BY
   A.amount_spent DESC

```

7. For each advertiser_name, find the average spend per ad.

```

WITH
ads_table AS (
SELECT
   advertiser_name,
   COUNT(ad_id) AS num_ads
FROM
   `bigquery-public-data.google_political_ads.creative_stats`
GROUP BY
   advertiser_name ),
ad_stats AS (
SELECT
   advertiser_name,
   SUM(spend_usd) AS total_spent
FROM
   `bigquery-public-data.google_political_ads.advertiser_weekly_spend`
GROUP BY
   advertiser_name )
SELECT
A.advertiser_name,
safe_divide(B.total_spent,
   A.num_ads) AS ad_avg
FROM
ads_table AS A
JOIN
ad_stats AS B
ON
A.advertiser_name = B.advertiser_name
ORDER BY
ad_avg DESC

```

8. Which advertiser_name had the lowest average spend per ad that was at least above 0.

```

   WITH
   ads_table AS (
   SELECT
      advertiser_name,
      COUNT(ad_id) AS num_ads
   FROM
      `bigquery-public-data.google_political_ads.creative_stats`
   GROUP BY
      advertiser_name ),
   ad_stats AS (
   SELECT
      advertiser_name,
      SUM(spend_usd) AS total_spent
   FROM
      `bigquery-public-data.google_political_ads.advertiser_weekly_spend`
   GROUP BY
      advertiser_name )
   SELECT
   A.advertiser_name,
   safe_divide(B.total_spent,
      A.num_ads) AS ad_avg,
   FROM
   ads_table AS A
   JOIN
   ad_stats AS B
   ON
   A.advertiser_name = B.advertiser_name
   ORDER BY
   ad_avg ASC
   LIMIT
   1
```

## For this next section, use the `new_york_citibike` datasets.

1. Who went on more bike trips, Males or Females?

```

SELECT
  gender,
  COUNT(gender) gender_count
FROM
  `bigquery-public-data.new_york_citibike.citibike_trips`
GROUP BY
  gender
ORDER BY
  gender_count DESC

```

2. What was the average, shortest, and longest bike trip taken in minutes?

```

SELECT
  round(avg(tripduration)) as duration,
  max(tripduration) as longest_duration,
  min(tripduration) as shortest_duration
FROM
  `bigquery-public-data.new_york_citibike.citibike_trips`


```

3. Write a query that, for every station_name, has the amount of trips that started there and the amount of trips that ended there. (Hint, use two temporary tables, one that counts the amount of starts, the other that counts the number of ends, and then join the two.)

```

WITH
  trip_starts AS (
  SELECT
    start_station_name,
    COUNT(start_station_name) AS count_trip_starts
  FROM
    `bigquery-public-data.new_york_citibike.citibike_trips`
  GROUP BY
    start_station_name ),
  trip_ends AS (
  SELECT
    end_station_name,
    COUNT(end_station_name) AS count_trip_ends
  FROM
    `bigquery-public-data.new_york_citibike.citibike_trips`
  GROUP BY
    end_station_name)
SELECT
  A.start_station_name,
  A.count_trip_starts,
  B.end_station_name,
  B.count_trip_ends
FROM
  trip_starts AS A
JOIN
  trip_ends AS B
ON
  A.start_station_name = B.end_station_name

```

# The next section is the Google Colab section.

1. Open up this [this Colab notebook](https://colab.research.google.com/drive/1kHdTtuHTPEaMH32GotVum41YVdeyzQ74?usp=sharing).
2. Save a copy of it in your drive.
3. Rename your saved version with your initials.
4. Click the 'Share' button on the top right.
5. Change the permissions so anyone with link can view.
6. Copy the link and paste it right below this line.

- YOUR LINK: https://colab.research.google.com/drive/1Mg1y4kHBpO5hc_yobsNGtmeNTYu-3awF?usp=sharing

7. Complete the two questions in the colab notebook file.

```

```
