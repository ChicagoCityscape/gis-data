I'm counting how many Divvy stations are at CTA or Metra rail station entrances. 

## First, count the features
673 Divvy stations

````
select count(*) from m_divvy_stations
where station_type = 'classic';
````

125 CTA stations (in Chicago)
````
select 
	s.name, 
	agency,
	count(*) 
from stations_entrances AS s, chicagoboundary AS c
where station_id is not null
and c.gid = 1
and st_intersects(c.geom, s.geom)
and agency = 'CTA'
group by s.name, lines, agency;
````

72 Metra stations (in Chicago)
````
select 
	s.name, 
	agency,
	count(*) 
from stations_entrances AS s, chicagoboundary AS c
where station_id is not null
and c.gid = 1
and st_intersects(c.geom, s.geom)
and agency = 'Metra'
group by s.name, lines, agency;
````

## Count how many CTA/Metra rail stations have a nearby Divvy station


## Count how many Divvy stations are near a CTA/Metra rail station
14 Divvy stations within 50 feet of a CTA/Metra rail station entrance (9 CTA, 5 Metra)
*50 feet is one of the 3 #SimpleBikeParking rules*
````
select distinct
	s.name as station,
	s.agency,
	d.name AS divvy
from m_divvy_stations AS d, stations_entrances AS s
where st_dwithin(s.geom, d.geom, 50)
and station_type = 'classic';
````

34 Divvy stations within 100 feet of a CTA/Metra rail station entrance (25 CTA, 9 Metra)
````
select distinct
	s.name as station,
	s.agency,
	d.name AS divvy
from m_divvy_stations AS d, stations_entrances AS s
where st_dwithin(s.geom, d.geom, 100)
and station_type = 'classic';
````

70 Divvy stations within 150 feet of a CTA/Metra rail station entrance (51 CTA, 19 Metra)
````
select distinct
	s.name as station,
	s.agency,
	d.name AS divvy
from m_divvy_stations AS d, stations_entrances AS s
where st_dwithin(s.geom, d.geom, 150)
and station_type = 'classic';
````

95 Divvy stations within 200 feet of a CTA/Metra rail station entrance (66 CTA, 29 Metra)
````
select distinct
	s.name as station,
	s.agency,
	d.name AS divvy
from m_divvy_stations AS d, 200 AS s
where st_dwithin(s.geom, d.geom, 150)
and station_type = 'classic';
````
