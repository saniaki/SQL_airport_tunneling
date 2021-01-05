# SQL_airport_tunneling
SQL queries for analyzing and managing big data available on Amazon Web Service coulde

## SQL query engine: Impala

## Phase 1: Finding two most approperiate airports to construct a high-speed rail tunnel

Criteria: 
* two airports must be between 300 and 400 miles apart
* at least 5000 average flights per year between airports, in each direction
* airport pair with largest total number of seats on the planes between them

What to report:
* pair of airport with above cretiria
* average arrival delay for flights between them
* average flight distance (in miles) between airports, in each direction
* average number of flights per year between airports, in each direction
* average yearly total number of seats (passengers) on the planes between the two airports

### Examining data
..........


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
First 7 rows of the results table:

<p align="center">
<img  align="center" src="https://github.com/saniaki/Digit-Image-Classifier/blob/master/Images/Sample_output_3.jpg" width="700"/>


First and second directions:

<pre>
Three-letter airport code for origin	        SFO	LAX
Three-letter airport code for destination	LAX	SFO
Average flight distance in miles	        337	337
Average number of flights per year	        14712	14540
Average annual passenger capacity	        1996597	1981059
Average arrival delay in minutes	        10	14
</pre>

Sanity checks: 
* Average flight distance is expected to be the same for each pairs
* Average flight distance for all rows in the results table should be between 300 and 400
* Average flight distance for all rows in the results table should be between 300 and 400


## Phase 2: Finding two most approperiate airports to construct a high-speed rail tunnel







<pre>
hello, this is
   just an     example
....
</pre>

The action of every agent <br />
  into the world <br />
starts <br />
  from their physical selves. <br />
