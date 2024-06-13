# Landsat Greenest Pixel Composite

This repository contains the code to process Landsat data and generate a greenest pixel composite. The goal is to retrieve Landsat scenes, filter out cloudy observations, and compute the greenest pixel based on NDVI for a given area of interest.

## Repository Structure

- `environment.yml`: Conda environment file to set up dependencies.
- `data`: Contains the GeoJSON file defining the area of interest.
- `src`: Source code for various processing steps.
- `notebooks`: Jupyter notebook for interactive processing.

## Setup

1. Clone the repository:
    ```bash
    git clone https://github.com/yourusername/landsat-greenest-pixel.git
    cd landsat-greenest-pixel
    ```

2. Create the conda environment:
    ```bash
    conda env create -f environment.yml
    conda activate landsat4
    ```

3. Run the main script:
    ```bash
    python src/main.py
    ```

## Files and Functions

- `fetch_stac.py`: Functions to query the STAC server for Landsat data.
- `create_windows.py`: Function to create sub-windows from the area of interest.
- `retrieve_data.py`: Functions to retrieve spectral and QA data from Landsat scenes.
- `process_data.py`: Functions to process the data and compute NDVI.
- `plot_results.py`: Functions to plot the results.
- `main.py`: Main script to run the entire process.

## Jupyter Notebook

For interactive exploration, use the notebook in the `notebooks` directory:
```bash
jupyter notebook notebooks/process_landsat.ipynb

