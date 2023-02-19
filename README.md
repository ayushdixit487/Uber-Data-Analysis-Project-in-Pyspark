# Uber-Data-Analysis-Project-in-Pyspark
This data project can be used as a take-home assignment to learn Pyspark and Data Engineering.

## Insights from City Supply and Demand Data

## Data Description

To answer the question, use the dataset from the file dataset.csv. For example, consider a row from this dataset:


![image](https://user-images.githubusercontent.com/25612446/219965433-956a1dc0-2acf-4d0d-b1cb-40723249d349.png)




This means that during the hour beginning at 4pm (hour 16), on September 10th, 2012, 11 people opened the Uber app (Eyeballs). 2 of them did not see any car (Zeroes) and 4 of them requested a car (Requests). Of the 4 requests, only 3 complete trips actually resulted (Completed Trips). During this time, there were a total of 6 drivers who logged in (Unique Drivers).

## Assignment 

Using the provided dataset, answer the following questions:

- ⚡ Which date had the most completed trips during the two week period?
- ⚡ What was the highest number of completed trips within a 24 hour period?
- ⚡ Which hour of the day had the most requests during the two week period?
- ⚡ What percentages of all zeroes during the two week period occurred on weekend (Friday at 5 pm to Sunday at 3 am)? Tip: The local time value is the start of the hour (e.g. 15 is the hour from 3:00pm - 4:00pm)
- ⚡ What is the weighted average ratio of completed trips per driver during the two week period? Tip: "Weighted average" means your answer should account for the total trip volume in each hour to determine the most accurate number in whole period.
- ⚡ In drafting a driver schedule in terms of 8 hours shifts, when are the busiest 8 consecutive hours over the two week period in terms of unique requests? A new shift starts in every 8 hours. Assume that a driver will work same shift each day.
- ⚡ True or False: Driver supply always increases when demand increases during the two week period. Tip: Visualize the data to confirm your answer if needed.
- ⚡ In which 72 hour period is the ratio of Zeroes to Eyeballs the highest?
- ⚡ If you could add 5 drivers to any single hour of every day during the two week period, which hour should you add them to? Hint: Consider both rider eyeballs and driver supply when choosing
- ⚡ Looking at the data from all two weeks, which time might make the most sense to consider a true "end day" instead of midnight? (i.e when are supply and demand at both their natural minimums) Tip: Visualize the data to confirm your answer if needed.

## Solution: 
To solve the questions using PySpark, we need to first create a SparkSession and load the dataset into a DataFrame. Here's how we can do it:
```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import *

# Create a SparkSession
spark = SparkSession.builder.appName("UberDataAnalysis").getOrCreate()

# Load the dataset into a DataFrame
df = spark.read.csv("uber.csv", header=True, inferSchema=True) 
```
Now that we have loaded the dataset into a data frame, we can start answering the questions.
Which date had the most completed trips during the two-week period?

- To find the date with the most completed trips, you can group the data by date and sum the completed trips column. Then, sort the results in descending order and select the top row.
```python
from pyspark.sql.functions import max

# Read the data from CSV file
uber = spark.read.csv("uber.csv", header=True, inferSchema=True)

# Group the data by date and sum the completed trips
completed_trips_by_date = uber.groupBy("Date").sum("Completed Trips")

# Find the date with the most completed trips
date_with_most_completed_trips = completed_trips_by_date \
    .orderBy("sum(Completed Trips)", ascending=False) \
    .select("Date") \
    .first()["Date"]

print(date_with_most_completed_trips)

#Output:  2012-09-15
```
- What was the highest number of completed trips within a 24-hour period?

To find the highest number of completed trips within a 24-hour period, you can group the data by date and use a window function to sum the completed trips column over a rolling 24-hour period. Then, you can sort the results in descending order and select the top row.

```python
from pyspark.sql.functions import sum, window

# Read the data from CSV file
uber = spark.read.csv("uber.csv", header=True, inferSchema=True)

# Group the data by 24-hour windows and sum the completed trips
completed_trips_by_window = uber \
    .groupBy(window("Time (Local)", "24 hours")) \
    .agg(sum("Completed Trips").alias("Total Completed Trips")) \
    .orderBy("Total Completed Trips", ascending=False)

# Get the highest number of completed trips within a 24-hour period
highest_completed_trips_in_24_hours = completed_trips_by_window \
    .select("Total Completed Trips") \
    .first()["Total Completed Trips"]

print(highest_completed_trips_in_24_hours)

#Output 2102
```
- Which hour of the day had the most requests during the two-week period?

To answer this question, we need to group the data by an hour and sum the "Requests" column for each hour. We can then sort the result by the sum of requests and select the hour with the highest sum.

```python
from pyspark.sql.functions import hour, sum

hourly_requests = df.groupBy(hour("Time (Local)").alias("hour")).agg(sum("Requests").alias("total_requests")).orderBy("total_requests", ascending=False)

most_requested_hour = hourly_requests.select("hour").first()[0]
print("The hour with the most requests is:", most_requested_hour)

#The hour with the most requests is: 17
```

- What percentages of all zeroes during the two-week period occurred on weekends (Friday at 5 pm to Sunday at 3 am)?

To answer this question, we need to filter the data to select only the rows that fall within the specified time range, count the total number of zeros, and count the number of zeros that occurred on weekends. We can then calculate the percentage of zeros that occurred on weekends.

```python
from pyspark.sql.functions import dayofweek, hour

weekend_zeros = df.filter((hour("Time (Local)") >= 17) | (hour("Time (Local)") < 3)).filter((dayofweek("Date") == 6) | (dayofweek("Date") == 7)).agg(sum("Zeroes").alias("weekend_zeros")).collect()[0]["weekend_zeros"]

total_zeros = df.agg(sum("Zeroes").alias("total_zeros")).collect()[0]["total_zeros"]

percent_weekend_zeros = weekend_zeros / total_zeros * 100

print("The percentage of zeros that occurred on weekends is:", percent_weekend_zeros, "%")

#The percentage of zeros that occurred on weekends is: 41.333414829040026 %
```

