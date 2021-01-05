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


## SQL query
SELECT origin, dest,
    ROUND(CAST(COUNT(*) AS decimal)/10) AS avg_num_flights,
    SUM(seats) AS tot_num_seats,
    ROUND(CAST(SUM(seats) AS decimal)/10) AS avg_annual_num_pass,
    ROUND(AVG(distance)) AS avg_flight_dist,
    ROUND(AVG(arr_delay)) AS avg_arr_delay_minutes
FROM fly.flights AS f
LEFT OUTER JOIN fly.planes AS p
    ON f.tailnum = p.tailnum
WHERE distance BETWEEN 300 AND 400
GROUP BY origin, dest
    HAVING avg_num_flights >= 5000  
ORDER BY tot_num_seats DESC;

