# SQL-PROJECT-WITH-POSTGRES
DATA_BREACH 

ADNR Corporate had a data breach. Company wants us to find who did it.
What we know; It's an inside job and data breach happened on 2020-06-23. 
We have 2 databases: I.morv(rides,users,vehicles info), II.morv_employees(employees info).
ADNR Corp;
location= Newyork
Latitude=-74.997 to -74.9968
Longitude= 40.5 to 40.6 

1. We cheack during that time any car that was in that area; 

SELECT * FROM vehicle_location_histories AS vlh
WHERE
     city = 'new york' AND 
     lat between -74.997 AND -74.9968 AND 
     long between 40.5 AND 40.6  AND 
     vlh.timestamp ::date = '2020=06-23'::date
ORDER BY long;

--428 rides occured

CREATE VIEW suspected_rides AS
SELECT * FROM vehicle_location_histories AS vlh
WHERE
     city = 'new york' AND 
     lat between -74.997 AND -74.9968 AND 
     long between 40.5 AND 40.6  AND 
     vlh.timestamp ::date = '2020=06-23'::date
ORDER BY long;

2.Based on those rides, interrogate the drivers; 

SELECT DISTINCT r.vehicle_id 
FROM suspected_rides AS sr
JOIN rides AS r ON r.id=sr.ride_id;

--89 drivers did 428 rides

3.Find information about drivers;

SELECT DISTINCT r.vehicle_id,u.name AS "owner name", u.address, v.status,v.current_location 
FROM suspected_rides AS sr 
JOIN rides AS r ON r.id=sr.ride_id 
JOIN vehicles AS v ON v.id=r.vehicle_id
JOIN users AS u ON u.id=v.owner_id ;

--89 spesific drivers that were in or around.This information won't lead us anywhere.It's not drivers, it must be riders who did it!

4. Which riders took those rides ;
   
SELECT DISTINCT r.vehicle_id ,u.name as "rider name" , u.address 
FROM suspected_rides AS sr
JOIN rides AS r ON r.id=sr.ride_id
JOIN users AS u ON u.id=r.rider_id ;

--109 riders, there is nothing conclusive. It's not the riders and It's not the drivers! 

LAST CLUE: THIS WAS AN INSIDE JOB THAT SOMEONE ON THE OUTSIDE WAS HELPED BY AN EMPLOYEE ON THE INSIDE. 2 people colabrated. 

5. We will cross-reference data between 2 databases

CREATE EXTENSION IF NOT EXISTS dblink;

6. Create same name formation;
   
CREATE VIEW suspect_rider_names AS 
SELECT DISTINCT 
     split_part(u.name,' ',1) AS "first_name",
     split_part(u.name,' ',2) AS "last_name"
FROM suspected_rides AS vlh 
JOIN rides AS r ON r.id=vlh.ride_id
JOIN users AS u ON u.id=r.rider_id ;

7. Correlate 2 dabases names;

SELECT DISTINCT 
     concat(t1.first_name, ' ' , t1.last_name) AS "employee",
     concat(u.first_name,' ' , u.last_name) AS "rider"
FROM dblink ('host=localhost user=postgres password=postgres dbname=morv_employees', 'select first_name, last_name from employees')
AS t1(first_name name , last_name name )
JOIN suspect_rider_names AS u ON t1.last_name=u.last_name ;

--11 names from employees and suspected rider names. 

It looks like they have the same last name!
