# Microsoft-Azure-and-Databricks

I am currently learning Microsoft Azure and I would like to document what I learned on a macro-level! 

<h2>Step 1. Extract data from data source to data storage </h2>

<h3> A. Created a storage account to store data </h3>
-Inside Azure's Resource page, I went to the Data Storage section to create a new container inside. 

-Inside the new container, I created 2 data folders: raw-data and transformed-data. I uploaded the datasets that I have already transformed using Python so I can proceed through all the ETL steps in the Microsoft Azure quicker.

<img src="https://github.com/w7978708wen/Microsoft-Azure-and-Databricks/blob/main/Images/Containers.png?raw=true"></img>

<h3> B. Created a Data Factory </h3>

-My data factory is called 'home-rental-df'.

-I used the Data Factory to create the data pipeline. 

-I extracted the data files from my GitHub repository, and loaded them to my data storage's container's raw-data folder. 

-Each data file needs to be extracted individiually. 

-In each data extraction, there are 3 sections to fill out: General, Source, Sink:


<h4>Source</h4>

-In the Source section, I set the data source as HTTP because this data is accessed through our HTTP server. 

-I collected the base url for my data file by clicking on my file's raw page. 

-Tip: preview the data to see if the data has been imported correctly. 



<h4>Sink</h4>
-This is the section to convert data format, if you want. I sticked with .csv delimiter text format for simplicity. 

-The data is extracted from Data Factory, and loaded to Data Lake Gen 2. 

-The linked service section establishes a link between Data Factory and our Data Lake Gen 2 storage account. 

-If you are using a free account, you would need to disable the Blob-only features for your storage account, so the file path connection can be established.



-After filling out all 3 sections (General, Source, Sink), vaildate the data (to see if there are any error messages), then press 'debug'. This is whe the activity status will go from 'Queued' to 'Succeeded'. The waiting time can take a while depending on your file size.

-Then, link the tables. The arrows between them indicate that the tables are executed sequentially — each one runs after the previous one finishes.

-After some troubleshooting, I am happy to obtain the expected result :)

<img src="https://github.com/w7978708wen/Microsoft-Azure-and-Databricks/blob/main/Images/Data%20Factory.jpg?raw=true"></img>



  
