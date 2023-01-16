# Google Data Analytics Capstone Project
## How Does a Bike-Share Navigate Speedy Success?

## 1. Introduction
Cyclistic is a fictional bike-share company located in Chicago. Cyclistic offers a bike-share program that features more than 5,800 bicycles and 600 docking stations througout the state. It also offers reclining bikes, hand tricycles, and cargo bikes to make bike-share more inclusive to people with disabilities and riders who can't use a standard two-wheeled bike. The majority of riders opt for traditional bikes, and about 8% of riders use the assistive options. Cyclistic users are more likely to ride for leisure, but about 30% use them to commute to work each day. 

### 1.1. Case Background
Until now, Cyclistic’s marketing strategy relied on building general awareness and appealing to broad consumer segments. One approach that helped make these things possible was the flexibility of its pricing plans: single-ride passes, full-day passes, and annual memberships. Customers who purchase single-ride or full-day passes are referred to as casual riders. Customers who purchase annual memberships are Cyclistic members.

Cyclistic’s finance analysts have concluded that annual members are much more profitable than casual riders. Although the pricing flexibility helps Cyclistic attract more customers, Moreno (the director of marketing) believes that maximizing the number of annual members will be key to future growth. Rather than creating a marketing campaign that targets all-new customers, Moreno believes there is a very good chance to convert casual riders into members. She notes that casual riders are already aware of the Cyclistic program and have chosen Cyclistic for their mobility needs.

### 1.2. The Goal and Business Tasks
Moreno as the director of marketing has set a clear goal to design marketing strategies aimed at converting casual riders into annual members. But first, the marketing analyst team need to analyze these following questions:
1. How do annual members and casual riders use Cyclistic bikes differently?
2. Why would casual riders buy Cyclistic annual memberships?
3. How can Cyclistic use digital media to influence casual riders to become members?

The analysis will be done by using Cyclistic historical bike trip data to identify trends.

### 1.3. The Stakeholders
1. Lily Moreno: The director of marketing. Moreno is responsible for the development of campaigns and initiatives to promote the bike-share program. These may include email, social media, and other channels.
2. Cyclistic marketing analytics team: A team of data analysts who are responsible for collecting, analyzing, and reporting data that helps guide Cyclistic marketing strategy.
3. Cyclistic executive team: The executive team that will decide whether to approve the recommended marketing program.


## 2. Data Preparation

In order to analyze the business tasks given, first I will need to download [Cyclistic's historical trip data](https://divvy-tripdata.s3.amazonaws.com/index.html). All their trip data are generated from their bikes trip log and stored as monthly csv files and compressed as zip files. For this analysis I will be using the previous 12 months of trip data which will be from December 2021 to November 2022.

All the downloaded files will be saved in a folder named cyclistic_1year_data, then I will extract each one of them so I will get the csv files. After that, I rename all those csv name as cyclistic_tripdata_[yyyymm].csv and sort them according to the period.


## 3. Processing the Data
During this step, I will be using spreadsheet (MS Excel) and SQL (BigQuery).

### 3.1. Spreadsheet
This is the first step taken where I will import each of csv file into an Excel file (.xlsx). 

Once imported, I will create a new column named "ride_length" where I will substarct the value from "ended_at" column and "started_at" column. This value will be used to analyze how long each rider use Cyclistic's service on one trip.

In this step, I found some inconsistent entries, where the end time (ended_at) value was earlier than the start time (started_at) which might due to some glitch in their logging system. However, since there are no clear instruction or documentation on how the logging system works, so the simple workaround solutions are either to delete the incostistent entries or swap the time so it can be calculated. For this step I choose to just swap the time.

The next step I did was to create another new column called "day_of_week" where I will be using WEEKDAY() function to extract what day fall on each repective date on started_at column. The result will be in number format of 1 to 7, where 1 represents Sunday and 7 represents Saturday.

After I've done modifying the Excel file, I convert them back to csv file so it can be use in the next step processing in SQL.

All those steps above were replicated for all 12 csv files.

Actually, there are more things to be cleaned like plenty of missing values (NULL) for start station and end stations. However, since the files are pretty big with hundreds of thousands of rows per file, it will took a lot of my hardware resources an run extremely slow. That's why I choose to process them furter in SQL.

### 3.2. SQL (BigQuery)
In this step, I started by importing all 12 csv files into BigQuery. However, since some of the csv files are quite big (more than 100 MB), so I decided to transfer them all to Google Cloud Drive. Once all the files completely imported to BigQuery then I continue with data cleaning processes.

a. First, I combined those 12 tables into one single table by using UNION ALL in this following query:
```sql
SELECT *
FROM
( 
SELECT *
FROM `capstone-case-1.cyclistic_monthly.tripdata_202112`
UNION ALL
SELECT *
FROM `capstone-case-1.cyclistic_monthly.tripdata_202201`
UNION ALL
SELECT *
FROM `capstone-case-1.cyclistic_monthly.tripdata_202202`
UNION ALL
SELECT *
FROM `capstone-case-1.cyclistic_monthly.tripdata_202203`
UNION ALL
SELECT *
FROM `capstone-case-1.cyclistic_monthly.tripdata_202204`
UNION ALL
SELECT *
FROM `capstone-case-1.cyclistic_monthly.tripdata_202205`
UNION ALL
SELECT *
FROM `capstone-case-1.cyclistic_monthly.tripdata_202206`
UNION ALL
SELECT *
FROM `capstone-case-1.cyclistic_monthly.tripdata_202207`
UNION ALL
SELECT *
FROM `capstone-case-1.cyclistic_monthly.tripdata_202208`
UNION ALL
SELECT *
FROM `capstone-case-1.cyclistic_monthly.tripdata_202209`
UNION ALL
SELECT *
FROM `capstone-case-1.cyclistic_monthly.tripdata_202210`
UNION ALL
SELECT *
FROM `capstone-case-1.cyclistic_monthly.tripdata_202211`
)
```
The combined table then named as **tripdata_12months**.

b. The next step is to remove all rows where the start_station_name or end_station_name have null values:
```sql
DELETE FROM `capstone-case-1.cyclistic_monthly.tripdata_12months`
WHERE start_station_name IS NULL OR end_station_name IS NULL
```
c. Then I removed all rows where the ride_length below 1 minute:
```sql
DELETE FROM `capstone-case-1.cyclistic_monthly.tripdata_12months`
WHERE ride_length < '00:01:00'
```
## 4. Analysis
In order to answer the first question on "how do annual members and casual riders use Cyclistic bikes differently", first, I will run a query to summarize the total number of users from casual users and member users on day and month basis, and also calculate their average ride length.
```sql
-- This query is to summarize the number of users and their average ride length from casual and member users in every day and month.
SELECT 
  member_casual AS user_type,
  --The following subquery is to extract year and month from TIMESTAMP data.
  CONCAT(
    CAST(EXTRACT(YEAR from started_at) AS string), 
    LPAD(CAST(EXTRACT(MONTH from started_at) AS string),3,' 0') 
    )  AS trip_time,
  COUNT(member_casual) AS no_of_user,
  -- The following subquery is to count average ride length for each user type in every month. 
  TIME(
     EXTRACT(hour   FROM AVG(ride_length - '0:0:0')), 
     EXTRACT(minute FROM AVG(ride_length - '0:0:0')), 
     EXTRACT(second FROM AVG(ride_length - '0:0:0'))
   ) AS avg_ride_length,
  day_of_week
  
FROM `capstone-case-1.cyclistic_monthly.tripdata_12months`

GROUP BY trip_time, user_type, day_of_week
ORDER BY trip_time, user_type, day_of_week
```
The result table will look like this

![image](https://user-images.githubusercontent.com/122529512/212571488-1be92ca1-13a8-47fa-89d7-29e765e60ae4.png)


Although member users seem bigger than casual user in total, I'd like to dig deeper to see if there are chances that some stations are actually attract more casual users than member users. For this reason, I created 2 new tables which are extracted from tripdata_12months table. Those 2 tables will contain data about total number of casual users and member users in each start station and end station. The queries are such follow: 
```sql
-- This query is used to calculate total number of casual user and member user in each start station.
SELECT 
  DISTINCT start_station_name,
  COUNT(CASE member_casual WHEN 'casual'THEN 1 else null end) AS casual_user,
  COUNT(CASE member_casual WHEN 'member'THEN 1 else null end) AS member_user
    
FROM `capstone-case-1.cyclistic_monthly.tripdata_12months`
GROUP BY start_station_name
ORDER BY casual_user DESC, member_user DESC
```

```sql
-- This query is used to calculate total number of casual user and member user in each end station.
SELECT 
  DISTINCT end_station_name,
    COUNT(CASE member_casual WHEN 'casual'THEN 1 else null end) AS casual_user,
  COUNT(CASE member_casual WHEN 'member'THEN 1 else null end) AS member_user
    
FROM `capstone-case-1.cyclistic_monthly.tripdata_12months`
GROUP BY end_station_name
ORDER BY casual_user DESC, member_user DESC
```

And the result tables are shown like these:

![image](https://user-images.githubusercontent.com/122529512/212580883-72c65f8a-df73-49a5-bbb9-21938ef41d52.png)

![image](https://user-images.githubusercontent.com/122529512/212580901-5b4eac5b-4bbe-4167-aaf2-4c5451932107.png)


From the 2 tables shown above, it's pretty obvious that there are actually some stations that has significantly higer number or casual users compared to member user. Moreover, we could see that some of these stations are function as both start station and end station with higher number of casual users. For this reason, I tried to create a comparison table by joining the previous 2 tables (start station users and end station users) to show only stations that function as both start and end station with higher casual users number than the member users. I limit the number of result by only showing stations with at least 1000 users.  

```sql
-- This query is to make a comparison table between stations with higher casual users than member users.
SELECT 
  start_station_name,
  start_user.casual_user AS start_station_casual,
  start_user.member_user AS start_station_member,
  end_station_name,
  end_user.casual_user AS end_station_casual,
  end_user.member_user AS end_station_member

-- To join results of stations which both function as start station and end station.
FROM `capstone-case-1.cyclistic_monthly.start_stations_users` AS start_user
JOIN `capstone-case-1.cyclistic_monthly.end_stations_users` AS end_user
ON start_user.start_station_name = end_user.end_station_name

-- Set conditions where casual users in start stations and end stations are bigger than member users, and bigger than 1000 users.
WHERE start_user.casual_user>start_user.member_user AND end_user.casual_user>end_user.member_user AND start_user.casual_user>1000 AND end_user.casual_user>1000

-- Order the result from highest number to lowest.
ORDER BY start_user.casual_user DESC, end_user.casual_user DESC
```

And here's the result table

![image](https://user-images.githubusercontent.com/122529512/212584451-9e74c94d-0034-4ff5-9e7a-451170ec7add.png)
![image](https://user-images.githubusercontent.com/122529512/212584471-ae43960b-1eea-4871-ad36-c6c8f6574295.png)

The table shows that there are 30 stations that both function as start station and end station with significantly higher casual users than end users.

**Analysis Summary**
1. Total number of member users are higher than casual users.
2. However, when I dig deeper to each stations I found that there are some stations that have significantly higher casual users than member users.
3. On average, casual users use the bike sharing service longer than member users.
4. There are significant increase of user number during mid year which brings me to a hypothesis that people mobility are correlated with seasons.


## 5. Sharing the Findings

In order to share and easily explain my findings from the analysis process, I made a dashboard containing charts and a table from the queried tables before.

### 5.1. Difference Number of User

![Dashboard 1](https://user-images.githubusercontent.com/122529512/212592210-d25b4153-f295-40ef-b1fe-188250fcb3cd.png)

[Tableau link](https://public.tableau.com/views/CyclisticsUsersDifference/Dashboard1?:language=en-US&:display_count=n&:origin=viz_share_link)

From the charts shown above, I draw some coclusions as follow:
1. Casual users are more active during weekends (Saturday and Sunday) while Member users are more active during work days (especially from Wednesday to Friday).
2. Overall, total number of member users are higher than casual users.
3. On average, casual users use the service twice longer (around 20 minutes) compared to member users (around 10 minutes).
4. There are significant increase of users from May to October on both casual and member users. From this chart I drew a hypothesis that users are highly mobile during late spring to early autumn and their mobility are highly affected by seasons. 

### 5.2. Top 30 Stations with More Casual User

The following table will show a list of stations which function as both start station and end station and have more casual users than member users.

![Dashboard 2](https://user-images.githubusercontent.com/122529512/212599740-a3d7d625-005e-46af-9bdc-6781a2addd54.png)

[Tableau link](https://public.tableau.com/views/Top30StationsWithMoreCasualUsers/Dashboard2?:language=en-US&:display_count=n&:origin=viz_share_link)

Since the main goal for this analysis was to convert casual users to member users, I find that this table will be usefull for targeted advertisement. I choose a simple table format so the stakeholders could easily see where they might want to intensify their marketing program in order to convert more casual users.

## 6. Closing

Based on the previous analysis and findings, here are my top 3 recommendations according to the business task given before:
1. Target the top 30 stations (or less if there's budget constraint) to intensify marketing programs and advertisement.
2. Provide special rates to attract existing casual users to convert as member, such as family membership, member get member perks, etc.
3. Use digital services for advertisement, such as location based SMS or promotion email to existing members, social media page, or install screen billboards on those top list stations.  


