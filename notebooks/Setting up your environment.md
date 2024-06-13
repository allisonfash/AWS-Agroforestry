# Landsat Greenest Pixel Composite

This repository contains the code to process Landsat data and generate a greenest pixel composite. The goal is to retrieve Landsat scenes, filter out cloudy observations, and compute the greenest pixel based on NDVI for a given area of interest.

## Repository Structure

- `environment.yml`: Conda environment file to set up dependencies.
- `data`: Contains the GeoJSON file defining the area of interest.
- `src`: Source code for various processing steps.
- `notebooks`: Jupyter notebook for interactive processing.


## Setup Instructions

### Prerequisites

1. **AWS Account**: Ensure you have an active AWS account.
2. **IAM Role**: Create an IAM role with the necessary permissions for SageMaker, S3, and other AWS services you will use.
3. **S3 Bucket**: Create an S3 bucket to store your data and notebook files.

### Steps to Set Up an Amazon SageMaker Notebook Instance

1. **Open the Amazon SageMaker Console**:
   - Go to the [Amazon SageMaker Console](https://console.aws.amazon.com/sagemaker/).

2. **Create a Notebook Instance**:
   - Click on **Notebook instances** in the left sidebar.
   - Click on the **Create notebook instance** button.

3. **Configure the Notebook Instance**:
   - **Notebook instance name**: `NASA-Agroforestry-Instance`
   - **Instance type**: `ml.t3.2xlarge` (or any other extra-large instance type as needed)
   - **IAM role**: Select an existing role or create a new role with the necessary permissions.

4. **Add a Git Repository**:
   - Under the **Git repositories** section, add your GitHub repository for this project.
   - Repository URL: `https://github.com/<your-github-username>/NASA_Agroforestry_Project.git`


##Setup Enviroment

1. Open a terminal from your Sagemaker instance:
2. Use ls to find your directory

3. Create the conda environment:
    ```bash
    conda env create -f environment.yml
    conda activate landsat4
    ```

4. Run the main script:
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

