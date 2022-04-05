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

A "classic" Divvy station has an electronic kiosk for people to walk up to and pay to check out a bike; it also has several docks connected to the kiosk. The alternative station type is one where only electric Divvy bikes can be docked and has no kiosk. 

````
select 
	s.name,
	s.agency,
	count(distinct d.name),
	jsonb_agg(distinct d.name)
from m_divvy_stations AS d, stations_entrances AS s
where st_dwithin(s.geom, d.geom, 250)
and station_type = 'classic'
group by s.name, lines, agency;
````

- 50 feet: 14
- 100 feet: 34
- 150 feet: 68 (51 CTA, 17 Metra)
- 200 feet: 91
- 250 feet: 105 (74 CTA, 31 Metra)
- 300 feet: 122 (90 CTA, 32 Metra)
