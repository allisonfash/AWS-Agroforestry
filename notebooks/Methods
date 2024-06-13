

## Repository Structure

- `/data`: Contains the configuration files and boundary data.
- `/notebooks`: Contains the Jupyter notebook for the analysis.
- `/scripts`: Contains the Python scripts for data processing and analysis.
- `README.md`: Project overview and instructions.
- `requirements.txt`: List of required Python packages.

## Methods and Functions

### Data Processing and Analysis

Here is an overview of the methods and functions used in the project:

```python
import os 
import requests as rq
import pandas as pd
import geopandas as gpd
import folium
import geojson
import warnings
import matplotlib.pyplot as plt
from matplotlib.pyplot import imshow
import rasterio as rio
import rioxarray
from rioxarray import merge
from rasterio.session import AWSSession
from rasterio.plot import show as rio_show
from rasterio.windows import from_bounds
from shapely.geometry import Polygon
import hvplot.xarray
import hvplot.pandas
import numpy as np
import boto3
warnings.filterwarnings("ignore")

# Load the area of interest (AOI) boundary
aoi_gdf =  gpd.read_file(os.path.join('ucayali_boundary.geojson')) # aoi geopandas dataframe
aoi_geojson = aoi_gdf.__geo_interface__

# Function to fetch data from STAC server
def fetch_stac_server(query):
    '''
    Queries the stac-server (STAC) backend.
    query is a python dictionary to pass as json to the request.
    '''
    search_url = f"https://landsatlook.usgs.gov/stac-server/search"
    query_return = rq.post(search_url, json=query).json()
    error = query_return.get("message", "")
    if error:
        raise Exception(f"STAC-Server failed and returned: {error}")
        
    if 'code' in query_return:
        print(query_return)   
    else:
        print(f"{len(query_return['features'])} STAC items found")
        return query_return['features']

# Function to send STAC query
def send_STAC_query(limit=20, collections=None, intersects=None, datetime=None, bbox=None):
    '''
    This function helps to create a simple parameter dictionary for querying 
    the Landsat Collection 2 Level 2 Surface Reflectance feature in the STAC Server.
    It prints the parameter dictionary and returns the query results.
    '''
    params = {}
    if limit is not None:
        params['limit'] = limit
    
    if collections is not None:
        params['collections'] = collections
        
    if intersects is not None:
        params['intersects'] = intersects
        
    if datetime is not None:
        params['datetime'] = datetime
        
    if bbox is not None:
        params['bbox'] = bbox
        
    print(f"Using parameters: {params}")
    return fetch_stac_server(params)
