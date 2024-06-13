## NASA Agroforestry Project

The NASA Agroforestry project takes place in various parts of the Brazilian and Peruvian forest. Recently, the project has aimed to focus mainly on the Peruvian side of the Amazon. The goal is to analyze deforestation rates over the years due to intensive agricultural practices. It has been observed on a continental level, and the data analysis process is being replicated onto Amazon Web Services.

The landsatTS package in R is a key aspect for facilitating time series as it measures surface reflection and “spectral indices” originally derived from the Landsat satellite.(Berner et al., 2023). This type of remote sensing analysis has been very important for determining how certain parts of the amazon forest have become developed or restored in the last few years. 

 Currently, we want to start integrating Landsat more heavily into AWS as it has guaranteed open source levels and has better capabilities for importing and exporting data. After importing the data into AWS, we can make a sagemaker notebook in the US-West2 region. With that notebook, it wil produce some data visualizations or maps that can be easily interpreted. After creating a new environment using the notebook code that shows all the mechanics for reading landsat from AWS data lake. In a AWS data lake you are able to store structured and unstructured data, and as of 2021 AWS now has the capabilities of not only holding raster data points but can now hold vector tiles as well. (DeMuth et al., 2022). 


## Overview 
This notebook demonstrates the mechanics of reading Landsat data from the AWS data lake. The primary goal is to create a composite image and generate NDVI for the greenest pixel, aiming for an annual average. Through this project we learn to import Landsat data into AWS using Sagemaker notebook. An Amazon Web services account is necessary to create your instance that supports the Jupyter notebook required for this code. 

To exercise this process, follow these steps:

1.Read the tutorials under the "Notebooks" file path
2. Start a SageMaker notebook in the US-West2 region.
3. Create the environment using the provided YML file.
4. Run the notebook.

Results (points with greenest pixels): 

<img width="517" alt="image" src="https://github.com/allisonfash/AWS-Agroforestry/assets/156024029/a090255d-ce57-4ef7-8856-f186d12fb4ce">

<img width="460" alt="image" src="https://github.com/allisonfash/AWS-Agroforestry/assets/156024029/3a060dbd-5184-4f70-a900-46a328210823">

<img width="481" alt="image" src="https://github.com/allisonfash/AWS-Agroforestry/assets/156024029/507df4eb-5f6e-4892-af41-0fe4672bfea9">
<img width="456" alt="image" src="https://github.com/allisonfash/AWS-Agroforestry/assets/156024029/7d201055-a2b6-404a-a0d2-55b1624cc3c1">
<img width="401" alt="image" src="https://github.com/allisonfash/AWS-Agroforestry/assets/156024029/cc9179f1-f740-4443-9ddf-bdc48f0d9b39">
<img width="455" alt="image" src="https://github.com/allisonfash/AWS-Agroforestry/assets/156024029/640b9c14-bd99-4d84-a56e-52a9b159b123">
<img width="454" alt="image" src="https://github.com/allisonfash/AWS-Agroforestry/assets/156024029/f291b3c8-e4e4-4033-834d-88525dd48c6e">
<img width="423" alt="image" src="https://github.com/allisonfash/AWS-Agroforestry/assets/156024029/13bb4598-06d9-4f53-a4c4-cd209ef569d5">


Sources: 

Berner, L. T., Assmann, J. J., Normand, S., & Goetz, S. J. (2023). 'LandsatTS': An R package to facilitate retrieval, cleaning, cross-calibration, and phenological modeling of Landsat time series data. Ecography. https://doi.org/10.1111/ecog.06768


DeMuth, J., Wells, L., & Boric, N. (2022, July 22). How to partition your geospatial data lake for analysis with Amazon Redshift. AWS Blog. https://aws.amazon.com/blogs/publicsector/how-to-partition-your-geospatial-data-lake-for-analysis-with-amazon-redshift/








