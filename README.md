<h2>Introduction</h2>

I am currently learning Microsoft Azure and I would like to document what I learned on a macro-level! 

To simulate the analysis that I created for my university's student home rental business (which is an academic project), I manually created two small datasets to capture the general trends observed. 

I aim to use my datasets to demonstrate my interest and experience in managing a complete ETL pipeline. Other key objectives include applying design principles to guide the audience’s focus and using data storytelling techniques to communicate insights clearly. This led me to design two versions of the project using different tools — the Power BI version is more visually oriented, while this version serves as practice working with a cloud platform.

<h4> Version 1. Using Microsoft Azure:</h4>

Azure Data Lake Storage (ADLS) for data ingestion and storage.

Data Factory for data pipeline management.

Databricks for data transformation and loading into folder for transformed data (using PySpark).

Bonus: Synapse Analytics for data analysis (using SQL)

This version is covered in this repository.
<br><br>

<h4>Version 2. Using Excel + Python + Power BI:</h4>

Excel for dataset creation.

Python for data transformation (and some data analysis).

Power BI for loading, modelling, and visualizing the data.

Bonus: Microsoft SQL Server can be used as an alternative environment for data analysis / ad-hoc analysis.

•Immediate operational insights include a capacity tracker, identified the top 10 most expensive room options that costs $1xxx , and a summary of return applicants’ first preferences. Please scroll down to near the end of this README.md for details. Here is the link to the SQL file.

See repository <a href="https://github.com/w7978708wen/Student-Home-Rental-Analysis">here</a>.
<br><br>

<h2>Section 1. Extract data from data source to data storage </h2>

<h3> A. Created a storage account to store data </h3>

-I created 2 data folders: raw-data and transformed-data. I uploaded the datasets that I have already transformed using Python so I can proceed through all the ETL steps in the Microsoft Azure quicker.

<img src="https://github.com/w7978708wen/Microsoft-Azure-and-Databricks/blob/main/Images/Containers.png?raw=true"></img>
<br><br>
<h3> B. Created a Data Factory </h3>

-My data factory is called "home-rental-df".

-I used the Data Factory to create the data pipeline. It is good practice to rename data pipeline, in case you would have multiple later.

-Then, I linked the tables. The arrows between them indicate that the tables are executed sequentially — each one runs after the previous one finishes.
I validated, debugged, and published the pipeline. After some troubleshooting, I am happy to obtain the expected result. :)

<img src="https://github.com/w7978708wen/Microsoft-Azure-and-Databricks/blob/main/Images/Data%20Factory.jpg?raw=true"></img>
<br><br>

<h2>Section 2. Azure Databricks for data transformation and data analysis (using PySpark) </h2>

<h3>Create a workspace</h3>

<h3>Create a cluster</h3>
When you create a new workspace, you would need to create a new cluster. 

The cluster will attach to notebook, and you would need to start the cluster before running the code. 

Since my dataset is not large, I chose the single-node type.

<br><br>

<h3>App Registration and Client Secret</h3>

-I created my app called "app02" (I initially created app01, but long-story short I had a hard time debugging so I switched from my default directory to a newly-made tenant directory. Hence, I had to create another app in this directory)

-Also obtained the client id, tenant id, new client secret, the generated secret key value for authentication set-up. 
<br><br>

<h3>Establish connection between Data Factory and Azure Data Lake Storage</h3>

-In Azure Databricks, create a Notebook. Start running the Cluster created and use the configuration template below (I used [] to indicate where you would replace with your own credentials). 

-I retrieved the storage account name and container name.

<img src="https://github.com/w7978708wen/Microsoft-Azure-and-Databricks/blob/main/Images/Add%20role%20assignment.png?raw=true"><img>

-If the authentication process is successful, the code's output would be "True". 


Configuration Template:
```python
configs = {"fs.azure.account.auth.type": "OAuth",
"fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
"fs.azure.account.oauth2.client.id": "[client_id]",
"fs.azure.account.oauth2.client.secret": "[secret_id]",
"fs.azure.account.oauth2.client.endpoint": "https://login.microsoftonline.com/[tenant_id]/oauth2/token"}

dbutils.fs.mount(
source = "abfss://[container-name]@[storage account name].dfs.core.windows.net", 
mount_point = "/mnt/[some name you want to mount it as]",
extra_configs = configs)
```

Note: In real-world applications, the credentials are stored in the "Key Vault". In the tutorial I followed, the credentials are exposed in the code, which I am aware is not good practice. 

<br><br>
<h3>Data transformation and data analysis using PySpark</h3>

Follow along by clicking on my Juypter notebook <a href="https://github.com/w7978708wen/Microsoft-Azure-and-Databricks/blob/main/Databricks%20Spark%20Data%20Transformation.ipynb"> here </a> .
<br><br>

<h4>Data transformation </h4>

The data transformation is done, so that good data could be passed forward onto the data analysis step. 

I used 2 methods: 

Firstly, I maually assessed each column's data type after the .csv file was read, and used the Cast method to change each wrong data type. 

Secondly, I let Spark infer what the data type should be, which correctly identified most date and boolean column types. There was 1 undetected data type, which I manually switched data type. 

One of the key things I learned is to use (\) to convert 2+ columns, given that they belong to the same data frame and are converted to the same data type.
<br><br>

<h4>Data analysis</h4>
I used the clean data to do data analysis.

Sample question: Find the housing option with the highest monthly utilities
  
```python
highest_utilities_housing = pricing.orderBy("Monthly Utilities", ascending=False).show()
```

Here is a snippet of the output (full details available in the Juypter notebook): 
<img src="https://github.com/w7978708wen/Microsoft-Azure-and-Databricks/blob/main/Images/Databricks1.png?raw=true"></img>
<br><br>

<h4>Output transformed dataframes into CSV files</h4>

Next, I output each transformed dataframe into CSV format. I can specify where to store this CSV file, which will be in the transformed-data folder (inside my storage account’s container). During the configuration step, I attached this folder to my mounting point. 


I have three coding templates, depending on whether it is the first time writing on the output csv file, and whether you would like to partition the CSV file into multiple files (because CSV file is too large to be in one file).
<br><br>

Template 1: Output the transformed dataframe to CSV (first-time write)

-When you write the code using Apache Spark, it will create the folder and then write the files with the metadata

```python
[dataframe_name].write.option("header","true").csv("/mnt/homerental/transformed-data/dataframe_name")
```
<br><br>

Template 2: Overwrite an existing file
If you run the code again after the folder has already been created, you’ll get an error. To avoid this, add <code>.mode("overwrite")</code>, which replaces the existing file.

```python
[dataframe_name].write.mode(“overwrite”).option("header","true").csv("/mnt/[mounting_point_name]/[folder_name]/[dataframe_name]")
```
<br><br>

Template 3: Split the output into multiple files (partitioning)
If your file is large, you can instruct Apache Spark to write the output into n partitions, resulting in multiple CSV files.

```python
[dataframe_name].repartition([n])write.mode(“overwrite”).option("header","true").csv("/mnt/[mounting_point_name]/[folder_name]/[dataframe_name]")
```
<br><br>

For this use case, I am going to export all of my csv files using only template 1 (no partitioning nor overwriting).

```python
applicant.write.option("header","true").csv("/mnt/homerental/transformed-data/applicant")
```

```python
capacity.write.option("header","true").csv("/mnt/homerental/transformed-data/capacity")
```

```python
pricing.write.option("header","true").csv("/mnt/homerental/transformed-data/pricing")
```

<br><br>
Each .csv file has its own folder under my transformed-data folder.

<img src="https://github.com/w7978708wen/Microsoft-Azure-and-Databricks/blob/main/Images/Databricks2.png?raw=true"></img>

<br><br>

The actual CSV file is stored in the file that says “part-0000-tid…” , and it can be downloaded. If you choose to output the CSV file into multiple partitions, there will be several files that look similar, such as "part-0000-tid.." and "part-0001-tid...".
<br><br>

<img src="https://github.com/w7978708wen/Microsoft-Azure-and-Databricks/blob/main/Images/Databricks3.png?raw=true"></img>

<br><br>

<h2>Bonus section: Synapse Analytics (using SQL) </h2>

<h3>Synapse Studio set-up</h3>

I used the transformed data sets from Databricks. 

To create a Synapse workspace, I used the same resource group as before. However, some other things like the storage account are tied to the Databricks workspace, such that they cannot be accessed outside of the Databricks workspace. Therefore, I created new things like a new storage account to dedicate to the Synanse workspace set-up. 

I downloaded the datasets from the transformed-data folder which belongs to Databrick's storage account. Then, I put them in the new storage account's folder, so I can easily access them after opening the Synapse Studio. 

<img src="https://github.com/w7978708wen/Microsoft-Azure-and-Databricks/blob/main/Images/Synapse_data_setup.png?raw=true"></img>

<br></br>

Next, I ran a SQL script on each dataset. 

<h3>Data analysis</h3>

<h4>Sample query: Distribution of applicants by classification (new vs returner) </h4>

```sql
SELECT [Classification Description], COUNT([Classification Description]) AS "Count"
FROM
    OPENROWSET(
        BULK 'https://homerentalsynapse2.dfs.core.windows.net/homerentalfs/transformed_applicant.csv.csv',
        FORMAT = 'CSV',
        PARSER_VERSION = '2.0',
        HEADER_ROW = TRUE
    ) AS [result]
    group by [Classification Description]
    ;
  ```

Output:

<img src="https://github.com/w7978708wen/Microsoft-Azure-and-Databricks/blob/main/Images/Synapse_output1.png?raw=true"></img>

<br></br>
Publish the SQL Script to save any code updates.












  
