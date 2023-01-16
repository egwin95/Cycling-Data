# CASE STUDY: CYCLISTIC BIKE-SHARE COMPANY
##### Author: Earl Gwin

##### Date: January 16, 2023

##### Tableau Dashboard

##### Tableau Story

#

_The follow six step process was followed for this case study:_

### [Ask](#1-ask)
### [Prepare](#2-prepare)
### [Process](#3-process)
### [Analyze](#4-analyze)
### [Share](#5-share)
### [Act](#6-act)

## Scenario
You are a junior data analyst working in the marketing analyst team at Cyclistic, a bike-share company in Chicago. The director
of marketing believes the company’s future success depends on maximizing the number of annual memberships. Therefore,
your team wants to understand how casual riders and annual members use Cyclistic bikes differently. From these insights,
your team will design a new marketing strategy to convert casual riders into annual members. But first, Cyclistic executives
must approve your recommendations, so they must be backed up with compelling data insights and professional data
visualizations.

## 1. Ask
  **BUSINESS TASK: Analyze Cyclistic data to determine how Casual and Members are using the service and what steps can be made to bring casual customers in as members by using digital media methods.
  
 Primary Stakeholders: Lily Moreno
 Secondary Stakeholders: Cyclistic marketing analytics team, Cyclistic executive team
 
## 2. Prepare
Data Source: Cyclistic's historical trip data over the last 12 months

The dataset has 13 CSV files, this data follows the ROCCC approach:

- Reliability: The data is from Cyclistic's historical trip data, this data is based off public access to the Bike-Share company and determines trip time based on where the bike was docked.
-  Original: Data is taken from the Bike-Share company
-  Comprehensive: Data is taken at an hourly level, when bikes are checked out, checked in or docked the data is updated with latitude and longitude of the starting and ending station the bike was docked at.
-  Current: Data is from 2022 and tracks data over the course of each month
-  Cited: Cyclistic

## 3. Process

Data cleaning was done using SQL in Google BigQuery, these initial steps are queries used to prepare the data for cleaning. First step was to take all twelve CSV files and combine them into one large database
```
CREATE TABLE Tripdata_2022.combined_tripdata AS
SELECT *
FROM (
     SELECT * FROM cycling-tripdata-2022.Tripdata_2022.Tripdata_January
     UNION ALL 
     SELECT * FROM cycling-tripdata-2022.Tripdata_2022.Tripdata_February
     UNION ALL 
     SELECT * FROM cycling-tripdata-2022.Tripdata_2022.Tripdata_March
     UNION ALL 
     SELECT * FROM cycling-tripdata-2022.Tripdata_2022.Tripdata_April
     UNION ALL 
     SELECT * FROM cycling-tripdata-2022.Tripdata_2022.Tripdata_May
     UNION ALL 
     SELECT * FROM cycling-tripdata-2022.Tripdata_2022.Tripdata_June
     UNION ALL 
     SELECT * FROM cycling-tripdata-2022.Tripdata_2022.Tripdata_July
     UNION ALL 
     SELECT * FROM cycling-tripdata-2022.Tripdata_2022.Tripdata_August
     UNION ALL 
     SELECT * FROM cycling-tripdata-2022.Tripdata_2022.Tripdata_September
     UNION ALL 
     SELECT * FROM cycling-tripdata-2022.Tripdata_2022.Tripdata_October
     UNION ALL 
     SELECT * FROM cycling-tripdata-2022.Tripdata_2022.Tripdata_November
     UNION ALL 
     SELECT * FROM cycling-tripdata-2022.Tripdata_2022.Tripdata_December
     );
```

Used DISTINCT on ride_id so no further cleaning was necessary, then used another query to determine the different ride_types:
```
SELECT DISTINCT rideable_type
FROM cycling-tripdata-2022.Tripdata_2022.combined_tripdata;
```
Next step was to perform a check on started_at and ended_at data sets to make sure rides that are greater than one minute and less than twenty-four hours are shown in our results:
```
SELECT *
FROM cycling-tripdata-2022.Tripdata_2022.combined_tripdata
WHERE TIMESTAMP_DIFF(ended_at, started_at, MINUTE) <= 1 OR
   TIMESTAMP_DIFF(ended_at, started_at, MINUTE) >= 1440;
```
Next I checked for any naming errors for start_station name and ID and end_station name and ID:
```
SELECT start_station_name, COUNT(*)
FROM cycling-tripdata-2022.Tripdata_2022.combined_tripdata
GROUP BY start_station_name
ORDER BY start_station_name;

SELECT end_station_name, COUNT(*)
FROM cycling-tripdata-2022.Tripdata_2022.combined_tripdata
GROUP BY end_station_name
ORDER BY end_station_name;

SELECT COUNT(DISTINCT(start_station_name)) AS startName,
    COUNT(DISTINCT(end_station_name)) AS endName,
    COUNT(DISTINCT(start_station_id)) AS startID,
    COUNT(DISTINCT(end_station_id)) AS endID,
FROM cycling-tripdata-2022.Tripdata_2022.combined_tripdata;
```
For this step I checked for the amount of rows that had NULL values, these rows will be removed later during our cleaning query set:
```
SELECT rideable_type, count(*) as num_of_rides
FROM cycling-tripdata-2022.Tripdata_2022.combined_tripdata
WHERE start_station_name IS NULL AND start_station_id IS NULL OR
    end_station_name IS NULL AND end_station_id IS NULL 
GROUP BY rideable_type;
```
NULLS in start_lat/lng and end_lat/lng:
```
SELECT * FROM cycling-tripdata-2022.Tripdata_2022.combined_tripdata
WHERE start_lat IS NULL OR
      start_lng IS NULL OR
      end_lat IS NULL OR
      end_lng IS NULL;
```
Last pre-cleaning step is to verify that there are only two member types in the member_casual column:
```
SELECT DISTINCT member_casual
FROM cycling-tripdata-2022.Tripdata_2022.combined_tripdata;
```
## 4. Analyze
[Back to Top](#author-Earl-Gwin)

- [Cleaning](#Cleaning)
- [Ride Type Total](#Ride-Type-Total)
- [Monthly Ride Amount](#Monthly-Ride-Amount)
- [Daily Ride Amount](#Daily-Ride-Amount)
- [Hourly Ride Amount](#Hourly-Ride-Amount)
- [Avg Daily Ride Length](#Avg-Daily-Ride-Length)
- [Starting Station for Casuals/Members](#Starting-Station-for-Casuals/Members)
- [Ending Station for Casuals/Members](#Ending-Station-for-Casuals/Members)


### Cleaning:

After the pre-cleaning we now have the necessary steps needed to clean up the data for a better analysis. First step is to find all classic bikes that do not begin or end at a docking station. These data points are not valuable because classic bikes are not able to be locked anywhere, they must be taken from and left at a docking station for the next user.
```
CREATE TABLE Tripdata_2022.null_station_names AS (SELECT ride_id AS bad_ride_id
FROM (SELECT ride_id, start_station_name, start_station_id,end_station_name, end_station_id
      FROM cycling-tripdata-2022.Tripdata_2022.combined_tripdata 
        WHERE rideable_type = 'docked_bike' OR rideable_type = 'classic_bike')
        WHERE start_station_name IS NULL AND start_station_id IS NULL OR 
end_station_name IS NULL AND end_station_id IS NULL);
```

Next step is to remove all the rows from the previous query and also remove any rows that have a NULL value for Longitude and Latitude:
```
CREATE TABLE Tripdata_2022.cleaned_station_names AS
(
  SELECT * 
  FROM cycling-tripdata-2022.Tripdata_2022.combined_tripdata
  LEFT JOIN cycling-tripdata-2022.Tripdata_2022.null_station_names
  ON ride_id = bad_ride_id
  WHERE bad_ride_id IS NULL AND
    start_lat IS NOT NULL AND
    start_lng IS NOT NULL AND
    end_lat IS NOT NULL AND
    end_lng IS NOT NULL
);
```
Here were are  replacing all instances of Docked_Bike with Classic_Bike. This is becaused Docked_Bike is the old name for Classic_Bike so these data sets are outdated and no longer useful:
```
SELECT ride_id, REPLACE (rideable_type, 'docked_bike', 'classic_bike') AS ride_type
FROM cycling-tripdata-2022.Tripdata_2022.cleaned_station_names
```
In this step electric bikes are the only type that do not need to have a starting or ending location, because of this users are able to leave the bike anywhere as long as it is locked up. In this query we will be replacing the NULL starting and ending location values with "Bike Locked":
```
SELECT started_at, ended_at, IFNULL(NULL, "Bike Locked")
FROM cycling-tripdata-2022.Tripdata_2022.cleaned_station_names;
```

We will now create a new column for date and ride length:
```
CREATE TABLE cycling-tripdata-2022.Tripdata_2022.combined_data_cleaned AS
(
  SELECT ride_id,
    REPLACE(rideable_type, 'docked_bike', 'classic_bike') AS ride_type,
    started_at,
    ended_at,
    IFNULL(TRIM(REPLACE(start_station_name, '(Temp)', '')), 'On Bike Lock') AS starting_station_name,
    IFNULL(TRIM(REPLACE(end_station_name, '(Temp)', '')), 'On Bike Lock') AS ending_station_name,
    CASE
      WHEN EXTRACT(DAYOFWEEK FROM started_at) = 1 THEN 'Sun'
      WHEN EXTRACT(DAYOFWEEK FROM started_at) = 2 THEN 'Mon'
      WHEN EXTRACT(DAYOFWEEK FROM started_at) = 3 THEN 'Tues'
      WHEN EXTRACT(DAYOFWEEK FROM started_at) = 4 THEN 'Wed'
      WHEN EXTRACT(DAYOFWEEK FROM started_at) = 5 THEN 'Thur'
      WHEN EXTRACT(DAYOFWEEK FROM started_at) = 6 THEN 'Fri'
      ELSE'Sat' 
    END AS day,
    CASE
      WHEN EXTRACT(MONTH FROM started_at) = 1 THEN 'Jan'
      WHEN EXTRACT(MONTH FROM started_at) = 2 THEN 'Feb'
      WHEN EXTRACT(MONTH FROM started_at) = 3 THEN 'Mar'
      WHEN EXTRACT(MONTH FROM started_at) = 4 THEN 'Apr'
      WHEN EXTRACT(MONTH FROM started_at) = 5 THEN 'May'
      WHEN EXTRACT(MONTH FROM started_at) = 6 THEN 'Jun'
      WHEN EXTRACT(MONTH FROM started_at) = 7 THEN 'July'
      WHEN EXTRACT(MONTH FROM started_at) = 8 THEN 'Aug'
      WHEN EXTRACT(MONTH FROM started_at) = 9 THEN 'Sept'
      WHEN EXTRACT(MONTH FROM started_at) = 10 THEN 'Oct'
      WHEN EXTRACT(MONTH FROM started_at) = 11 THEN 'Nov'
      ELSE 'Dec'
    END AS month,
    EXTRACT(DAY FROM started_at) as day,
    EXTRACT(YEAR FROM started_at) AS year,
    TIMESTAMP_DIFF(ended_at, started_at, MINUTE) AS ride_time_minutes,
    start_lat,
    start_lng,
    end_lat,
    end_lng,
    member_casual AS member_type
  FROM cycling-tripdata-2022.Tripdata_2022.cleaned_station_names
);
```
Now our data is all cleaned and ready for the share process, in order to start that process we will need to create some queries to display results that we can use for our visualization software. These queries include:

### Ride Type Total:
```
SELECT ride_type, member_type, count(*) AS amount_of_rides
FROM cycling-tripdata-2022.Tripdata_2022.combined_data_cleaned
GROUP BY ride_type, member_type
ORDER BY member_type, amount_of_rides DESC;
```

### Monthly Ride Amount:
```
SELECT member_type, month, COUNT(*) AS monthly_rides
FROM cycling-tripdata-2022.Tripdata_2022.combined_data_cleaned
GROUP BY member_type, month;
```

### Daily Ride Amount;
```
SELECT member_type, day_of_week, COUNT(*) AS daily_rides
FROM cycling-tripdata-2022.Tripdata_2022.combined_data_cleaned
GROUP BY member_type, day_of_week;
```

### Hourly Ride Amount:
```
SELECT member_type, EXTRACT(HOUR FROM started_at) AS time_daily, COUNT(*) AS hourly_rides
FROM cycling-tripdata-2022.Tripdata_2022.combined_data_cleaned
GROUP BY member_type, time_daily;
```

### Avg Daily Ride Length;
```
SELECT member_type, day_of_week,ROUND(AVG(ride_time_minutes),0) AS average_time,
AVG(AVG(ride_time_minutes)) OVER(PARTITION BY member_type) AS combined_avg_ride_time
FROM cycling-tripdata-2022.Tripdata_2022.combined_data_cleaned
GROUP BY member_type, day_of_week;
```

### Starting Station for Casuals/Members:
_Casuals_
```
SELECT starting_station_name, AVG(start_lat) AS start_lat, AVG(start_lng) AS start_lng,
COUNT(*) AS number_of_rides
FROM cycling-tripdata-2022.Tripdata_2022.combined_data_cleaned
WHERE member_type = 'casual' AND starting_station_name != 'On Bike Lock'
GROUP BY starting_station_name;
```
_Members_
```
SELECT starting_station_name, AVG(start_lat) AS start_lat, AVG(start_lng) AS start_lng,
COUNT(*) AS number_of_rides
FROM cycling-tripdata-2022.Tripdata_2022.combined_data_cleaned
WHERE member_type = 'member' AND starting_station_name != 'On Bike Lock'
GROUP BY starting_station_name;
```

### Ending Station for Casuals/Members:
_Casuals_
```
SELECT ending_station_name, AVG(end_lat) AS end_lat, AVG(end_lng) AS end_lng,
COUNT(*) AS number_of_rides
FROM cycling-tripdata-2022.Tripdata_2022.combined_data_cleaned
WHERE member_type = 'casual' AND starting_station_name != 'On Bike Lock'
GROUP BY ending_station_name;
```
_Members_
```
SELECT ending_station_name, AVG(end_lat) AS end_lat, AVG(end_lng) AS end_lng,
COUNT(*) AS number_of_rides
FROM cycling-tripdata-2022.Tripdata_2022.combined_data_cleaned
WHERE member_type = 'member' AND starting_station_name != 'On Bike Lock'
GROUP BY ending_station_name;
```

## 5. Share
[Back to Top](#author-Earl-Gwin)

### [Cyclistic Data Analysis Dashboard](https://public.tableau.com/views/CyclingData_16737315586610/Dashboard6?:language=en-US&:display_count=n&:origin=viz_share_link)
![Dashboard 6](https://user-images.githubusercontent.com/42590575/212768785-ac0f8294-9df3-4ab8-b38a-2f6e4094dcea.png)




