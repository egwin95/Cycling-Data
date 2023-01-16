# CASE STUDY: CYCLISTIC BIKE-SHARE COMPANY
##### Author: Earl Gwin

##### Date: January 16, 2023

##### Tableau Dashboard

##### Tableau Story

#

_The follow six step process was followed for this case study:_

### [Ask](#1- Ask)
### [Prepare](#2- Prepare)
### [Process](#3- Process)
### [Analyze](#4- Analyze)
### [Share](#5- Share)
### [Act](#6- Act)

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

Data cleaning was done using SQL in Google BigQuery, first step was to take all twelve CSV files and combine them into one large database
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

