
### Setting Up the Environment (Refer to "setting up enviroment.md for more detailed explanation):

1. **Clone the Repository**:
   - Open a terminal in Jupyter and run:
     ```bash
     git clone https://github.com/<your-github-username>/NASA_Agroforestry_Project.git
     cd NASA_Agroforestry_Project
     ```

2. **Create the Conda Environment**:
   - In the terminal, run:
     ```bash
     conda env create -f data/landsat4.yml
     conda activate landsat4
     ```

3. **Launch Jupyter Notebook**:
   - In the terminal, run:
     ```bash
     jupyter notebook
     ```
   - Open the `NASA_SERVIR.ipynb` notebook located in the `notebooks` directory or copy and paste the code below for better understanding/configuartion.

## Repository Structure

- `/data`: Contains the configuration files and boundary data.
- `/notebooks`: Contains the Jupyter notebook for the analysis.
- `/scripts`: Contains the Python scripts for data processing and analysis.
- `README.md`: Project overview and instructions.
- `requirements.txt`: List of required Python packages.


## Methods and Functions

### Importing Libraries
### Packages and their uses:

- **os**: Interacting with the operating system.
- **pyproj**: Handling projection operations.
- **requests**: Making HTTP requests.
- **pandas**: Data manipulation and analysis.
- **geopandas**: Handling geospatial data.
- **folium**: Creating interactive maps.
- **geojson**: Working with GeoJSON data.
- **warnings**: Managing warnings.
- **matplotlib.pyplot**: Plotting data.
- **rasterio**: Reading and writing geospatial raster data.
- **rioxarray**: Working with raster data in an xarray format.
- **shapely.geometry**: Working with geometric objects.
- **hvplot**: High-level plotting.
- **numpy**: Numerical operations.
- **boto3**: AWS SDK for Python.

##### After creating your enviroment with the landsat4.yml set your kernel to your new enviroment and start your script in this order
```python
import os
import pyproj

# Set the PROJ_LIB environment variable to the path where proj data is located
os.environ['PROJ_LIB'] = '/home/ec2-user/anaconda3/envs/JupyterSystemEnv/share/proj'

# Now you can proceed with the rest of your code
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
```

##### Next step reads a GeoJSON file containing the boundary of the area of interest (AOI) using GeoPandas and converts it into a GeoJSON format. If there are troubles with this step try copying the path directly from Sagemaker.
```python
#This code will allow you to open and read the geojson data we uploaded into sagemaker
aoi_gdf =  gpd.read_file(os.path.join('ucayali_boundary.geojson')) # aoi geopandas dataframe
aoi_geojson = aoi_gdf.__geo_interface__
aoi_gdf.geometry[0]
```



### Set Up STAC Functions

##### Function Overview: Sends a POST request to the STAC server to search for satellite imagery data based on the provided query dictionary. It handles errors and returns the features found.
```python
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
```
##### Function overview: Creates a dictionary of parameters for querying the Landsat Collection 2 Level 2 Surface Reflectance feature in the STAC server. It prints the parameters used and calls the fetch_stac_server function to get the query results.
```python
def send_STAC_query(limit=20, collections=None, intersects=None, datetime=None, bbox=None):
    '''
    This function helps to create a simple parameter dcitionary for querying 
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
        
    # print(params) 
    
    return fetch_stac_server(params)
```

### Separate GEOJSON into Windows


```python
def create_windows(aoi_gdf):
    # Define the window size (in degrees)
    window_size = 0.5  # Adjust this value as needed

    # Extract total bounds from the GeoDataFrame
    xmin, ymin, xmax, ymax = aoi_gdf.total_bounds

    # Calculate the number of sub-windows along x and y axes
    num_x_windows = int(np.ceil((xmax - xmin) / window_size))
    num_y_windows = int(np.ceil((ymax - ymin) / window_size))

    # Initialize a list to store the bounds of each sub-window
    sub_window_bounds = []

    # Iterate over each sub-window
    for i in range(num_x_windows):
        for j in range(num_y_windows):
            # Calculate the bounds of the current sub-window
            sub_xmin = xmin + i * window_size
            sub_ymin = ymin + j * window_size
            sub_xmax = min(sub_xmin + window_size, xmax)
            sub_ymax = min(sub_ymin + window_size, ymax)

            # Append the bounds of the current sub-window to the list
            sub_window_bounds.append((sub_xmin, sub_ymin, sub_xmax, sub_ymax))

    # Print the bounds of each sub-window
    # for idx, bounds in enumerate(sub_window_bounds):
    #     print(f"Sub-window {idx+1}: {bounds}")

    # Define a list to store sub-windows that intersect the AOI
    intersecting_sub_windows = []

    # Iterate over each sub-window bounds
    for bounds in sub_window_bounds:
        sub_xmin, sub_ymin, sub_xmax, sub_ymax = bounds

        # Create a GeoDataFrame for the sub-window bounds
        sub_window_gdf = gpd.GeoDataFrame(geometry=[Polygon([(sub_xmin, sub_ymin), (sub_xmin, sub_ymax), (sub_xmax, sub_ymax), (sub_xmax, sub_ymin)])])

        # Check if the sub-window intersects with the AOI
        if sub_window_gdf.intersects(aoi_gdf.unary_union).any():
            intersecting_sub_windows.append(bounds)

    # Display sub-windows that intersect the AOI
    # print("Sub-windows that intersect the AOI:")
    # for bounds in intersecting_sub_windows:
    #     print(bounds)
    print(f'{len(intersecting_sub_windows)} total windows')
    return intersecting_sub_windows
```
```python
def create_windows(aoi_gdf):
    # Define the window size (in degrees)
    window_size = 0.5  # Adjust this value as needed

    # Extract total bounds from the GeoDataFrame
    xmin, ymin, xmax, ymax = aoi_gdf.total_bounds

    # Calculate the number of sub-windows along x and y axes
    num_x_windows = int(np.ceil((xmax - xmin) / window_size))
    num_y_windows = int(np.ceil((ymax - ymin) / window_size))

    # Initialize a list to store the bounds of each sub-window
    sub_window_bounds = []

    # Iterate over each sub-window
    for i in range(num_x_windows):
        for j in range(num_y_windows):
            # Calculate the bounds of the current sub-window
            sub_xmin = xmin + i * window_size
            sub_ymin = ymin + j * window_size
            sub_xmax = min(sub_xmin + window_size, xmax)
            sub_ymax = min(sub_ymin + window_size, ymax)

            # Append the bounds of the current sub-window to the list
            sub_window_bounds.append((sub_xmin, sub_ymin, sub_xmax, sub_ymax))

    # Print the bounds of each sub-window
    # for idx, bounds in enumerate(sub_window_bounds):
    #     print(f"Sub-window {idx+1}: {bounds}")

    # Define a list to store sub-windows that intersect the AOI
    intersecting_sub_windows = []

    # Iterate over each sub-window bounds
    for bounds in sub_window_bounds:
        sub_xmin, sub_ymin, sub_xmax, sub_ymax = bounds

        # Create a GeoDataFrame for the sub-window bounds
        sub_window_gdf = gpd.GeoDataFrame(geometry=[Polygon([(sub_xmin, sub_ymin), (sub_xmin, sub_ymax), (sub_xmax, sub_ymax), (sub_xmax, sub_ymin)])])

        # Check if the sub-window intersects with the AOI
        if sub_window_gdf.intersects(aoi_gdf.unary_union).any():
            intersecting_sub_windows.append(bounds)

    # Display sub-windows that intersect the AOI
    # print("Sub-windows that intersect the AOI:")
    # for bounds in intersecting_sub_windows:
    #     print(bounds)
    print(f'{len(intersecting_sub_windows)} total windows')
    return intersecting_sub_windows
```

### Get Landsat Scenes

##### Function overview: This function creates a grid of sub-windows within the total bounds of the area of interest (AOI). It calculates the number of sub-windows along the x and y axes based on the specified window size. It then checks which sub-windows intersect with the AOI and returns a list of these intersecting sub-windows.

- **window_size**: Defines the size of each sub-window.
- **xmin, ymin, xmax, ymax**: Extracts the total bounds of the AOI.
- **num_x_windows, num_y_windows**: Calculates the number of sub-windows along the x and y axes.
- **sub_window_bounds**: Stores the bounds of each sub-window.
- **intersecting_sub_windows**: Stores the bounds of sub-windows that intersect with the AOI.

```python
windows = create_windows(aoi_gdf)
test_window = 0
window = windows[test_window]
```
```python
# use the specified datetime format to create string variable of desired time range
year = 2001
date_time = f'{year}-09-01T00:00:00Z/{year+1}-03-15T23:59:59Z'
date_time
```
```python
tuple(window)
```
```python
query_returns = send_STAC_query(limit=1000, collections='landsat-c2l2-sr', bbox=tuple(window), datetime=date_time)
```
```python
# [print(query_item['properties']['proj:epsg']) for query_item in query_returns]
```
```python
import os
import boto3
import pandas as pd
import geopandas as gpd
import hvplot.pandas
import pyproj
import folium
from shapely.geometry import Polygon
import numpy as np

# Set the PROJ_LIB environment variable to the path where proj data is located
os.environ['PROJ_LIB'] = '/home/ec2-user/anaconda3/envs/JupyterSystemEnv/share/proj'

# Function to create windows
def create_windows(aoi_gdf):
    window_size = 0.5  # Adjust this value as needed

    xmin, ymin, xmax, ymax = aoi_gdf.total_bounds

    num_x_windows = int(np.ceil((xmax - xmin) / window_size))
    num_y_windows = int(np.ceil((ymax - ymin) / window_size))

    sub_window_bounds = []

    for i in range(num_x_windows):
        for j in range(num_y_windows):
            sub_xmin = xmin + i * window_size
            sub_ymin = ymin + j * window_size
            sub_xmax = min(sub_xmin + window_size, xmax)
            sub_ymax = min(sub_ymin + window_size, ymax)

            sub_window_bounds.append((sub_xmin, sub_ymin, sub_xmax, sub_ymax))

    intersecting_sub_windows = []

    for bounds in sub_window_bounds:
        sub_xmin, sub_ymin, sub_xmax, sub_ymax = bounds

        sub_window_gdf = gpd.GeoDataFrame(geometry=[Polygon([(sub_xmin, sub_ymin), (sub_xmin, sub_ymax), (sub_xmax, sub_ymax), (sub_xmax, sub_ymin)])])

        if sub_window_gdf.intersects(aoi_gdf.unary_union).any():
            intersecting_sub_windows.append(bounds)

    print(f'{len(intersecting_sub_windows)} total windows')
    return intersecting_sub_windows
```
##### Function overview: This code block handles loading the AOI GeoDataFrame and ensuring it has a Coordinate Reference System (CRS) set. It includes debugging steps to print the CRS information before and after setting it to EPSG:4326 if it's not already set.

- **aoi_gdf**: The GeoDataFrame representing the area of interest.
- **print(f'Initial CRS of AOI: {aoi_gdf.crs}')**: Prints the initial CRS of the AOI.
- **aoi_gdf.set_crs('EPSG:4326', inplace=True)**: Sets the CRS to EPSG:4326 if it's not already set.
- **print(f'CRS of AOI after setting: {aoi_gdf.crs}')**: Prints the CRS of the AOI after setting it.

```
# Load your AOI GeoDataFrame here
# Example: aoi_gdf = gpd.read_file('path/to/your/aoi.geojson')

# Debugging step: Print CRS information
print(f'Initial CRS of AOI: {aoi_gdf.crs}')

# Ensure the AOI GeoDataFrame has a CRS set
if aoi_gdf.crs is None:
    aoi_gdf.set_crs('EPSG:4326', inplace=True)

# Debugging step: Print CRS information after sett
```

##### Function overview: This code first creates sub-windows from the AOI and selects a specific window for further analysis. It constructs a datetime string for the desired time range and sends a STAC query to get Landsat scenes within the specified window and time range. The results are stored in `query_returns`.

- **windows**: List of sub-windows intersecting the AOI.
- **test_window**: Index of the test window to use.
- **window**: Selected test window.
- **year**: Year to specify the time range for the Landsat data query.
- **date_time**: Datetime string defining the time range for the query.
- **tuple(window)**: Converts the window bounds to a tuple format.
- **query_returns**: Stores the results of the STAC query.

```python
windows = create_windows(aoi_gdf)
test_window = 0
window = windows[test_window]

# Use the specified datetime format to create string variable of desired time range
year = 2001
date_time = f'{year}-09-01T00:00:00Z/{year+1}-03-15T23:59:59Z'
date_time

tuple(window)
(-75.94788601799996, -9.448341979999952, -75.44788601799996, -8.948341979999952)

query_returns = send_STAC_query(limit=1000, collections='landsat-c2l2-sr', bbox=tuple(window), datetime=date_time)
```
### Setup Processing Functions

##### Function overview: To extract and return a list of URLs (links) for specific spectral bands from a provided query item.

```python
def get_SR_band_links(query_item):
    band_names = ['blue', 'green','red', 'nir08', 'swir16', 'swir22']

    band_links = []
    
    for b in band_names: #sort through a string to find relevant band names
            band_links.append(query_item['assets'][b]['alternate']['s3']['href'])

    return band_links
```
```python
def get_QA_band_link(query_item):
    return query_item['assets']['qa_pixel']['alternate']['s3']['href']
```
##### Explanation: This function reprojects the bounding coordinates of a sub-window from the source CRS (Coordinate Reference System) of the AOI (area of interest) to the destination CRS.

```python
def reproject_window(window, aoi_gdf, dst_crs):
    # dst_crs = 'EPSG:32619'  # UTM Zone 19N
    # Reproject the first intersecting sub-window to the CRS of the TIFF file
    return  rio.warp.transform_bounds(
        src_crs=aoi_gdf.crs,
        dst_crs=dst_crs,
        left=window[0],
        bottom=window[1],
        right=window[2],
        top=window[3]
    )


def retrieve_cog(geotiff_path, window, aoi_gdf):
    """
    Retrieve a Cloud-Optimized GeoTIFF (COG) within a specified bounding box.

    Parameters:
        - geotiff_path (str): Path to the COG GeoTIFF file.
        - aoi_gdf (geopandas.GeoDataFrame): GeoDataFrame representing the area of interest.

    Returns:
        - cog (numpy.ndarray): Numpy array containing the data within the specified bounding box.
    """
    aws_session = AWSSession(boto3.Session(), requester_pays=True)
    with rio.Env(aws_session):
        with rio.open(geotiff_path) as tif:
            # Reproject the GeoTIFF to EPSG:32619
            # dst_crs = 'EPSG:32619'  # UTM Zone 19N
            dst_crs = tif.crs
            window = reproject_window(window, aoi_gdf, dst_crs)
            
            if tif.crs == dst_crs:
                # print(window)
                cog = tif.read(1, window=from_bounds(
                    window[0],
                    window[1],
                    window[2],
                    window[3],
                    tif.transform), boundless=True, fill_value=0)
                cog = np.array(cog)
            else:  
                cog = reproject_and_clip_tif(geotiff_path, window, dst_crs='EPSG:32619')

    return cog


def retrieve_spectral_data(query_returns, window, aoi_gdf):
    cog = (retrieve_cog(get_SR_band_links(query_returns[0])[0], window, aoi_gdf))
    row, col = cog.shape
    spectral_arr = np.zeros(shape=(6, len(query_returns), row, col), dtype=int)
    qa_arr = np.zeros(shape=(len(query_returns), row, col), dtype=int)

    for idx1, query_item in enumerate(query_returns):
        print(f" Pulling: {idx1} {query_item['id']}")
        band_links = get_SR_band_links(query_item)

        qa_link = get_QA_band_link(query_item)

        for idx2, b in enumerate(band_links):
            spectral_arr[idx2, idx1] = retrieve_cog(b,  window, aoi_gdf)

        qa_arr[idx1] = retrieve_cog(qa_link,  window, aoi_gdf)
        
    return (row, col), spectral_arr, qa_arr
```


```python
def filter_returns(query_returns):
    
    print('Filtering Query Returns')
    from collections import Counter

    # Extract EPSG values from query items
    epsg_values = [query_item['properties']['proj:epsg'] for query_item in query_returns]

    # Count occurrences of each EPSG value
    epsg_counts = Counter(epsg_values)

    # Find the most common EPSG value and its count
    most_common_epsg, most_common_count = epsg_counts.most_common(1)[0]

    # Filter query items to keep only those with the most common EPSG value
    filtered_query_returns = [query_item for query_item in query_returns if query_item['properties']['proj:epsg'] == most_common_epsg]

    # Print the most common EPSG value and the filtered query items
    print(" Most common EPSG value:", most_common_epsg)
    if len(filtered_query_returns) < len(query_returns):
        print(f" Removed {np.abs(len(filtered_query_returns)-len(query_returns))} items from list")
    print(f" Using - {len(filtered_query_returns)} - scenes")
    # for query_item in filtered_query_returns:
    #     print(query_item['properties']['proj:epsg'])
    return filtered_query_returns
```
##### Function overview: creates a boolean mask based on the specified quality assessment (QA) type, where `True` indicates the presence of the specified condition.

- **qa_rod**: The quality assessment array.
- **mask_type**: The type of mask to create. Valid options are: "fill", "dilated", "cirrus", "cloud", "shadow", "snow", "clear", and "water".
- **masks**: Dictionary mapping mask types to their corresponding bit positions.
- **bit**: The bit position corresponding to the specified mask type.

```python
def qa_mask(qa_rod, mask_type):
    """
    Creates a boolean mask based on the specified filter type. Where True indicates presence

    Args:
        qa_arr (np.ndarray): The quality assessment array.
        mask_type (str): The type of mask to create. Valid options are:
            "fill", "dilated", "cirrus", "cloud", "shadow", "snow", "clear" and "water"

    Returns:
        np.ndarray: The boolean mask.

    Raises:
        ValueError: If an invalid mask type is provided.
    """

    masks = {
        "fill": 0,
        "dilated": 1,
        "cirrus": 2,
        "cloud": 3,
        "shadow": 4,
        "snow": 5,
        "clear": 6,
        "water": 7
    }
    
    mask_type = mask_type.lower()  # Convert mask type to lowercase
    
    if mask_type not in masks:
        raise ValueError(f"Invalid mask type: {mask_type}")

    bit = masks[mask_type]
    return (qa_rod & 1 << bit) > 0
```
```python
def qa_filter(spectral_rod, qa_rod):
    # index of observations without clouds
    clear_idxs = [idx for idx, val in enumerate(~(qa_mask(qa_rod, 'cloud'))) if val]

    # filling the bands array with zeros ensures that when the mask_idx is null values will still be passed through
    if len(clear_idxs) == 0:
        #make the band array the size of the spectral_rod if the mask array is empty
        filtered_rod = bands = [[0] * spectral_rod.shape[1] for _ in range(spectral_rod.shape[0])]
    else: 
        filtered_rod = spectral_rod[:, clear_idxs]

    # print('bands:', bands)
    return np.array(filtered_rod)
```
```python
def process_rod(spectral_rod, qa_rod):
    
    filtered_rod = qa_filter(spectral_rod, qa_rod)
    obs_idx = np.argmax(calculate_ndvi(filtered_rod))
    
    return filtered_rod[:,obs_idx]
```
```python
def calculate_ndvi(spectral_rod):
    obs_ndvi = np.zeros(spectral_rod.shape[1])
    for i in range(np.shape(spectral_rod)[1]):
        if spectral_rod[:,i][3] != 0 and spectral_rod[:,i][2] !=0:
            obs_ndvi[i] = (spectral_rod[:,i][3] - spectral_rod[:,i][2]) / (spectral_rod[:,i][3] + spectral_rod[:,i][2])
    return obs_ndvi
```
```python
def process_window(spectral_arr, qa_arr):
    num_bands, rows, cols = spectral_arr.shape[0], spectral_arr.shape[2], spectral_arr.shape[3]

    # create empty array of zeros
    out_comp = [[[0] * cols for _ in range(rows)] for _ in range(num_bands)]
    # out_n_clear_obs = np.array([[[0] * cols for _ in range(rows)]])
    
    processed_pixels = 0
    for row in range(spectral_arr.shape[2]):
        for col in range(spectral_arr.shape[3]):
            out_vals = process_rod(spectral_arr[:, :, row, col], qa_arr[:, row, col])
            
            # Increment the count of processed pixels
            processed_pixels += 1
            # Check if 25% of the pixels have been processed
            if processed_pixels % np.ceil((rows * cols)//10) == 0:
                # Calculate the percentage of completion
                print(f"  > {processed_pixels / (rows * cols) * 100:.0f}% of pixels processed...")
            
            #fill out array
            for b_idx, val in enumerate(out_vals):
                out_comp[b_idx][row][col] = val
                
    return np.array(out_comp)
```
### Run Composite Processing


```python
(height, width), spectral_arr, qa_arr  = retrieve_spectral_data(filter_returns(query_returns), window, aoi_gdf)
```
```python
out_arr = process_window(spectral_arr, qa_arr)
```
```python
out_arr.shape
```

### Save Raster

##### Function overview: 

```python
def get_geometry(query_returns, window, aoi_gdf):
    # Path to the GeoTIFF file
    tif_path = get_SR_band_links(filter_returns(query_returns)[0])[0]
    aws_session = AWSSession(boto3.Session(), requester_pays=True)
    with rio.Env(aws_session):
        # Open the GeoTIFF file
        with rio.open(tif_path) as src:
            return src.crs.to_wkt(), src.window_transform(rio.windows.from_bounds(*reproject_window(window, aoi_gdf, src.crs), src.transform))
```
```python
crs, transform = get_geometry(query_returns, window, aoi_gdf)
```
```python
#%%====================== Set out file names ============================
out_comp_tif = f'comp_peru_{test_window:03d}_{year}.tif'
with rio.open(out_comp_tif, 'w', driver='COG', compress='lzw', nodata=None, width=width, height=height,
              count=6, dtype='uint16', crs=crs, transform=transform) as out_comp_dataset:
    out_comp_dataset.write(out_arr)
```
### Plot Results

##### Function overview: The final step. Your composite photos should appear after this 

```python 
def reproject_window(window, aoi_gdf, dst_crs):
    # dst_crs = 'EPSG:32619'  # UTM Zone 19N
    # Reproject the first intersecting sub-window to the CRS of the TIFF file
    return rio.warp.transform_bounds(
        src_crs=aoi_gdf.crs,
        dst_crs=dst_crs,
        left=window[0],
        bottom=window[1],
        right=window[2],
        top=window[3]
    )
```

```python
 for q in query_returns:
     print(q['properties']['proj:epsg'], q['properties']['proj:transform'], q['properties']['proj:shape'])
```


