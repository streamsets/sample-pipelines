![StreamSets Logo](../../images/StreamSets_Full_Color_Transparent.png)

<h1><p align="center">MySQL to BigQuery - CDC</p></h1>

# MySQL to BigQuery - CDC (Updated: 2022.05)

## Built with Data Collector v5.0.0 - please use engine v5+
## PREREQUISITES

* Access to [StreamSets DataOps Platform](https://cloud.login.streamsets.com/) account
  * Setup [Environment](https://docs.streamsets.com/portal/#platform-controlhub/controlhub/UserGuide/Environments/Overview.html#concept_z4x_nw2_v4b)
  * Setup [Deployment](https://docs.streamsets.com/portal/#platform-controlhub/controlhub/UserGuide/Deployments/Overview.html#concept_srv_jgf_v4b) with engine type [Data Collector](https://docs.streamsets.com/portal/#datacollector/latest/help/datacollector/UserGuide/Getting_Started/GettingStarted_Title.html#concept_sjz_rmx_3q)
    * Once a deployment has been successfully activated, the Data Collector engine should be up
and running before you can create pipelines and run jobs.
* Access to [Google BigQuery](https://cloud.google.com/free/) account
* Access to MySQL database
  * Check [versions of MySQL](https://docs.streamsets.com/portal/#datacollector/4.0.x/help/datacollector/UserGuide/Installation/SupportedSystemVersions.html#concept_k4l_5ft_v4b) supported for CDC

Complete [MySQL CDC prerequisites](https://docs.streamsets.com/portal/#datacollector/latest/help/datacollector/UserGuide/Origins/MySQLBinaryLog.html#concept_nwf_f4x_1bb)

Complete [Google BigQuery prerequisites](https://docs.streamsets.com/portal/platform-datacollector/latest/datacollector/UserGuide/Destinations/GBigQuery.html#concept_vsg_fxb_3rb)

## OVERVIEW

In order to implement a Change Data Capture (CDC) process, you will need to perform a BULK load in order to instantiate the data in your destination which will run once and then perform continuous CDC to keep the data in sync.

The first pipeline (**MySQL_GBQ[a]_BULK**) will be a batch pipeline that runs once and reads data from MySQL and loads data to Snowflake.  This is in a separate folder located [here](https://github.com/streamsets/sample-pipelines/tree/master/Data%20Collector/MySQL%20to%20BigQuery%20-%20BULK)

The second pipeline (**MySQL_GBQ[b]_CDC**) will be a streaming pipeline that will replicate changes from your source to the destination performing inserts, updates and deletes.

**Disclaimer:** *These pipelines are meant to serve as a templates.  Some of the parameters, tables and fields may be different for your environment and may need additional customizations.  Please consult the [StreamSets documentation](https://docs.streamsets.com/) for full information on configuration of each stage used below.*

## PIPELINE #1 - BULK load

![Pipeline](images/MySQLtoGBQ_pipeline.png "MySQL CDC to Snowflake")

## DOCUMENTATION

[MySQL Binlog Origin](https://docs.streamsets.com/portal/platform-datacollector/latest/datacollector/UserGuide/Origins/MySQLBinaryLog.html#concept_kqg_1yh_xx)

[StreamSelector](https://docs.streamsets.com/portal/platform-datacollector/latest/datacollector/UserGuide/Processors/StreamSelector.html#concept_tqv_t5r_wq)

[Field Renamer](https://docs.streamsets.com/portal/platform-datacollector/latest/datacollector/UserGuide/Processors/FieldRenamer.html#concept_vyv_zsg_ht)

[Expression Evaluator](https://docs.streamsets.com/portal/platform-datacollector/latest/datacollector/UserGuide/Processors/Expression.html#concept_zm2_pp3_wq)

[Google BigQuery (Enterprise) Destination](https://docs.streamsets.com/portal/platform-datacollector/latest/datacollector/UserGuide/Destinations/GBigQuery.html#concept_nfr_by1_3rb)

## STEP-BY-STEP

### Step 1: Download the pipeline

[Click Here](./MySQL_GBQ[b]_CDC.zip?raw=true) to download the pipeline and save it to your drive.

### Step 2: Import the pipeline

**For more information on Importing Pipelines, click [here](https://docs.streamsets.com/portal/platform-controlhub/controlhub/UserGuide/ExportImport/Importing.html#concept_gsm_tjx_bdb)**

Click the up arrow in the Pipelines list to start the import process.

![Step 2](images/MySQLtoGBQ_step2.png "Import the Pipeline")

Select 'Archive File', enter a Commit Message, then click "Browse File" and locate the pipeline file you just downloaded and select it. Click "Import"

![Step 2a](images/MySQLtoGBQ_step2a.png "Import the Pipeline")

### Step 3: Configure the parameters

Click on the pipeline you just imported to open it and click on the "Parameters" tab and fill in the appropriate information for your environment.

**Important:** *The pipeline template uses the most common default settings for things like the BigQuery region, staging location, etc. All of these are configurable and if you need to change those, you can opt to not use the built-in parameters and choose the appropriate settings yourself. Please refer to the documentation listed in this document for all the available options.*

![Step 3](images/MySQLtoGBQ_step3.png "Configure the parameters")

The following parameters are set up for this pipeline:

| Parameter Name | Description |
| --- | --- |
| MySQL JDBC | Connection string used to connect to the database. Use the connection string format required by the database vendor.<br>For example, use the following formats for these database vendors:<br>MySQL - jdbc:mysql://[host]:[port]/[database_name]|
| MySQL_USER | User name for the JDBC connection.<br>The user account must have the correct permissions or privileges in the database.|
| MySQL_PASSWORD | Password for the JDBC user name.<br>Tip: To secure sensitive information such as user names and passwords, you can use [runtime resources](https://docs.streamsets.com/portal/platform-datacollector/latest/datacollector/UserGuide/Pipeline_Configuration/RuntimeValues.html#concept_bs4_5nm_2s) or [credential stores](https://docs.streamsets.com/portal/platform-datacollector/latest/datacollector/UserGuide/Configuration/CredentialStores.html#concept_bt1_bpj_r1b).
| GBQ_PROJECTID | Project ID to connect to. |
| GBQ_AUTHFILE | Path to the Google Cloud service account credentials file used to connect. The credentials file must be a JSON file. |
| GBQ_STAGING_BUCKET | Location to stage the pipeline data in CSV files.<br>At this time, only Google Cloud Storage is supported as the staging location. |
| GBQ_DATASET | BigQuery datasets to load data into. To load data into a single dataset, enter the dataset name.<br>To load data into multiple datasets, enter an expression that evaluates to the field in the record that contains the dataset name. For example: ${record:value('/dataset')} |

### Other settings

You may want to check the following settings:
| Location | Setting |
| --- | --- |
| BigQuery Destination | Verify the correct entries for connecting to BigQuery |
| BigQuery Destination --> Data | You **MUST** enter in the table and Key Columns in order to perform updates and deletes. |
| | |
![Step 2b](images/MySQLtoGBQ_step2b.png "Keys")
### Step 3a: Validate the pipeline

Once the parameters have been entered, you will want to validate the pipeline and address any issues.

![Step 3a](images/MySQLtoGBQ_step3a.png "Validate the pipeline")

![Step 3a](images/MySQLtoGBQ_step3a2.png "Validate the pipeline")

### Step 4: Run the pipeline

Click the "Draft Run" button and select Start Pipeline option to run the pipeline.

![Step 4](images/MySQLtoGBQ_step4.png "Run the pipeline")

![Step 4a](images/MySQLtoGBQ_step4a.png "Run the pipeline")

### Step 5: Make changes to the MySQL source table and see the pipeline process them

![Step 5](images/MySQLtoGBQ_step5.png "View the results")
