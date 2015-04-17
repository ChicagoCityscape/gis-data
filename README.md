# Train Station Info

The location of train station entrances in Chicago, for Chicago Transit Authority 'L' and Metra commuter rail stations, created by [Yonah Freemark](http://www.thetransportpolitic.com). 

## License

This data is licensed CC0, public domain. 

## Data

### Chicago train stations

The entrance locations are not well-defined and discussion about new or modified entrances is welcome. 

#### CTA

This includes all 144 CTA stations, having 383 entrances. The [CTA Rail Stations datasets](https://data.cityofchicago.org/browse?q=cta%20rail%20stations&sortBy=relevance) on the Chicago open data portal has a non-specific point representing the station, where the point often represents the centroid of the platform or station house. 

This dataset includes proposed entrances at under-construction stations: Wilson (Red & Purple Lines; there will be 4 entrances), Washington-Wabash (actual entrances are estimated), and 95th Street (Red Line, 3 entrances).

##### Download
GeoJSON: [View CTA station entrances](https://github.com/ChicagoCityscape/tod-data/blob/master/stations_cta/cta.geojson) or [download them](https://github.com/ChicagoCityscape/tod-data/raw/master/stations_cta/cta.geojson), or [as CSV](https://github.com/ChicagoCityscape/tod-data/blob/master/stations_cta/cta.csv).

#### Metra

This includes 249 entrances at 90 Metra stations in Chicago and within 1,200 feet of Chicago (among others, including stations in Blue Island). This also includes the upcoming Peterson and Auburn Gresham stations. The ````station_id```` field is derived from the Metra stations GIS dataset on the Chicago open data portal. There were several stations with ````0```` in this field; those have been changed to 5-digit arbitrary IDs starting with 99. 

##### Download
GeoJSON: [View Metra station entrances](https://github.com/ChicagoCityscape/tod-data/blob/master/stations_metra/metra_entrances.json) or [download them](https://github.com/ChicagoCityscape/tod-data/raw/master/stations_metra/metra_entrances.json), or [as CSV](https://github.com/ChicagoCityscape/tod-data/blob/master/stations_metra/metra.csv).

### Other data

#### MPEA taxing boundary 
The Metropolitan Pier and Exposition Authority (also known as MPEA and McPier) owns Navy Pier and McCormick Place and imposes an additional sales tax on retail food and beverages in a large boundary containing all of the Central Business District (and then some). 

Its boundaries, drawn by Yonah Freemark, are:

Lake Michigan area boundaries:
- 150 feet north of the north side of Diversey Avenue
- 150 feet west of the west side of Ashland Avenue
- South side of the Stevenson Expressway [This is an interesting boundary because it's an elevated highway, but there are businesses underneath that I think this boundary would include.]
- Shoreline of Lake Michigan, including Navy Pier and all other improvements fixed to land, docks, or piers

Midway:
- South side of 55th St
- East side of Central Ave
- North side of 63rd St
- West side of Cicero Ave

O'Hare: All of ZIP code 60666

##### Download
* [McPier boundary as GeoJSON](https://github.com/ChicagoCityscape/tod-data/blob/master/mcpier/mcpier.geojson)
* [McPier boundary as Shapefile, EPSG:4326](https://github.com/ChicagoCityscape/tod-data/blob/master/mcpier/mcpier_shapefile_4326.zip)
* [McPier boundary as Shapefile, EPSG:3435](https://github.com/ChicagoCityscape/tod-data/blob/master/mcpier/mcpier_shapefile_3435.zip)
* [McPier boundary as KML](https://github.com/ChicagoCityscape/tod-data/blob/master/mcpier/mcpier.kml)

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
ogr2ogr -f GeoJSON -lco COORDINATE_PRECISION=6 cta.geojson PG:"host=hostname port='5432' dbname='database_name' user='username' password='password'" -sql "select name,'CTA' as agency,description,stationid,st_force2d(geom_4326) from cta" -t_srs "epsg:4326"
````
Note: The ````st_force2d()```` function is used to convert a three-point geometry (XYZ) to a two-point geometry (XY) because when you upload KML to PostGIS it creates an XYZ (three point) geometry (where Z is elevation), and for some reason this cannot be exported to GeoJSON.

The "-lco COORDINATE_PRECISION=6" argument is specific to GeoJSON and it means the coordinates will have a maximum of 6 digits to the right of the decimal point. 

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

### simplify a shapefile
Run this command in the directory where you have the "Chicago-Parcels.shp" file stored. On Mac, the easiest way to do that is to drag the directory icon from the Finder onto the Terminal icon in the Dock. 

The number after the ````-simplify```` argument is the tolerance you want. There's no magic number, or even a rule of thumb. The best number depends on the complexity (number of vertices and edges) of your shape. Too high of a number and you can simplify the shape to non-existence (it will disappear).

For our purpose, we need to simplify the geometry of the shapes of all the parcels in the City of Chicago to display on a web map (starting at 99.7 MB). Tested (without inspection of shape integrity):
* .0009: 4.5% file size reduction
* .009: 12.0% reduction
* .09: 16.0% reduction
* .9: 16.9% reduction
````
ogr2ogr -f "ESRI Shapefile" parcels_chicago_simplified.shp -simplify 0.0009 "City-Parcels-New.shp"
````
### convert PostGIS to simplified shapefile and re-project
````
ogr2ogr -f "ESRI Shapefile" parcel_chicago_4326.shp PG:"host=localhost port='5432' dbname='database' user='username' password='password' " -sql "SELECT pin14, ST_SimplifyPreserveTopology(geom, 0.9) as geom FROM parcel_chicago" -t_srs "epsg:4326"
````
