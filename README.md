# Cyclistic Case Study – Google Data Analytics Capstone

## Business Task
Identify how casual riders and annual members use Cyclistic bikes differently to inform a marketing strategy aimed at converting casual riders into members.

## Data Visualisation and Presentation

- [Cyclystic case study - Tableau Viz](https://public.tableau.com/app/profile/aleah.skerrett.mcgeary/viz/CyclisticCaseStudy_17554330262020/Dashboard2)
- [How can Cyclystic navigate a speedy success - Presentation](https://docs.google.com/presentation/d/1o22WrFxQnMYvc_BtG7orxv9mngM31xa9/edit?usp=drive_link&ouid=105025862529639766636&rtpof=true&sd=true)
---

## 1. Data Source
- Source: [Divvy Trip Data](https://divvy-tripdata.s3.amazonaws.com/index.html)
- Format: Monthly CSVs
- Loaded into: Google BigQuery (via direct upload)

---

## 2. Data Cleaning (in BigQuery)

### SQL Query Highlights
```sql
-- Calculate ride duration and filter out bad data
SELECT
  ride_id,
  start_station_name,
  started_at,
  end_station_name,
  ended_at,
  TIMESTAMP_DIFF(ended_at, started_at, MINUTE) AS duration_minutes
FROM `portfolio-465720.2025_cyclistic.202501_cyclistic_data`
WHERE ended_at > started_at AND end_station_name <> 'null' AND start_station_name <> 'null'

LIMIT 1000
```

- Removed null values
- Found duration time in seconds. Later changed to minutes

```
SELECT
  ride_id,
  start_station_name,
  started_at,
  end_station_name,
  ended_at,
  TIMESTAMP_DIFF(ended_at, started_at, MINUTE) AS duration_minutes
FROM `portfolio-465720.2025_cyclistic.202501_cyclistic_data`
WHERE ended_at > started_at AND end_station_name <> 'null' AND start_station_name <> 'null'
UNION ALL

SELECT
  ride_id,
  start_station_name,
  started_at,
  end_station_name,
  ended_at,
  TIMESTAMP_DIFF(ended_at, started_at, MINUTE) AS duration_minutes
FROM `portfolio-465720.2025_cyclistic.202502_cyclistic_data`
WHERE ended_at > started_at AND end_station_name <> 'null' AND start_station_name <> 'null'
UNION ALL

SELECT
  ride_id,
  start_station_name,
  started_at,
  end_station_name,
  ended_at,
  TIMESTAMP_DIFF(ended_at, started_at, MINUTE) AS duration_minutes
FROM `portfolio-465720.2025_cyclistic.202503_cyclistic_data`
WHERE ended_at > started_at AND end_station_name <> 'null' AND start_station_name <> 'null'
UNION ALL

SELECT
  ride_id,
  start_station_name,
  started_at,
  end_station_name,
  ended_at,
  TIMESTAMP_DIFF(ended_at, started_at, MINUTE) AS duration_minutes
FROM `portfolio-465720.2025_cyclistic.202504_cyclistic_data`
WHERE ended_at > started_at AND end_station_name <> 'null' AND start_station_name <> 'null'
```

#### About the above SQL
- Combined the data for the four months using UNION ALL
- Duration in minutes
- Lots of data to sort through

```
SELECT
  ride_id,
  rideable_type,
  start_station_name,
  started_at,
  end_station_name,
  ended_at,
  member_casual,
  TIMESTAMP_DIFF(ended_at, started_at, MINUTE) AS duration_minutes
FROM `portfolio-465720.2025_cyclistic.202501_cyclistic_data`
WHERE ended_at > started_at AND end_station_name <> 'null' AND start_station_name <> 'null'
UNION ALL
SELECT
  ride_id,
  rideable_type,
  start_station_name,
  started_at,
  end_station_name,
  ended_at,
  member_casual,
  TIMESTAMP_DIFF(ended_at, started_at, MINUTE) AS duration_minutes
FROM `portfolio-465720.2025_cyclistic.202502_cyclistic_data`
WHERE ended_at > started_at AND end_station_name <> 'null' AND start_station_name <> 'null'
UNION ALL
SELECT
  ride_id,
  rideable_type,
  start_station_name,
  started_at,
  end_station_name,
  ended_at,
  member_casual,
  TIMESTAMP_DIFF(ended_at, started_at, MINUTE) AS duration_minutes
FROM `portfolio-465720.2025_cyclistic.202503_cyclistic_data`
WHERE ended_at > started_at AND end_station_name <> 'null' AND start_station_name <> 'null'
UNION ALL
SELECT
  ride_id,
  rideable_type,
  start_station_name,
  started_at,
  end_station_name,
  ended_at,
  member_casual,
  TIMESTAMP_DIFF(ended_at, started_at, MINUTE) AS duration_minutes
FROM `portfolio-465720.2025_cyclistic.202504_cyclistic_data`
WHERE ended_at > started_at AND end_station_name <> 'null' AND start_station_name <> 'null'
```

#### About the above SQL
- Updated to include more columns, so when I do further queries, I have more info
- Saved as view called 2025_clyclistic_unioned_table

**Calculate average ride duration**

```
SELECT
  rideable_type,
  ROUND(AVG(duration_minutes),2) AS average_duration_minutes
FROM `portfolio-465720.2025_cyclistic.2025_cyclistic_unioned_table`
GROUP BY rideable_type
```

- Avg duration in minutes grouped by rideable_type
- Rounded to 2 decimal places to neatness
- Saved and exported as csv

```
SELECT
  member_casual,
  ROUND(AVG(duration_minutes),2) AS average_duration_minutes
FROM `portfolio-465720.2025_cyclistic.2025_cyclistic_unioned_table`
GROUP BY member_casual
```

- Avg duration grouped by membership status
- Rounded to 2 decimal places to neatness
- Saved and exported as csv

**Calculate the total number of rides per hour of each day**

```
SELECT
  EXTRACT(HOUR FROM started_at) AS ride_hour,
  COUNT (*) AS ride_count
FROM `portfolio-465720.2025_cyclistic.2025_cyclistic_unioned_table`
GROUP BY ride_hour
ORDER BY ride_count DESC
```

- Extracted the hour from each ride’s start time so I can see the busiest hours
- COUNT (*) tallies how many rides start in each hour
- ORDER BY in DESC to show busiest hours first

```
SELECT
  EXTRACT(HOUR FROM started_at) AS ride_hour, member_casual,
  COUNT (*) AS ride_count
FROM `portfolio-465720.2025_cyclistic.2025_cyclistic_unioned_table`
GROUP BY ride_hour, member_casual
ORDER BY ride_count, member_casual DESC
```

- Added in member_casual to see the hours by member type
- Saved and exported as csv

```
SELECT
  EXTRACT(HOUR FROM started_at) AS ride_hour, rideable_type,
  COUNT (*) AS ride_count
FROM `portfolio-465720.2025_cyclistic.2025_cyclistic_unioned_table`
GROUP BY ride_hour, rideable_type
ORDER BY ride_count, rideable_type DESC
```

- Added in rideable_type to see the hours by bike type
- Saved and exported as csv

**Calculate average number of rides per hour**

```
SELECT
  EXTRACT(HOUR FROM started_at) AS ride_hour,
  ROUND(AVG(duration_minutes), 2) AS avg_duration_by_hour_in_mins
FROM `portfolio-465720.2025_cyclistic.2025_cyclistic_unioned_table`
GROUP BY ride_hour
ORDER BY ride_hour DESC
```

- Took it a step further: average duration by hour
- Before it was total hours, but I wanted to see which hour had the longest time on average

```
SELECT
  EXTRACT(HOUR FROM started_at) AS ride_hour, rideable_type,
  ROUND(AVG(duration_minutes), 2) AS avg_duration_by_hour_in_mins
FROM `portfolio-465720.2025_cyclistic.2025_cyclistic_unioned_table`
GROUP BY ride_hour, rideable_type
ORDER BY ride_hour, rideable_type DESC

SELECT
  EXTRACT(HOUR FROM started_at) AS ride_hour, member_casual,
  ROUND(AVG(duration_minutes), 2) AS avg_duration_by_hour_in_mins
FROM `portfolio-465720.2025_cyclistic.2025_cyclistic_unioned_table`
GROUP BY ride_hour, member_casual
ORDER BY ride_hour, member_casual DESC
```

- I added rideable_type and  member_casual to break it down further and offer deeper insights
- Rounded to 2 decimal places for neatness

Returned to first query and include start and end latitude and longitude
```
  start_station_name,
  start_lat, start_lng,
  started_at,
  end_station_name,
  end_lat, end_lng,
```
Starting in Tableau made me realise I may want to show a map

```
WITH rides_per_day AS (
  SELECT
    DATE(started_at) AS ride_date,
    FORMAT_DATE('%A', DATE(started_at)) AS day_of_week,
    EXTRACT(DAYOFWEEK FROM started_at) AS day_num,
    member_casual,
    COUNT(*) AS daily_ride_count
  FROM
    `portfolio-465720.2025_cyclistic.2025_cyclistic_unioned_table`
  GROUP BY
    ride_date, day_of_week, day_num, member_casual
),

average_rides AS (
  SELECT
    day_of_week,
    day_num,
    member_casual,
    AVG(daily_ride_count) AS avg_rides_per_day
  FROM
    rides_per_day
  GROUP BY
    day_of_week, day_num, member_casual
)

SELECT
  day_of_week,
  member_casual,
  ROUND(avg_rides_per_day, 2) AS avg_rides_per_day
FROM
  average_rides
ORDER BY
  day_num, member_casual;
```

Showing average number of rides per day by membership status
Tricky so used ChatGPT for help

- **Step 1 (CTE rides_per_day):** Calculates total rides per day, grouped by member_casual and ride_date.

- **Step 2 (CTE average_rides):** Calculates the average ride count for each weekday across the 4-month span.

- **Final SELECT:** Returns the weekday name, member type, and average rides, ordered from Sunday to Saturday.

```
WITH rides_per_day AS (
  SELECT
    DATE(started_at) AS ride_date,
    FORMAT_DATE('%A', DATE(started_at)) AS day_of_week,
    EXTRACT(DAYOFWEEK FROM started_at) AS day_num,
    rideable_type,
    COUNT(*) AS daily_ride_count
  FROM
    `portfolio-465720.2025_cyclistic.2025_cyclistic_unioned_table`
  GROUP BY
    ride_date, day_of_week, day_num, rideable_type
),

average_rides AS (
  SELECT
    day_of_week,
    day_num,
    rideable_type,
    AVG(daily_ride_count) AS avg_rides_per_day
  FROM
    rides_per_day
  GROUP BY
    day_of_week, day_num, rideable_type
)

SELECT
  day_of_week,
  rideable_type,
  ROUND(avg_rides_per_day, 2) AS avg_rides_per_day
FROM
  average_rides
ORDER BY
  day_num, rideable_type;
```

- Repeated the process for rideable type

```
WITH daily_rides AS (
  SELECT
    DATE(started_at) AS ride_date,
    FORMAT_DATE('%B', DATE(started_at)) AS month_name,
    EXTRACT(MONTH FROM started_at) AS month_num,
    member_casual,
    COUNT(*) AS daily_ride_count
  FROM
    `portfolio-465720.2025_cyclistic.2025_cyclistic_unioned_table`
  GROUP BY
    ride_date, month_name, month_num, member_casual
),


monthly_avg AS (
  SELECT
    month_name,
    month_num,
    member_casual,
    ROUND(AVG(daily_ride_count), 2) AS avg_daily_rides
  FROM
    daily_rides
  GROUP BY
    month_name, month_num, member_casual
)


SELECT
  month_name,
  member_casual,
  avg_daily_rides
FROM
  monthly_avg
ORDER BY
  month_num, member_casual;
```
I’ve repeated the query but edited it so it shows average number of rides in months Jan-Apr 2025

- **daily_rides CTE:** Counts rides per day for each member_casual in each month.

- **monthly_avg CTE:** Calculates the average number of rides per day in each month.

- **Final SELECT:** Outputs results in calendar order, not alphabetically.

```
WITH daily_rides AS (
  SELECT
    DATE(started_at) AS ride_date,
    FORMAT_DATE('%B', DATE(started_at)) AS month_name,
    EXTRACT(MONTH FROM started_at) AS month_num,
    rideable_type,
    COUNT(*) AS daily_ride_count
  FROM
    `portfolio-465720.2025_cyclistic.2025_cyclistic_unioned_table`
  GROUP BY
    ride_date, month_name, month_num, rideable_type
),


monthly_avg AS (
  SELECT
    month_name,
    month_num,
    rideable_type,
    ROUND(AVG(daily_ride_count), 2) AS avg_daily_rides
  FROM
    daily_rides
  GROUP BY
    month_name, month_num, rideable_type
)


SELECT
  month_name,
  rideable_type,
  avg_daily_rides
FROM
  monthly_avg
ORDER BY
  month_num, rideable_type;
```

---

## 3. Data Visualisation (in Tableau)

Exported all of my queries into csv format and uploaded them to Tableau

Took some attempts, but I finally settled on a passable Tableau viz. https://public.tableau.com/app/profile/aleah.skerrett.mcgeary/viz/CyclisticCaseStudy_17554330262020/Dashboard2

- **Format:** Tableau

- **Link:** Cyclistic Case Study | Tableau Public

## 4. Time to Act (in Google Sheets)

The **ACT** phase of this project involves a data-driven marketing strategy.
- [View Google Slides Presentation](https://docs.google.com/presentation/d/1o22WrFxQnMYvc_BtG7orxv9mngM31xa9/edit?usp=drive_link&ouid=105025862529639766636&rtpof=true&sd=true)
