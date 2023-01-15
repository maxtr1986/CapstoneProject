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

The next step I did was to create another new column called "day_of_week" where I will be using WEEKDAY() function to extract what day fall on each repective date on started_at column. The result will be in number format of 1 to 7, where 1 represents Sunday and 7 represents Saturday. Unfortunately, I won't be using the results from this new column for some reasons which I will explain later in **conclusion part**.

After I've done modifying the Excel file, I convert them back to csv file so it can be use in the next step processing in SQL.

All those steps above were replicated for all 12 csv files.

Actually, there are more things to be cleaned like plenty of missing values (NULL) for start station and end stations. However, since the files are pretty big with hundreds of thousands of rows per file, it will took a lot of my hardware resources an run extremely slow. That's why I choose to process them furter in SQL.

### 3.2. SQL (BigQuery)
In this step, I started by importing all 12 csv files into BigQuery. However, since some of the csv files are quite big (more than 100 MB), so I decided to transfer them all to Google Cloud Drive. Once all the files completely imported to BigQuery then I continue with data cleaning processes.

a. First, I combined those 12 tables into one single table by using UNION ALL in this [following query](https://console.cloud.google.com/bigquery?sq=505738757381:411ddfd2ae884edb81b0dbec661c55f3):

SELECT *<br />
FROM <br />
( <br />
SELECT * <br />
FROM `capstone-case-1.cyclistic_monthly.tripdata_202112` <br />
UNION ALL <br />
SELECT * <br />
FROM `capstone-case-1.cyclistic_monthly.tripdata_202201` <br />
UNION ALL <br />
SELECT * <br />
FROM `capstone-case-1.cyclistic_monthly.tripdata_202202` <br />
UNION ALL <br />
SELECT * <br />
FROM `capstone-case-1.cyclistic_monthly.tripdata_202203` <br />
UNION ALL <br />
SELECT * <br />
FROM `capstone-case-1.cyclistic_monthly.tripdata_202204` <br />
UNION ALL <br />
SELECT * <br />
FROM `capstone-case-1.cyclistic_monthly.tripdata_202205` <br />
UNION ALL <br />
SELECT * <br />
FROM `capstone-case-1.cyclistic_monthly.tripdata_202206` <br />
UNION ALL <br />
SELECT * <br />
FROM `capstone-case-1.cyclistic_monthly.tripdata_202207` <br />
UNION ALL <br />
SELECT * <br />
FROM `capstone-case-1.cyclistic_monthly.tripdata_202208` <br />
UNION ALL <br />
SELECT * <br />
FROM `capstone-case-1.cyclistic_monthly.tripdata_202209` <br />
UNION ALL <br />
SELECT * <br />
FROM `capstone-case-1.cyclistic_monthly.tripdata_202210` <br />
UNION ALL <br />
SELECT * <br />
FROM `capstone-case-1.cyclistic_monthly.tripdata_202211` <br />
) <br />

The combined table then named as "tripdata_12months" which consist of 4,334,153 rows of data. from December 2021 to November 2022.

b. The next step is to remove all rows where the start_station_name or end_station_name have null values:

DELETE FROM `capstone-case-1.cyclistic_monthly.tripdata_12months` <br />
WHERE start_station_name IS NULL OR end_station_name IS NULL <br />

c. Then I removed all rows where the ride_length below 1 minute:

DELETE FROM `capstone-case-1.cyclistic_monthly.tripdata_12months` <br />
WHERE ride_length <= '00:00:59' <br />
