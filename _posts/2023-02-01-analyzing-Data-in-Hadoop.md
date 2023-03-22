---
title: Data Analysis in Hadoop
tags: Hadoop bigdata SQL
aside:
  toc: true
  selectors: 'h3'
author: Farooq
cover: "/assets/images/hadoop.png"
show_edit_on_github: false
---

During my Master's at UTD I enrolled in a Big Data course by a great professor, Rami El-Youssef. I enjoyed his lectures a lot and took detailed notes every class. I did his assignments every week which gave me practical knowledge. But I wanted to put all these learnings together with a project. So the professor came up with a truck telematics dataset and asked me to come up with some business questions myself.

After looking at the data I was able to understand that there is data related to 3 entities trucks, drivers, and violations. 

After a lot of thinking I came up with 3 business requirements.

1. A Dashboard to look at generic metrics like - Number of trucks in fleet, count by truck make, MPG
2. A Dashboard to identify risky drivers based on violations, and at which location they have occured
3. An Analysis to identify which truck models had the best MPG ratings among the fleet.

Now to build a database to fulfill these requirements I needed to perform some tranformations and build new tables. So I came up with something which looked like this to make my mental model clear. This is not a ERD diagram.


Once I had these transformations on paper I went ahead and created them using SQL DDL statements with HiveQL and Impala. This is a sample DDL statement

```sql
CREATE TABLE geolocation
(
truckid string,
driverid string,
event string,
latitude DOUBLE,
longitude DOUBLE,
city string,
state string,
velocity BIGINT,
event_ind BIGINT,
idling_ind BIGINT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
TBLPROPERTIES (”skip.header.|ine.count"="l");
```

Later I had to do load the data from file or by calculations and do some sanity checks to make sure that all the data was inplace like needed.

The next step was to solve the business requirements

First I needed to connect to Tableau to make the dashboard and for that I need a JDBC driver which i doenalodd from so and so and IP addrss of my Hadoop server to coonect to. Once connected I created my charts and create dthe dashbord

Later I connected hadoop to R using so and so and from trucks_mg data I imported 2 columsna and applied Tukey HSD to find out this and concluded that 

In a perfect world I would not conduct some hypoethesis tests and check for randomization but I assumed all this data was random and conducted 3 way ANOVA






---

You can find more details on this repository --> [Data Analysis in Hadoop]()
