# Case Study - Cyclistic Bike Share, Google Data Analytics Capstone Project

## Senario 
  You are a junior data analyst working on the marketing analyst team at Cyclistic, a bike-share companyinChicago. The director of marketing believes the company’s future success depends on maximizing the number of annual memberships. Therefore, your team wants to understand how casual riders and annual members use Cyclistic bikes differently. From these insights, your team will design a new marketing strategy to convert casual riders into annual members. But first, Cyclistic executives must approve your recommendations, so they must be backed up with compelling data insights and professional data visualizations.You are a junior data analyst working on the marketing analyst team at Cyclistic, a bike-share company in Chicago. 

### Business Task 
  Design marketing strategies aimed at converting casual riders into annual members.

### Key Information 
-  The program has 5,824 bicycles that are geotracked and locked into a network of 692 stations across Chicago. The bikes can be unlocked from one station and returned to any other station in the system anytime.
- Pricing plans: single-ride passes, full l-day passes, and annual memberships. Customers who purchase single-ride or full-day passes are referred to as casual riders. Customers who purchase annual memberships are Cyclistic members.
- Cyclistic’s finance analysts have concluded that annual members are much more profitable than casual riders.  Maximizing the number of annual members will be key to future growth.


## Ask 
  How do annual members and casual riders use Cyclistic bikes differently?

## Prepare 

### Data Source

  I will use Cyclistic's historical trip data from September 2023 to August 2024 downloaded from [divvy-tripdata](https://divvy-tripdata.s3.amazonaws.com/index.html). This data has been made available by Motivate International Inc under this [license](https://divvybikes.com/data-license-agreement).

  This is public data that can be used to explore how different customer types are using Cyclistic bikes. But note that data-privacy issues prohibit from using riders’ personally identifiable information.

### Data Organization

  There are 12 files with the naming convention of YYYYMM-divvy-tripdata and each file includes information for one month. Each file has 13 columns; ride_id, rideable_type, started_at, ended_at, start_station_name, end_station_id, start_lat, start_lng, end_lat, end_lng, member_casual. 

## Process

### 1. Union Tables

```sql
  CREATE TABLE upheld-rain-403114.bike_trips.data_23_24 AS 
SELECT *
FROM `upheld-rain-403114.bike_trips.Sep_23` 
UNION ALL
SELECT *
FROM `upheld-rain-403114.bike_trips.oct_23` 
UNION ALL
SELECT *
FROM `upheld-rain-403114.bike_trips.nov_23` 
UNION ALL
SELECT *
FROM `upheld-rain-403114.bike_trips.dec_23` 
UNION ALL
SELECT *
FROM `upheld-rain-403114.bike_trips.jan_24` 
UNION ALL
SELECT *
FROM `upheld-rain-403114.bike_trips.feb_24` 
UNION ALL
SELECT *
FROM `upheld-rain-403114.bike_trips.mar_24` 
UNION ALL
SELECT *
FROM `upheld-rain-403114.bike_trips.apr_24` 
UNION ALL
SELECT *
FROM `upheld-rain-403114.bike_trips.may_24` 
UNION ALL
SELECT *
FROM `upheld-rain-403114.bike_trips.jun_24` 
UNION ALL
SELECT *
FROM `upheld-rain-403114.bike_trips.jul_24` 
UNION ALL
SELECT *
FROM `upheld-rain-403114.bike_trips.aug_24`
```

### 2. Check for Null Values

```sql
 SELECT
  SUM(CASE WHEN ride_id IS NULL THEN 1 ELSE 0 END) AS rideid_null_count,
  SUM(CASE WHEN  rideable_type IS NULL THEN 1 ELSE 0 END) AS ridetype_null_count,
  SUM(CASE WHEN  member_casual IS NULL THEN 1 ELSE 0 END) AS mc_null_count,
  SUM(CASE WHEN  started_at IS NULL THEN 1 ELSE 0 END) AS start_null_count, 
  SUM(CASE WHEN  ended_at IS NULL THEN 1 ELSE 0 END) AS end_count,
  SUM(CASE WHEN  start_station_name IS NULL THEN 1 ELSE 0 END) AS station_null_count

FROM `upheld-rain-.bike_trips.c_data_23_24`
```

  - 968674 null values in start_station_name column
  - data missing from september

### 3. Delete Duplicates
 
  Check for Duplicates 
```sql
SELECT 
count(ride_id),
count(distinct ride_id) 
FROM `upheld-rain-403114.bike_trips.data_23_24`
```
* Found 211 Duplicates 

  Delete Duplicates 
  
```sql
CREATE OR REPLACE TABLE upheld-rain-403114.bike_trips.data_23_24 AS
WITH RankedRows AS (
  SELECT
    *,
    ROW_NUMBER() OVER (PARTITION BY ride_id ORDER BY ride_id) AS row_num
  FROM
    upheld-rain-403114.bike_trips.data_23_24
)
SELECT
  *
FROM
  RankedRows
WHERE
  row_num = 1;
-- keeps only the first instance of each ride_id
```

### 4. Create Tables

  Add New Columns 
```sql
SELECT
  *,
  timestamp_diff(ended_at, started_at, MINUTE) as length,
  format_timestamp('%m', started_at) AS month,
  format_timestamp('%A', started_at) AS day_of_week,
  format_timestamp('%H', started_at) AS hour
FROM `upheld-rain-403114.bike_trips.data_23_24` 
WHERE
  timestamp_diff(ended_at, started_at, MINUTE)>=0
/*
- add new columns; length of ride, month, day of week, hour
- delete negative rows from length of ride
*/
```


  Add New Columns, Aggregate by Start Station 
```sql
SELECT 
    start_station_name,
    count(ride_id) AS rides,
    countif(member_casual = 'member') as member, 
    countif(member_casual = 'casual') AS casual,
    round(countif(member_casual = 'member') / count(ride_id)*100)  as member_percent,
    round(countif(member_casual = 'casual') / count(ride_id)*100) as casual_percent,
    countif(member_casual = 'casual') - countif(member_casual = 'member') as diff

FROM `upheld-rain-403114.bike_trips.c_data_23_24`
group by start_station_name
order by diff desc

/*
  - for each station count ride; total, member, and casual
  - add new columns; percent member, percent casual, causal member difference
*/
```

### 5. Save Results


## Analyze 

Analyzed data using Tableau Public

![mc_month](https://github.com/user-attachments/assets/a48b95f8-681d-4b24-8ee9-854b287b7f7a)
- Number of rides are highest during summer months, and lowest during winter months
- Number of rides likely correlate with factors such as temperature and daylight hours


![mc_day](https://github.com/user-attachments/assets/5ffff2e2-f3d5-44d8-ab9a-1d08653c655a)
- Casual rides increase on the weekends
- Member rides increase on weekdays


![mc_hour](https://github.com/user-attachments/assets/5c9a10ce-d4a8-4735-a9cc-e634f43a1ea5)
- Members ride most during commuting hours
- Causal rides peak at 5:00 PM


![mc_total_30_60](https://github.com/user-attachments/assets/39f91961-afa2-4d4d-9a1d-f998167b471d)
- Members are more likely to take rides under 30 minutes, while casual riders are more likely to take rides over 30 minutes.


![mc_type](https://github.com/user-attachments/assets/01a02c19-b526-4704-8b05-c084e6419a52)
- Casual and member rides both each rideable type very similarly


![len_day](https://github.com/user-attachments/assets/e13fb4e6-e3c4-42c3-94ba-63fa1dab8692)
- Casual riders on average ride longer on the weekends


![difference](https://github.com/user-attachments/assets/8e2f9b83-e6b3-4db8-a639-d4498c499dda)
- Top station discrepancy between casual and member use shows potential for conversion growth where casual members are already using the service frequently.
- Knowing stations dominated by causal riders can allow you to create focused marketing and membership incentives
- Understating usage patterns like tourist spots can help targeted marketing efforts 


## Share 

![mc_dash](https://github.com/user-attachments/assets/0bc6c39f-544e-44ba-89b1-d85ae6d6d910)


![len_dash](https://github.com/user-attachments/assets/374d683b-f410-47e8-a949-996b4ecbd88a)





