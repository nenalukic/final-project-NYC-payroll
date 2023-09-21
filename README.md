# Data Integration Pipelines for NYC Payroll Data Analytics

## Project Introduction

The City of New York would like to develop a Data Analytics platform on Azure Synapse Analytics to accomplish two primary objectives:

1. Analyze how the City's financial resources are allocated and how much of the City's budget is being devoted to overtime.
2. Make the data available to the interested public to show how the City’s budget is being spent on salary and overtime pay for all municipal employees.

As a Data Engineer you have to create high-quality data pipelines that are dynamic, can be automated, and monitored for efficient operation. The project team also includes the city’s quality assurance experts who will test the pipelines to find any errors and improve overall data quality.

The source data resides in Azure Data Lake and needs to be processed in a NYC data warehouse in Azure Synapse Analytics. The source datasets consist of CSV files with Employee master data and monthly payroll data entered by various City agencies.

## Project Instructions

For this project, you'll do your work in the Azure Portal, using several Azure resources including:

- Azure Data Lake Gen2
- Azure SQL DB
- Azure Data Factory
- Azure Synapse Analytics

![Alt text](Screenshots/1.%20Step%20Prepare%20the%20Data%20Infrastructure/All%20Resources.png)

## Step 1: Prepare the Data Infrastructure

1.Create the data lake and upload data

Log into your temporary Azure account (instructions on the previous page) and create the following resources. Please use the provided resource group to create each resource. You will use these resources for the whole project, in all of the steps, so you'll only need to create one of each:

- Create an Azure Data Lake Storage Gen2 (storage account) and associated storage container resource named adlsnycpayroll-yourfirstname-lastintial. Create three directories in this storage container named

- dirpayrollfiles
- dirhistoryfiles
- dirstaging

![Alt text](Screenshots/1.%20Step%20Prepare%20the%20Data%20Infrastructure/Container%20with%20directories.png)

Upload these files from the project data to the dirpayrollfiles folder.

- EmpMaster.csv
- AgencyMaster.csv
- TitleMaster.csv
- nycpayroll_2021.csv

![Alt text](Screenshots/1.%20Step%20Prepare%20the%20Data%20Infrastructure/Uploaded%20csv%20files.png)

Upload this file (historical data) from the project data to the dirhistoryfiles folder

- nycpayroll_2020.csv

![Alt text](Screenshots/1.%20Step%20Prepare%20the%20Data%20Infrastructure/Uploaded%20historical%20file.png)

2. Create an Azure Data Factory Resource

3. Create a SQL Database to store the current year of the payroll data

In the Azure portal, create a SQL Database resource named db_nycpayroll

Add client IP address to the SQL DB firewall

Create a table called NYC_Payroll_Data in db_nycpayroll in the Azure Query Editor

![Alt text](Screenshots/1.%20Step%20Prepare%20the%20Data%20Infrastructure/Database%20with%20table.png)

4. Create a Synapse Analytics workspace, or use one you already have created.

5. Create master data tables and payroll transaction tables in Synapse Analytics workspace.

IMPORTANT: It is better to use your own Azure account and use SQL Dedicated pool. With that you can use CREATE TABLE instead CREATE EXTERNAL TABLE. As part of the recent updates to the Udacity cloud labs, you will not be allowed to create SQL dedicated pool in the Synapse Analytics workspace.

![Alt text](Screenshots/1.%20Step%20Prepare%20the%20Data%20Infrastructure/Master%20data%20tables%20in%20Synapse%20Analytics%20workspace.png)

## Step 2: Create Linked Services

1.Create a Linked Service for Azure Data Lake

- In Azure Data Factory, create a linked service to the data lake that contains the data files

- From the data stores, select Azure Data Lake Gen 2
Test the connection

2.Create a Linked Service to SQL Database that has the current (2021) data

- If you get a connection error, remember to add the IP address to the firewall settings in SQL DB in the Azure Portal
3. Create a Linked Service for Synapse Analytics

- Create the linked service to the SQL pool.

![Alt text](Screenshots/2.%20Step%20Create%20Linked%20Services/Linked%20Services.png)

## Step 3: Create Datasets in Azure Data Factory

1.Create the datasets for the 2021 Payroll file on Azure Data Lake Gen2

- Select DelimitedText
- Set the path to the nycpayroll_2021.csv in the Data Lake
- Preview the data to make sure it is correctly parsed

2. Repeat the same process to create datasets for the rest of the data files in the Data Lake

EmpMaster.csv
TitleMaster.csv
AgencyMaster.csv
Remember to publish all the datasets

3. Create the dataset for transaction data table that should contain current (2021) data in SQL DB

4. Create the datasets for destination (target) tables in Synapse Analytics

- dataset for NYC_Payroll_EMP_MD
- for NYC_Payroll_TITLE_MD
- for NYC_Payroll_AGENCY_MD
- for NYC_Payroll_Data

![Alt text](Screenshots/3.%20Step%20Create%20Datasets%20in%20Azure%20Data%20Factory/Datasets.png)

## Step 4: Create Data Flows

1.In Azure Data Factory, create the data flow to load 2021 Payroll Data to SQL DB transaction table (in the future NYC will load all the transaction data into this table).

- Create a new data flow
- Select the dataset for the 2021 payroll file as the source
- Select the sink dataset as the payroll table on SQL DB
- Make sure to reassign any missing source to target mappings

2.Create Pipeline to load 2021 Payroll data into transaction table in the SQL DB

- Create a new pipeline
- Select the data flow to load the 2021 file into SQLDB
- Trigger the pipeline
- Monitor the pipeline
- Take a screenshot of the Azure Data Factory screen pipeline run after it has finished.
- Make sure the data is successfully loaded into the SQL DB table

3. Create data flows to load the data from the data lake files into the Synapse Analytics data tables

- Create the data flows for loading Employee, Title, and Agency files into corresponding SQL pool tables on Synapse Analytics
- For each Employee, Title, and Agency file data flow, sink the data into each target Synaspe table

4. Create a data flow to load 2021 data from SQL DB to Synapse Analytics

![Alt text](Screenshots/4.%20Step%20Create%20Data%20Flows/Data%20flows.png)

5. Create pipelines for Employee, Title, Agency, and year 2021 Payroll transaction data to Synapse Analytics containing the data flows.

- Select the dirstaging folder in the data lake storage for staging
- Optionally you can also create one master pipeline to invoke all the Data Flows
- Validate and publish the pipelines

6. Trigger and monitor the Pipelines

- Take a screenshot of each pipeline run after it has finished, or one after your master pipeline run has finished.
- In total, you should have 6 pipelines or one master pipeline



![Alt text](Screenshots/4.%20Step%20Create%20Data%20Flows/Trigger%20and%20monitor%20pipelines.png)

## Step 5: Data Aggregation and Parameterization

1.Create a Summary table in Synapse with the following SQL script and create a dataset named table_synapse_nycpayroll_summary 

CREATE TABLE [dbo].[NYC_Payroll_Summary]( [FiscalYear] [int] NULL, [AgencyName] [varchar](50) NULL, [TotalPaid] [float] NULL )

2.Create a new dataset for the Azure Data Lake Gen2 folder that contains the historical files.

- Select dirhistoryfiles in the data lake as the source

3.Create new data flow and name it Dataflow Aggregate Data

- Create a data flow level parameter for Fiscal Year
- Add first Source for table_sqldb_nyc_payroll_data table
- Add second Source for the Azure Data Lake history folder

4.Create a new Union activity in the data flow and Union with history files

5.Add a Filter activity after Union

- In Expression Builder, enter toInteger(FiscalYear) >= $dataflow_param_fiscalyear

6.Derive a new TotalPaid column

- In Expression Builder, enter RegularGrossPaid + TotalOTPaid+TotalOtherPay

7.Add an Aggregate activity to the data flow next to the TotalPaid activity

- Under Group By, Select AgencyName and Fiscal Year

8.Add a Sink activity to the Data Flow

- Select the dataset to target (sink) the data into the Synapse Analytics Payroll Summary table.
- In Settings, select Truncate Table

9.Create a new Pipeline and add the Aggregate data flow

- Create a new Global Parameter (This will be the Parameter at the global pipeline level that will be passed on to the data flow
- In Parameters, select Pipeline Expression
- Choose the parameter created at the Pipeline level

10.Validate, Publish and Trigger the pipeline. Enter the desired value for the parameter.

11.Monitor the Pipeline run and take a screenshot of the finished pipeline run.

![Alt text](Screenshots/5.%20Step%20Data%20Aggregation%20and%20Parameterization/Data%20flow%20Aggregate%20Data.png)

## Step 6: Connect your Project to Github

![Alt text](Screenshots/6.%20Step%20Connect%20your%20Project%20to%20Github/GIt%20configuration-GitHub.png)