# Train Station Info

The location of train station entrances in Chicago, for Chicago Transit Authority 'L' and Metra commuter rail stations, created by [Yonah Freemark](http://www.thetransportpolitic.com). 

## License

This data is licensed CC0, public domain. 

## Chicago train stations

The entrance locations are not well-defined and discussion about new or modified entrances is welcome. 

### CTA

This includes all 144 CTA stations, having 383 entrances. The [CTA Rail Stations datasets](https://data.cityofchicago.org/browse?q=cta%20rail%20stations&sortBy=relevance) on the Chicago open data portal has a non-specific point representing the station, where the point often represents the centroid of the platform or station house. 

This dataset includes proposed entrances at under-construction stations: Wilson (Red & Purple Lines; there will be 4 entrances), Washington-Wabash (actual entrances are estimated), and 95th Street (Red Line, 3 entrances).

#### Download
GeoJSON: [View CTA station entrances](https://github.com/ChicagoCityscape/tod-data/blob/master/stations_cta/cta_entrances.json) or [download them](https://github.com/ChicagoCityscape/tod-data/raw/master/stations_cta/cta_entrances.json), or [as CSV](https://github.com/ChicagoCityscape/tod-data/blob/master/stations_cta/cta.csv).

### Metra

This includes 249 entrances at 90 Metra stations in Chicago and within 1,200 feet of Chicago. This also includes the upcoming Peterson and Auburn Gresham stations.

#### Download
GeoJSON: [View Metra station entrances](https://github.com/ChicagoCityscape/tod-data/blob/master/stations_metra/metra_entrances.json) or [download them](https://github.com/ChicagoCityscape/tod-data/raw/master/stations_metra/metra_entrances.json), or [as CSV](https://github.com/ChicagoCityscape/tod-data/blob/master/stations_metra/metra.csv).

## ogr2ogr cheatsheet

Yonah created the data using Google Maps, because it's easy to drop a point marker on the station entrance by looking at satellite imagery. The data needs to be converted from KML to CSV (to view as a spreadsheet), and to PostGIS (so it can be used as a database). 

### convert KML to shapefile
"metra.shp" is the destination file, and "Metra entrances.kml" is the source file. It doesn't work with KMZ (compressed KML) files – you must convert these using Google Earth. 

````
ogr2ogr -f "ESRI Shapefile" metra.shp "Metra entrances.kml"
````

### convert KML to CSV
The "-lco GEOMETRY=AS_XY" flag and argument is required to output the geometry of the KML to CSV. Otherwise geometry data is discarded upon conversion to CSV. 

````
ogr2ogr -f "CSV" metra.csv "Metra entrances.kml" -lco GEOMETRY=AS_XY
````

### convert KML to PostGIS
This will create a new table called "metra entrances". The SRID of the "wkb_geometry" field will be EPSG:4326, which is the standard for KML. 
````
ogr2ogr -f "PostgreSQL" PG:"host=hostname port='5432' dbname='database_name' user='username' password='password'" "Metra entrances.kml"
````

### convert PostGIS to GeoJSON
````
ogr2ogr -f GeoJSON cta.geojson PG:"host=hostname port='5432' dbname='database_name' user='username' password='password'" -sql "select name,'CTA' as agency,description,stationid,st_force2d(geom_4326) from cta" -t_srs "epsg:4326"
````
Note: The ````st_force2d()```` function is used to convert a three-point geometry (XYZ) to a two-point geometry (XY) because when you upload KML to PostGIS it creates an XYZ (three point) geometry (where Z is elevation), and for some reason this cannot be exported to GeoJSON.

### convert CSV to PostGIS
This simply uploads a CSV file (with or without X/Y or latitude/longitude fields) to PostgreSQL. If there are coordinate fields they will not be recognized by PostGIS as geometry fields – you'll have to create them. The table will be called "cta". 
````
ogr2ogr -f "PostgreSQL" PG:"host=hostname port='5432' dbname='database_name' user='username' password='password'" "cta.csv"
````
Use this query to create a geometry field from the X/Y fields:
````
BEGIN;
SELECT AddGeometryColumn ('cta','geom_4326',4326,'POINT',2);
UPDATE cta SET geom_4326 = ST_SetSRID(ST_MakePoint(x::numeric, y::numeric),4326);
COMMIT;
````