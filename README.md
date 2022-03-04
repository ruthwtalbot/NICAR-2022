# Combining and Analyzing Data in QGIS

Link to slides: https://docs.google.com/presentation/d/19zR6xYUu7FWxby94WDf7TaAuxhOwnoAo-TAVW17DZwE/edit

## Analyze Grocery Store Data

### Load in Shapefiles and Text Data

#### Load Shapefiles

- In QGIS, go to Layer > Add Layer > Add Vector Layer
- In the pop-up window that opens:
  - Source Type = File should be the default, but if it isn’t, select that. 
  - In Vector Dataset(s) under 'Source', open the file finder (3 dots to the right)
    - In the data provided, select Georgia_Census_Tracts > Georgia_Census_Tracts.shp.
  - Once the .shp file is selected, click ‘Add’. 
- Repeat this process with the Grocery_Stores_Atlanta_Metro shapefile. 
- Click ‘Close’ to close the window and view the data. You should see both datasets load in the view and their names listed in the 'Layers' section to the bottom left.

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
- Browse to the Income folder in the data and select the .csv in the folder named 'income.csv' and click ‘Open’
- Under Geometry Definition, make sure ‘No geometry’ is selected. (Census data will have GEOIDs we can use to join with geometric data, but it does not contain geometry itself.)
- Repeat the process for ‘Race data’

Congratulations! You should now have four layers in the QGIS layers panel.

### Join Data

#### Filter Shapefile by ID to Get Counties of Interest
- For the Georgia_Census_Tracts, go to Attribute Table and click on 'Select Features by Expression" (looks like an E and a square)
- In the expression area, paste the formula below

    array_contains( array('13057', '13063', '13067', '13089', '13097', '13113', '13121', '13135', '13151', '13247'), left( "GEOID", 5))
    
- Click 'Select Features' on the bottom right, and then hit 'Close'
  - You should see the tracts within those counties highlighted
- Right click on the file and go to Export > Export Selected Features As 
  - Important: select the 'Selected' Features option, which is the second option, not just export features as 
- And save files as Atlanta_Metro_Census_Tracts or something similar. That should now appear in your layers bar.
- Unselect or Remove Georgia_Census_Tracts, we won't be using it anymore.

#### Change field values in QGIS
To join data with geometries to data without geometries, we need a field to join across datasets. 

To inspect a dataset's attributes, right click on the dataset in the Layers panel (bottom left) > Open Attribute Table

Open the attribute table for either census dataset (Income or Race) and look at the column GEO_ID. It should be in the format 1400000US13001950100. Notice how they all have the prefix ‘1400000US’ — the only meaningful digits for us are the last 11: The first two digits, `13`, represents the FIPS code for Georgia, the next 3 digits indicate the county, and the last 6 indicate census tracts/block etc. 

Open the attribute table for the Atlanta_Metro_Census_Tracts shapefile, and you’ll see the GEOIDs are in the format 13001950100 (missing the prefix), so we need to change them to match. To do that, follow steps below:
- Select the Field calculator in the top menu (looks like an abacus)
- Set 'Output Field' = `id` or some similar name
- Set 'Output Field type' =  Text(string)
- Set 'Output Field length' = 20 (must be large enough to accomodate a 20-letter id)
- In the expression box (empty text box below), type '1400000US' + "GEOID"
  - This tells the Field Calculator to create a new field of type string, but adding the `1400000US` prefix to each GEOID 
- Click 'Ok'

Note: you can do all sort of expressions in the field calculator. The box on the right-hand side is a great starting point to explore!

#### Join data on Field
- Double click on the Atlanta_metro_area in the Layers area
- Click on the ‘Joins’ option on the left-hand side of the pop up
- Click the green plus button on the bottom left
- Set Join Layer = to either Income or Race data
- Set Join field = GEO_ID (or whatever the id field is)
- Set Target field = the new field in we created in the step above, `id` 
- Click 'Ok'. It might take a second but you should see a join appear in the Joins box

### Visualize the Data (Optional)
- Double click the Atlanta_Metro_Census_Tracts layer again
- Go to ‘Symbology’
- In 'Value', select the column you want to color by. 
  - If you don't see the column you want appear, it may be the wrong type. To convert from string to number, click the E sign next to the Value dropdown and paste   to_int( "income_S1903_C03_001E") into the expression area.
- In the dropdown at the top, select ‘Graduated’ (default should be ‘Single Symbol’)
- On the bottom right, increase the number of ‘Classes’ to somewhere around 10. 
- Check the ranges generated are expected given whatever the data is.  
- Click 'Ok'

This will tell QGIS to color the geodata by whatever property you select, giving you a way to eyeball the data, looking for possible trends.

### Count Number of Grocery Stores in Each Tract
- Go to Vector > Analysis Tools > Count Points in Polygon
- Select 'Polygons' = to the Atlanta_metro_area layer
- Select the Points = to the Grocery_Stores_Atlanta_Metro layer
- Make sure 'Count Field Name' = NUMPOINTS
  - This tells QGIS to sum by # of points in each polygon/tract
- Click ‘Run’
- You should see a new layer called ‘Count’ in the layers area
- Right click on ‘Count’ and go to Export > Save Features As and save the file as a .csv

You should now have a .csv with GEOID, median income, % white, population, and the # grocery stores for every tract in the Atlanta Metro Area! Analyze however you prefer — but we can talk you through some basic ideas :) 

Quartile functions:

=SUM(C2:INDIRECT("C"&ROUND(738/4, 0)))

=SUM(INDIRECT("C"&ROUND(738/4, 0)+1):INDIRECT("C"&ROUND(738/4, 0) * 2))

=SUM(INDIRECT("C"&ROUND(738/4, 0)*2+1):INDIRECT("C"&ROUND(738/4, 0) * 3))

=SUM(INDIRECT("C"&ROUND(738/4, 0)*3+1):INDIRECT("C"&ROUND(738/4, 0) * 4))

## Optional Next Exercise -- Area Analysis

Sometimes the geodata we want to analyse isn't lat/lngs, it's area. For instance, we might want to look at the % of each census tract in the Atlanta Metro area that is public park. 

- First, load in the Greenspace data set from the data folder, then follow the instructions below. 

### Calculate Area of Census Tracts
We need to add in an area field to each census tract, so we know the area covered by each tract

- Open the Field Calculator for the Atlanta Census Tract layer
- Set 'Output Field' =  `tract_area` or some similar name
- Set 'Output Field type' =  Decimal number 
- Type the expression `$area` in the text field
- Click 'Ok'

This might take a second, because it's calcuating the area covered by each tract.

### Calculate Difference between Tract and Parks
- Go to Vector > Geoprocessing Tools > Difference
- Select the Input Layer = Atlanta Census Tract layer and the Overlay = Greenspace
- Click 'Run'

You should see a new layer called 'Difference' outputted, unselect Greenspace and Atlanta Tract Area to see it more clearly. This contains the parts of census tracts NOT covered in park, but still contain the census tract total area as a field.

### Calculate New Area of Census Tracts Without Parks
Do the same calculation we did above, just with these new polygons.

- Open the Field Calculator for the Difference layer
- Set 'Output Field' =  `tract_area` or some similar name
- Set 'Output Field type' =  Decimal number 
- Type the expression `$area` in the text field
- Click 'Ok'

### Calculate the % of Total Area
- Open the Field Calculator again
- Set 'Output Field' =  `percent` or some similar name
- Set 'Output Field type' =  Decimal number  
- Enter the expression below and click 'Ok'

    1 -  "area_part" / "area_full" 
    
This will add a field with the percent of tract covered by greenspace.

Export and analyze!





