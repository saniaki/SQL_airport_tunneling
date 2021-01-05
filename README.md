# SQL_airport_tunneling
SQL queries for analyzing and managing big data available on Amazon Web Service coulde

<p align="center">
<img  align="center" src="https://github.com/saniaki/SQL_airport_tunneling/blob/main/images/map.jpg" width="400"/>


**Databases** : <br>
1 - fly databse
* ten full years of data, representing flights and airports statistics from January 1, 2008 through December 31, 2017. 
* four tables, flights (61392822 rows), planes (453361 rows), airports (1333 rows), airlines (25 rows)
* source https://www.bts.dot.gov/newsroom/2018-traffic-data-us-airlines-and-foreign-airlines-us-flights  <br>
2 - ....

**SQL query engine**: Impala using Hue web interface

## Phase 1: Finding two most approperiate airports to construct a high-speed rail tunnel

Criteria: 
* two airports must be between 300 and 400 miles apart
* at least 5000 average flights per year between airports, in each direction
* airport pair with largest total number of seats on the planes between them

What to report:
* pair of airports with above criteria
* average arrival delay for flights between them
* average flight distance (in miles) between airports, in each direction
* average number of flights per year between airports, in each direction
* average yearly total number of seats (passengers) on the planes between the two airports

### Examining data
Following quesries are used to examin data in Hue.

setting "fly" database as current
<pre>
USE fly;
</pre>

Showing tables available in the databse:
<pre>
SHOW TABLES;
</pre>
<p align="center">
<img  align="center" src="https://github.com/saniaki/SQL_airport_tunneling/blob/main/images/Tables%20in%20fly%20Databse.jpg" width="100"/>

Showing columns in the "flights" and "planes" tables:
<pre>
SHOW TABLES fly.flights;
</pre>
<p align="center">
<img  align="center" src="https://github.com/saniaki/SQL_airport_tunneling/blob/main/images/flights%20Columns.jpg" width="700"/>
<pre>
SHOW TABLES fly.planes;
</pre>
<p align="center">
<img  align="center" src="https://github.com/saniaki/SQL_airport_tunneling/blob/main/images/planes%20Columns.jpg" width="700"/>

Showing few lines of "flights" and "planes" tables:
<pre>
SELECT * FROM fly.flights
    LIMIT 5; 
</pre>
<p align="center">
<img  align="center" src="https://github.com/saniaki/SQL_airport_tunneling/blob/main/images/flights%20Samples.jpg" width="900"/>
<pre>
SELECT * FROM fly.planes
    LIMIT 5;
</pre>
<p align="center">
<img  align="center" src="https://github.com/saniaki/SQL_airport_tunneling/blob/main/images/planes%20Samples.jpg" width="900"/>



### SQL query
<pre>
SELECT origin, dest,<br>
        ROUND(CAST(COUNT(*) AS decimal)/10) AS avg_num_flights,<br>
        SUM(seats) AS tot_num_seats,<br>
        ROUND(CAST(SUM(seats) AS decimal)/10) AS avg_annual_num_pass,<br>
        ROUND(AVG(distance)) AS avg_flight_dist,<br>
        ROUND(AVG(arr_delay)) AS avg_arr_delay_minutes<br>
FROM fly.flights AS f<br>
LEFT OUTER JOIN fly.planes AS p<br>
        ON f.tailnum = p.tailnum<br>
WHERE distance BETWEEN 300 AND 400<br>
GROUP BY origin, dest<br>
        HAVING avg_num_flights >= 5000  <br>
ORDER BY tot_num_seats DESC;<br>
</pre>


### Results
First 8 rows of the results table:

<p align="center">
<img  align="center" src="https://github.com/saniaki/SQL_airport_tunneling/blob/main/images/Results.jpg" width="900"/>


First and second directions:

<pre>
                                                First direction     Second direction
Three-letter airport code for origin            SFO	                LAX
Three-letter airport code for destination       LAX	                SFO
Average flight distance in miles                337	                337
Average number of flights per year              14712               14540
    Average annual passenger capacity           1996597             1981059
Average arrival delay in minutes                10	                14
</pre>

Sanity checks: 
* Average flight distance is expected to be the same for each pairs
* Average flight distance for all rows in the results table should be between 300 and 400
* Average flight distance for all rows in the results table should be between 300 and 400


## Phase 2: Finding two most approperiate airports to construct a high-speed rail tunnel

### Examining data files
Following quesries are used to examin data usin AWS CLI (Amazon Web Service Command Line Interface)
<pre>
aws s3 ls s3://training-coursera2/tbm_sf_la/
</pre>
<p align="center">
<img  align="center" src="https://github.com/saniaki/SQL_airport_tunneling/blob/main/images/map.jpg" width="400"/>
    
<pre>
aws s3 cp s3://training-coursera2/tbm_sf_la/central/hourly_central.csv -|head
</pre>
<p align="center">
<img  align="center" src="https://github.com/saniaki/SQL_airport_tunneling/blob/main/images/map.jpg" width="400"/>
    
<pre>
aws s3 cp s3://training-coursera2/tbm_sf_la/north/hourly_north.csv -|head
</pre>
<p align="center">
<img  align="center" src="https://github.com/saniaki/SQL_airport_tunneling/blob/main/images/map.jpg" width="400"/>
    
<pre>
aws s3 cp s3://training-coursera2/tbm_sf_la/south/hourly_south.tsv -|head
</pre>
<p align="center">
<img  align="center" src="https://github.com/saniaki/SQL_airport_tunneling/blob/main/images/map.jpg" width="400"/>

**Observations**:
* First and second data files are comma delimited and third data files is tab delimited
* Value 999999 is used instead of NULL values in first data file
* First data file has a header


### Creating three individual tables in HDFS for each TBM (bouring machine)
<pre>
CREATE EXTERNAL TABLE dig.tbm_sf_la_central
    (tbm STRING,year SMALLINT,month TINYINT,day TINYINT,hour TINYINT,
    dist DECIMAL(8,2),lon DOUBLE,lat DOUBLE)
    ROW FORMAT DELIMITED
    FIELDS TERMINATED BY ','
    TBLPROPERTIES('skip.header.line.count'='1','serialization.null.format'='999999');
</pre>

<pre>
CREATE EXTERNAL TABLE dig.tbm_sf_la_north
    (tbm STRING,year SMALLINT,month TINYINT,day TINYINT,hour TINYINT,
    dist DECIMAL(8,2),lon DOUBLE,lat DOUBLE)
    ROW FORMAT DELIMITED
    FIELDS TERMINATED BY ',';
</pre>

<pre>
CREATE EXTERNAL TABLE dig.tbm_sf_la_south
    (tbm STRING,year SMALLINT,month TINYINT,day TINYINT,hour TINYINT,
    dist DECIMAL(8,2),lon DOUBLE,lat DOUBLE)
    ROW FORMAT DELIMITED
    FIELDS TERMINATED BY '\t';
</pre>


### Loading data files isto the tables directories on HDFS
<pre>
hdfs dfs -cp s3a://training-coursera2/tbm_sf_la/central/hourly_central.csv /user/hive/warehouse/dig.db/tbm_sf_la_central/
</pre>

<pre>
hdfs dfs -cp s3a://training-coursera2/tbm_sf_la/north/hourly_north.csv /user/hive/warehouse/dig.db/tbm_sf_la_north/
</pre>

<pre>
hdfs dfs -cp s3a://training-coursera2/tbm_sf_la/south/hourly_south.tsv /user/hive/warehouse/dig.db/tbm_sf_la_south/
</pre>

### Combining individual tables into a single table including all data, CTAS method is used

<pre>
CREATE EXTERNAL TABLE dig.tbm_sf_la AS
    SELECT tbm, year, month, day, hour, dist, lon, lat
        FROM dig.tbm_sf_la_central
    UNION ALL
    SELECT tbm, year, month, day, hour, dist, lon, lat
        FROM dig.tbm_sf_la_north
    UNION ALL
    SELECT tbm, year, month, day, hour, dist, lon, lat
        FROM dig.tbm_sf_la_south;
</pre>

### Results
Number of rows for each TBM
<pre>
SELECT tbm, COUNT(*) AS num_rows FROM dig.tbm_sf_la GROUP BY tbm ORDER BY tbm;
</pre>

<pre>
(TBM)               (num_rows)
Bertha II           91619
Diggy McDigface     93163
Shai-Hulud          94237
</pre>

<pre>
hello, this is
   just an     example
....
</pre>

The action of every agent <br />
  into the world <br />
starts <br />
  from their physical selves. <br />
