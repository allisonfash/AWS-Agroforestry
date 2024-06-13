
### Setting Up the Environment

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
   - Open the `NASA_SERVIR.ipynb` notebook located in the `notebooks` directory.

## Repository Structure

- `/data`: Contains the configuration files and boundary data.
- `/notebooks`: Contains the Jupyter notebook for the analysis.
- `/scripts`: Contains the Python scripts for data processing and analysis.
- `README.md`: Project overview and instructions.
- `requirements.txt`: List of required Python packages.

## Methods and Functions

### Importing Libraries

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
