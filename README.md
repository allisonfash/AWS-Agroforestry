## NASA Agroforestry Project

The NASA Agroforestry project takes place in various parts of the Brazilian and Peruvian forest. Recently, the project has aimed to focus mainly on the Peruvian side of the Amazon. The goal is to analyze deforestation rates over the years due to intensive agricultural practices. It has been observed on a continental level, and the data analysis process is being replicated onto Amazon Web Services. For the scope of this individual project I am specifically focusing on the Ucayali region of Peru which has unfortunately experienced these high rates of deforestation.  

### Methods

The methods for the NASA SERVIR Agroforestry, Ucayali project involved several key steps, utilizing various computational resources and geospatial analysis tools to achieve the project’s objectives. 

#### Computational Resources
To handle the extensive data processing required for this project, I utilized Amazon Web Services (AWS). Specifically, I experimented with three different AWS instances to determine the most efficient setup:
- **Medium Instance:** Initial tests were conducted using a medium instance. While it was sufficient for basic tasks, it lacked the necessary power for large-scale data processing.
- **Large Instance:** The large instance provided more computational power, but it still struggled with the intensive processing tasks required for generating composite images.
- **Extra Large Instance:** This instance offered the best performance, allowing for efficient processing of the large datasets involved in the project.

#### Sagemaker Notebooks vs. Sagemaker Labs
Through my experimentation, I discovered that **Sagemaker Notebooks** was a superior option compared to Sagemaker Labs for this project. Sagemaker Notebooks provided the necessary computational capabilities and flexibility to process the composite images effectively, which was a critical requirement for the project.

#### Code Development and Environment Configuration
- **Code Skeleton:** I started with a code skeleton provided for the project. This initial code served as a foundation, but required significant modifications to meet the specific needs of the NASA SERVIR Ucayali project.
- **Environment Configuration:** Using a `.yml` file, I configured the environment to ensure all necessary libraries and dependencies were correctly installed. This configuration was difficult but crucial for the successful execution of the code and the production of the required images.

#### Data Processing and Image Generation
Once the environment was correctly configured, I executed the code to process the satellite imagery and generate composite images that broadcasted the greenest NDVI (Normalized Difference Vegetation Index) pixel. These images are integral to the project’s objectives, providing visual representations of environmental changes and helping to inform resource management and disaster preparedness strategies. 

In summary, the methodology for this project involved a series of computational experiments to determine the optimal AWS instance, the utilization of Sagemaker Notebooks for effective data processing, and extensive code modifications and environment configurations to achieve the desired outcomes.


#### Other Resources
The landsat package is another key aspect for facilitating time series as it measures surface reflection and “spectral indices” originally derived from the Landsat satellite.(Berner et al., 2023). This type of remote sensing analysis has been very important for determining how certain parts of the amazon forest have become developed or restored in the last few years. 

 Currently, we want to start integrating Landsat more heavily into AWS as it has guaranteed open source levels and has better capabilities for importing and exporting data. After importing the data into AWS, we can make a sagemaker notebook in the US-West2 region. With that notebook, it wil produce some data visualizations or maps that can be easily interpreted. After creating a new environment using the notebook code that shows all the mechanics for reading landsat from AWS data lake. In a AWS data lake you are able to store structured and unstructured data, and as of 2021 AWS now has the capabilities of not only holding raster data points but can now hold vector tiles as well. (DeMuth et al., 2022). 


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








