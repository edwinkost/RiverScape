# Introduction

[//]: <> (General idea of this notebook.)
This notebook facilitates using the RiverScape Python package (Straatsma and Kleinhans, 2018) to parameterize and position landscaping measures and update the input data for the two-dimensional (2D) flow model Delft3D Flexible Mesh (DFM).

[//]: <> (Study area.)
For the current notebook version, we would use the River Waal, which is the main distributary of the River Rhine in the Netherlands.

[//]: <> (TODO: Add Fig. 3 of Straatsma and Kleinhans, 2018)




## Requirement (Python modules/packages)

To run this notebook, please import the following Python modules.


``` code
%load_ext autoreload
%autoreload 2

# import standard modules
import os
import sys
import string
import subprocess
import time
import math
import pprint as pp

# import required modules/packages (which may require installation)
import numpy as np
import pandas as pd
import geopandas as gpd
from scipy.spatial import Delaunay
from shapely.geometry import MultiLineString
from shapely.ops import cascaded_union, polygonize


from collections import OrderedDict
import pcraster as pcr
```

Please also make sure that this notebook file is in the same folder as the RiverScape Python module files that must be loaded.
You can then import the RiverScape Python module:

``` code
from riverscape import *
import geoviews
geoviews.extension('bokeh')
```



# Input and output folders

In the following please define the input and output folders.


``` code
# Default locations
input_dir    = '/home/schmi109/development/projects/RiverScape/input_files/input' #os.getcwd()
output_dir   = os.getcwd()
scratch_dir  = os.path.join(output_dir, "tmp")

```
You may also want to set the folder, interactively

``` code
# input_dir  = select_directory()
```

``` code
# output_dir = select_directory()
```
``` code
# make scratch directory and go to this folder
pcrRecipes.make_dir(scratch_dir)
os.chdir(scratch_dir)

# print some statements about the folder locations:
msg = "The input folder is set on   : {}".format(input_dir)
print(msg)
msg = "The output folder is set on  : {}".format(output_dir)
print(msg)
msg = "The scratch folder is set on : {}".format(scratch_dir)
print(msg)
```


# Start processing/calculation

To start processing, please load the following cells in order to set and test some basic configuration.



``` code
# go to the scratch folder
pcrRecipes.make_dir(scratch_dir)
os.chdir(scratch_dir)

# set global option for pcraster such that length of cells is computed in true length of cells
pcr.setglobaloption('unittrue')

# set the pcraster clone map
current_dir = os.path.join(input_dir, 'reference_maps')
pcr.setclone(os.path.join(current_dir, 'clone.map'))
```





## Loading input files

By running the following cells, the input files would be read. These input files consist of the following attributes of current/existing condition.

* main_dike: current/existing river embankment properties, e.g. length, volume and height
* minemb: minor embankment properties, e.g. length, volume and height
* groynse: groyne properties, e.g. length, volume and height
* hydro: hydrodynamics (delft3d-fm) attributes, e.g. chezy, nikuradse, specific discharge, velocity, water depth, water level, etc.
* mesh: delft3d mesh
* axis: river attributes, e.g. location, radius, turning direction, velocity, water depth, water level, etc.
* geom: river geometry attributes, e.g. clone, dem, dist_to_main_dike, dist_to_groyne_field, dist_to_main_channel, flpl_width, flpl_narrow, flpl_wide, main_channel_width, river_side, shore_line
* lulc: land use and land cover attributes, e.g. backwaters, ecotopos, floodplain, groyne_field, main_channel, trachytopes, sections, winter_bed, real_estate_value

For further information about them, please check Menno and Kleinhans (2018), see e.g. their Table 2.


``` code
# change to the 'current_dir' (input data) for reading/importing input data
os.chdir(current_dir)


#TODO: Check why do we need this? It seems that os.path.join does not work?

# reading current/existing river embankment properties for main dikes, minor embankments and groynes
# - for each, this will return location, length, volume, and height
main_dike = msr.read_dike_maps(current_dir, 'main_dike')
minemb    = msr.read_dike_maps(current_dir, 'minemb')
groynes   = msr.read_dike_maps(current_dir, 'groyne')

# reading Hydrodynamics (delft3d-fm) attributes
# - chezy, nikuradse, specific discharge, velocity, water depth, water level, etc
hydro = msr.read_hydro_maps(current_dir)
# - hydrodynamic mesh
mesh = msr.read_mesh_maps(current_dir)

# reading RiverScape attributes
# - axis: location, radius, turning_direction, rkm, rkm_point, rkm_line, rkm_full
axis = msr.read_axis_maps(current_dir)
# - geometry: clone, dem, distance to main dike, distance to groyne, distance to main channel, floodplain widths, contiguous narrow floodplain, contiguous wide floodplain
geom = msr.read_geom_maps(current_dir)
# - land use land cover attributes
lulc = msr.read_lulc_maps(current_dir)

msg = "Files should be already read from: " + current_dir
print("\n")
print(msg)
```




### Listing attributes/variables

After succesfully reading the files/maps, you should be able to list the attributes/variables of main_dike, minemb, groynes, hydro, mesh, axis, geom, and lulc in the following.


``` code
for obj in ["main_dike", "minemb", "groynes", "hydro", "mesh", "axis", "geom", "lulc"]:
    print("\n")
    print(obj)
    print(vars()[obj].__dict__.keys())
```




### Visualizing existing attributes

You may want to check existing attributes by plotting the corresponding raster maps. You can use the plot function for that, for example to plot the digital elevation model type:

``` python
plot(geom.dem)
```



``` code
```




``` code
# plot river main channel width
plot(geom.main_channel_width)
```




``` code
# plot height of main_dike
plot(main_dike.height)
```



``` code
# plot flpl_wide IDs
plot(geom.flpl_wide)
```



<br>

## Initiating the River and Its Measures

Given the aforementioned attributes, the River Waal and its current measured would be initiated by executing the following cells.



``` code
# The River Wall is initiated based on the aforementioned given attributes.
waal = msr.River('Waal', axis, main_dike, minemb, groynes, hydro,
                         mesh, lulc, geom)
```


``` code
# Initiate the Measures for the River.
waal_msr = msr.RiverMeasures(waal)
```


# Configuration for the Measures

First, you can have a look at the current specification of the ecotope and trachytope classes present in the area.
The floodplain sections are depicted as well.
Ecotopes are ...
Trachytopes are ...

Open the maps with the following command.

``` code
plot_eco_trachy_sec()
```




## Side channel measure:

### Make your own side channel properties:


<!-- The following properties are the default/setting configuration to the Measures. -->
The measures are configured with certain properties.
You can inspect and change a few of them:



``` code
settings = measures_settings()
```




You may want to modify the side channel properties using the following interactive cell. Note that height and width are not true to scale.


``` code
channel_values = channel_properties()
```




Load the following cell to use your configuration.


``` code
settings['channel_width'] = channel_values.kwargs['width']
settings['channel_depth'] = channel_values.kwargs['depth']
settings['channel_slope'] = channel_values.kwargs['slope']
pp.pprint(settings)
waal_msr.settings = settings
```


### Selecting the mask/region:

Please select the areas where you want to perform this measure.






Please also give a label for this measure.



``` code
label = 'custom_label'
```

``` code
# Default mask and label
#label = 'everywhere'
#mask = pcr.boolean(1)

# TODO: Make interactive input for selecting mask for every measure.

# - some examples to select a limited mask region

#label = "large_sections"
#mask = pcr.ifthen(pcr.areaarea(waal.geom.flpl_wide) > 1e6, pcr.boolean(1.0))

# -- this will return floodplain with IDs < 5
#label = "edwin_sections"
#mask = pcr.ifthen(pcr.scalar(waal.geom.flpl_wide) < 5, pcr.boolean(1.0))

# - plot the chosen mask
#plot(mask)
#chosen_flpl_wide = pcr.ifthen(mask, waal.geom.flpl_wide)
#plot(chosen_flpl_wide)
```


``` code

selection = select_area(waal.geom.flpl_wide)

```
``` code
mask = generate_mask(waal.geom.flpl_wide, selection)

```

``` code
plot(mask)
```

``` code
chosen_flpl_wide = pcr.ifthen(mask, waal.geom.flpl_wide)
plot(chosen_flpl_wide)
```




### Implementing the measure

By running the following cell, the measure will be implemented.


``` code
# measure by side channel constrcution
# - this includes looping over floodplain IDs
chan_msr = waal_msr.side_channel_measure(settings, mask = mask, ID = label)
```


### Exploring the measure

You can explore the measure implemented by running the following plotting cell. Plotting may take a while before completed.


``` code
# plot/explore side channel measure
chan_msr.plot()
```



## Floodplain lowering measure:

For the floodplain lowering measure you need to select new areas where you want to introduce this measure.
First, give a new identifier:

``` code
ID = 'everywhere'
```

Then specify the sections that will form the new mask:


``` code
sections = pcr.readmap('flpl_sections.map')
selection = select_area(sections)
```

and generate the new mask

``` code
mask = generate_mask(sections, selection)
```

For checking purposes you could plot the mask map.


``` code

```

As before, you can now perform the measure and visualise the results.


``` code
# floodplain lowering measure
lowering_msr = waal_msr.lowering_measure(settings, mask=mask, ID=ID)
```


``` code
# plot/explore floodplain lowering measure
lowering_msr.plot()
```







## Groyne lowering measure:

Please set the ID/label for this measure and set the mask where you want to introduce this measure.


``` code
ID = 'everywhere'
```

For the groyne lowering measure you can specify a specific area, defined by the distance.
The minimum and maximum values should be between 867 and 960.

``` code
min_value = 890
max_value = 910
rkm = pcr.readmap('rkm_full.map')
mask = pcr.ifthen((rkm >= min_value) & (rkm <= max_value), pcr.boolean(1))
```

Plot the new mask:

``` code

```



``` code
# groyne lowering measure
groyne_low_msr = waal_msr.groyne_lowering_msr(settings, mask=mask, ID=ID)
```


``` code
# plot/explore groyne lowering measure
groyne_low_msr.plot()
```






## Minor embankment lowering measure:

Please set the ID/label for this measure and set the mask where you want to introduce this measure.


``` code
mask = pcr.boolean(1)
ID = 'everywhere'
```


``` code
# measure by minor embankment lowering
minemb_low_msr = waal_msr.minemb_lowering_msr(settings, mask=mask, ID=ID)
```


``` code
# plot/explore minor embankment lowering
minemb_low_msr.plot()
```





## Main dike raising measure:

Please set the ID/label for this measure and set the mask where you want to introduce this measure.


``` code
mask = pcr.boolean(1)
ID = 'everywhere'
```


``` code
# measure by main dike raising
main_dike_raise_msr = waal_msr.main_dike_raising_msr(settings, mask=mask, ID=ID)
```


``` code
# plot/explore main dike raising measure
main_dike_raise_msr.plot()
```





## Roughness smoothing measure:

Please set the ID/label for this measure and set the mask where you want to introduce this measure.


``` code
mask = pcr.boolean(1)
ID = 'everywhere'
```


``` code
# measure by roughness smoothing
smooth_msr = waal_msr.smoothing_measure(settings, mask=mask, ID=ID)
```


``` code
# plot/explore measure by roughness smoothing
smooth_msr.plot()
```






Saving measures to the disk
========================


``` code
# list of measures
msr_list = [groyne_low_msr, minemb_low_msr,
            main_dike_raise_msr, lowering_msr, chan_msr, smooth_msr]
# - preparing the directory
msr_root_dir = os.path.join(output_dir, 'measures_ensemble/maps')
pcrRecipes.make_dir(msr_root_dir)
for measure in msr_list:
    msr.write_measure(measure, msr_root_dir)
```
