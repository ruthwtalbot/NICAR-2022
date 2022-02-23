# Combining and Analyzing Data in QGIS

## Analyze Grocery Store Data

### Load in Shapefiles and Text Data

#### Load Shapefiles

- In QGIS, go to Layer > Add Layer > Add Vector Layer
- In the pop-up window that opens:
  - Source Type = File should be the default, but if it isn’t, select that. 
  - In Vector Dataset(s) under 'Source', open the file finder (3 dots to the right)
    - In the data provided, select Atlanta_Metro_Census_Tracts > Atlanta_metro_area.shp.
  - Once the .shp file is selected, click ‘Add’. 
- Repeat this process with the Grocery_Stores_Atlanta_Metro shapefile. 
- Click ‘Close’ close the window and view the data. You should see both datasets load in the view and their names listed in the 'Layers' section to the bottom left.

Note: You should be able to load shapefile data by selecting any of the main 3 files in the folder (.shp, .dbf, .shx). Explanations of file types are below for those interested!

- .shp: contains the geometry for all features.
- .shx: indexes the geometry.
- .dbf: stores feature attributes in a table.

These should be in every shapefile, you may also see:
- .prj: contains information on projection format including the coordinate system and projection information
- .sbn and .sbx: spatial index of the features.
- .xml: geospatial metadata in XML format

#### Load Text Data
- Go to Layer > Add Layer > Add Delimited Text Layer 
- Go to the Income folder in the data and select the .csv in the folder named 'income.csv' and click ‘Open’
- Under Geometry Definition, make sure ‘No geometry’ is selected. (Census data will have GEOIDs we can use to join with geometric data, but it does not contain geometry itself.)
- Repeat the process for ‘Race data’

### Join Data

#### Change field values in QGIS
To join data with geometries to data without geometries, we need a field to join across datasets. 

To inspect a datasets attributes right click on the dataset in the Layers detail (bottom left) > Open Attribute Table

Open the attribute table for either census dataset (Income or Race) and look at the column GEO_ID. It should be in the format 1400000US13001950100, with the prefix ‘1400000US’. The only meaningful digits for us are the last 11. The first two digits, 13, represents the fips code for Georgia, the next 3 digits indicate the county, and the last 6 indicate census tracts/block etc. 

Open the attribute table for the Census Tract shapefile, and you’ll see the GEOID10s are in the format 13001950100 (missing the prefix), so we need to change them to match. To do that, follow steps below:
- Select the Field calculator in the top menu (looks like an abacus)
- Set 'Output Field' = to geoid or some similar name
- Set 'Output Field type' = to Text(string)
- Set 'Output Field length' = 20 (must be large enough to accomodate a 20-letter geoid)
- In the expression box (empty text box below), type '1400000US' + "GEOID10"
  - This tells the Field Calculator to create a new field of type string, but adding the 14... prefix to each GEOID 
- Click 'Ok'

Note: you can do all sort of expressions in the field calculator. The box on the right hand side is a great starting point to explore!

#### Join data on Field
- Double click on the Atlanta_metro_area in the Layers area
- Click on the ‘Joins’ option on the left-hand side of the pop up
- Click the green plus button on the bottom left
- Set Join Layer = to either Income or Race data
- Set Join field = GEO_ID (or whatever the geoid field is)
- Set Target field = the new field in we created in the step above 
- Click 'Ok', it might take a second but you should see a join appear in the Joins box

### Visualize the Data (Optional)
- Double click the Atlanta Metro Tract again
- Go to ‘Symbology’
- In 'Value', select the column you want to color by. 
- In the dropdown at the top, select ‘Graduated’ (default should be ‘Single Symbol’)
- On the bottom right, increase the number of ‘Classes’ to somewhere around 10. 
 - Check the ranges generated are expected given whatever the data is.  
- Click 'Ok'

This will tell QGIS to color the geodata by whatever property you select, giving you a way to eyeball the data, looking for possible trends.

### Count Number of Grocery Stores in Each Tract
- Go to Vector > Analysis Tools > Count Points in Polygon
- Select 'Polygons' = to the Atlanta_metro_area layer
- Select the Points = to the grocery store layer
- Make sure 'Count Field Name' = NUMPOINTS
  - This tells QGIS to sum by # of points in each polygon/tract
- Click ‘Run’
- You should see a new layer called ‘Count’ in the layers area
- Right click on ‘Count’ and go to Export > Save Features As and save the file as a .csv

You should now have a .csv with GEOID, median income, % white, population, and the # grocery stores for every tract in the Atlanta Metro Area! Analyze however you prefer (but we can talk you through some basic ideas if you'd like :)




## Optional Next Exercise -- Area Analysis

Sometimes the geodata we want to analyse isn't lat/lngs, it's area. For instance, we might want to look at the % of each census tract in the Atlanta Metro area that is public park. 

- First, load in the Greenspace data set from the data folder, then follow the instructions below. 

### Calculate Area of Census Tracts
First, we need to add in an area field to each census tract, so we know the area covered by each tract

- Open the Field Calculator for the Atlanta Census Tract layer
- Set 'Output Field' = to tract_area or some similar name
- Set 'Output Field type' = to Decimal number 
- Put the expression $area in the text field
- Click 'Ok'

This might take a second, because it's calcuating the area covered by each tract.

### Split Up Parks by Tract

Some parks' boundaries cross census tracts, so we need to break apart the parks that are in many tracts before calculating area.

- Vector -> Geoprocessing tools -> Intersect 
- Input Layer = Atlanta Metro Area Tracts
- Overlay Layer = Greenspace Layer
- Click ‘Run’ (this may take a minute)

This will produce a layer called ‘Intersection’ which will contain the overlap between the parks and the tracts, meaning it will split the parks by tract boundaries AND add tract IDs to the new, split up park data. 


### Calculate Area of Parks within Tracts

- Open the Field Calculator for the 'Intersection' layer
- Set 'Output Field' = to park_area or some similar name
- Set 'Output Field type' = to Decimal number 
- Put the expression $area in the text field
- Click 'Ok'

This might take a second.

### Merge Data
- Go to Vector > Data Management > Merge Layers
- Select the Atlanta Metro Area Tracts layer and the Intersection layer, click ‘Run’
- Export the result to csv, where we will use pivot tables to join by census tract and divide by area.





