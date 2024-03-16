# Cyclistic Project

As a data analyst in training I set out to experiment with databases to view, modify and analyze data that could be useful. 

*What is this project about?*  This is the first project I do, the database was proposed in coursera, where I received my data analyst certification. It is my first approach as a data analyst to a real project. 
  
🎯 ***Cyclistic** is a bike sharing company in Chicago. 
In this case study, the marketing director asked a junior data analyst because she believed that instead of acquiring new customers, it would be more profitable to convert casual members to annual members. Our task is to understand how different types of members use Cyclistic. From our insights, the team will design a new marketing strategy.*

>❗️To note: those customers who purchase a single or day pass are considered **casual riders**, and those who purchase an annual membership are **Cyclistic members**.

*To complete this project, I will use SQlite (in this case DB browser) and Tableau to be able to capture our results.*

# Code

    ## Create a database and import the tables. I will be working in 2023.
    
    1. Clean the database: look for duplicates, nulls or inconsistency. Ride_id should be unique so look for duplicates:
    
    SELECT ride_id, COUNT(*)
    FROM tripdata_2023
    GROUP BY ride_id
    ORDER BY ride_id DESC;
    
    ##Checking for nulls
    SELECT COUNT(*) FILTER (WHERE rideable_type IS NULL) AS rideable_type_nulls,
    COUNT(*) FILTER (WHERE started_at IS NULL) AS started_at_nulls,
    COUNT(*) FILTER (WHERE ended_at IS NULL) AS ended_at_nulls,
    COUNT(*) FILTER (WHERE start_station_name IS NULL) AS start_station_name_nulls,
    COUNT(*) FILTER (WHERE start_station_id IS NULL) AS start_station_id_nulls,
    COUNT(*) FILTER (WHERE end_station_name IS NULL) AS end_station_name_nulls,
    COUNT(*) FILTER (WHERE end_station_id IS NULL) AS end_station_id_nulls,
    COUNT(*) FILTER (WHERE start_lat IS NULL) AS start_lat_nulls,
    COUNT(*) FILTER (WHERE start_lng IS NULL) AS start_lng_nulls,
    COUNT(*) FILTER (WHERE end_lat IS NULL) AS end_lat_nulls,
    COUNT(*) FILTER (WHERE end_lng IS NULL) AS end_lng_nulls,
    COUNT(*) FILTER (WHERE member_casual IS NULL) AS member_casual_nulls
    FROM tripdata_2023;
    
    ## Nulls are less than 16% in columns like start_station_name, start_station_id, end_station_name and end_station_id therefore I'll keep the columns. I’ll exclude the outliers later on the visualization.
    
      
    
    2. Now I’ll transform the database. I’ll remove: rideable_type since it is not relevant for the analysis. I will also drop start_station_id and end_station_id because they are not consistent therefore it will not help me to identify the stations. I need to create a new column called “Ride_length”, so I split started_at and ended_at and got new columns only within hours.
    
    UPDATE tripdata_2023
    SET started_time = SUBSTR(started_at, 12, 8),
        ended_time = SUBSTR(ended_at, 12,8);

    ## I’ll keep the original columns only as date
    
    UPDATE tripdata_2023
    SET started_at = SUBSTR(started_at, 1, 10),
        ended_at = SUBSTR(ended_at, 1,10);
    
    ## Now I can easily calculate the ride length
    
    UPDATE tripdata_2023
    SET ride_length = strftime('%H:%M:%S', (strftime('%s', ended_time) - strftime('%s', started_time)), 'unixepoch');
    
    ## Also i needed a new column with the day of the week from 0 to 7
    
    UPDATE tripdata_2023
    SET day_of_the_week = strftime('%w', started_at ) ;
    
    ## I will cast it so it’s easier to understand
    
    UPDATE tripdata_2023
    SET day =
    CASE day_of_the_week
    WHEN 0 THEN 'Sunday'
    WHEN 1 THEN 'Monday'
    WHEN 2 THEN 'Tuesday'
    WHEN 3 THEN 'Wednesday'
    WHEN 4 THEN 'Thursday'
    WHEN 5 THEN 'Friday'
    ELSE 'Saturday'
    END;
    
    ## The last column I need to create is ride_distance
    
    UPDATE tripdata_2023
    SET ride_distance = round (6371 * 2 * ASIN(SQRT(SIN(RADIANS(end_lat - start_lat) / 2) * SIN(RADIANS(end_lat - start_lat) / 2) +
    
    COS(RADIANS(start_lat)) * COS(RADIANS(end_lat)) * SIN(RADIANS(end_lng - start_lng) / 2) * SIN(RADIANS(end_lng - start_lng) / 2)
     )) ,2);
    
    ## There were a few outliers, I had to removed them to not bias the database
    
    DELETE from tripdata_2023
    where started_time > ended_time AND started_date = ended_date;
    AND
    DELETE from tripdata_2023
    where end_lat= ‘0.’0;
    
    ## For some reason there were trips than last less than 1 minute and the bike never left the started_station
    
    DELETE from tripdata_2023
    where ride_length < "00:01:00"
    
    3. Now let’s get some insights, I will base my analysis on the comparative between members and casual riders. To begin with I want to know how many members and how many casuals riders we have
    
    SELECT count(ride_id), member_casual
    FROM tripdata_2023
    GROUP BY member_casual;
   
    ## Let’s see how they use Cyclistic so then we can plan how to transform casuals riders into members
    
    SELECT count(ride_id), member_casual, day
    FROM tripdata_2023
    GROUP BY member_casual,day;
   
    ## Let's also look at the distance covered by a casual rider and a Cyclistic member
    
    SELECT avg(ride_distance_km), member_casual
    FROM tripdata_2023
    GROUP BY member_casual;
    
    SELECT round(sum(ride_distance_km),2), member_casual
    FROM tripdata_2023
    GROUP BY member_casual;
    
    ## Taking into account the number of kilometers traveled, let's see the average number of trips made.
    
    SELECT strftime('%M:%S',avg(ride_length)) as average_ride_length, member_casual
    FROM tripdata_2023
    GROUP BY member_casual;
    
    ## Let's check which are the most popular stations for casual riders and members
    
    SELECT member_casual, start_station_name, count(*) as popular_stations
    FROM tripdata_2023
    WHERE start_station_name is NOT NULL
    AND member_casual = 'member'
    GROUP BY start_station_name, member_casual
    ORDER BY popular_stations DESC
    LIMIT 3;
    
    SELECT member_casual, start_station_name, count(*) as popular_stations
    FROM tripdata_2023
    WHERE start_station_name is NOT NULL
    AND member_casual = ‘casual’
    GROUP BY start_station_name, member_casual
    ORDER BY popular_stations DESC
    LIMIT 3;
    
    ## I also would like to know some date context, different seasons of the year may modify the use of the service, and we also take into account the time frames.
    
    SELECT count(ride_id),member_casual
    FROM tripdata_2023
    WHERE started_date BETWEEN '2023-06-21' AND '2023-09-22'
    GROUP by member_casual;
    
    SELECT count(ride_id),member_casual
    FROM tripdata_2023
    WHERE (started_date BETWEEN '2023-01-01' AND '2023-03-20') or ( started_date BETWEEN '2023-12-21' AND '2023-12-31')
    GROUP BY member_casual;
    
    SELECT count(ride_id),member_casual
    FROM tripdata_2023
    WHERE started_time BETWEEN '07:00:00' AND '13:00:00'
    GROUP BY member_casual;
    
    SELECT count(ride_id),member_casual
    FROM tripdata_2023
    WHERE started_time BETWEEN '14:00:00' AND '18:00:00'
    GROUP by member_casual;
    
      
## Visualization

[Check Cyclistic's visualization on Tableau](https://public.tableau.com/views/Cyclisticproject_17053805753600/Dashboard1?:language=es-ES&:sid=&:display_count=n&:origin=viz_share_link)


## Conclusions

Based on our analysis, the difference between casual riders and members lies in the following:

**Casual** riders use Cyclistic more on *Saturdays and Sundays*, presenting higher amounts of trips between 2pm and 7pm, we can infer that they use the bikes to ride on the weekend. While **members** ride more on T*uesdays, Wednesdays and Thursdays*, and with peak times between 8am and 5pm, we infer that they use Cyclistic to get to work.

While there are 1,558,765 more members than casual riders, in terms of time, **casual riders are the ones who use it the longest**, with an average of almost 21:43 minutes, compared to members who have an average of 12:23 minutes. However the average kilometers are more or less similar, with 2.14 kilometers for members and 2.15 for casual riders.

Additionally we include the 3 most used start stations because in order to provide a good service it is important that Cyclistic has enough bikes at each station. Thanks to the map, we can see that the stations that casual riders use are those that go along the coast, while members tend to go inside the city.

***Recommendations for the marketing team:***
Knowing how each type of user uses Cyclistic, and keeping in mind for profitability purposes we need to convert casual riders to members, some of the suggestions are:

 - Since casual members have an average number of minutes that is almost
   double than members, we can offer discounts on the membership,  so   
   casual riders will prefer to pay for the membership, almost for  the 
   same price.
   
 - Since casual riders bike on the weekend, one option is a pack or   
   discounts for members that includes Saturday and Sunday rides.

     

 - Understanding that peak usage hours are between 12pm and 7pm, with
   peaks at 5pm, you can offer exclusive benefits to members, such as priority access to bikes at peak times or discounts on rentals.
 - Payment facilities can be offered to those who are members.
 - Considering that it is an annual membership and that from November to April trips are almost nil, some kind of compensation or  benefits can be offered.
 - As part of customer loyalty, a points system can be implemented, where points are earned for each trip.


