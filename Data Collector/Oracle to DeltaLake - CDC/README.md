![StreamSets Logo](../../images/StreamSets_Full_Color_Transparent.png)

<h1><p align="center">Oracle to DeltaLake - CDC</p></h1>

# Oracle to DeltaLake - CDC (Updated: 2022.05)

## Built with Data Collector v5.0.0 - please use engine v5+
## PREREQUISITES

* Access to [StreamSets DataOps Platform](https://cloud.login.streamsets.com/) account
  * Setup [Environment](https://docs.streamsets.com/portal/#platform-controlhub/controlhub/UserGuide/Environments/Overview.html#concept_z4x_nw2_v4b)
  * Setup [Deployment](https://docs.streamsets.com/portal/#platform-controlhub/controlhub/UserGuide/Deployments/Overview.html#concept_srv_jgf_v4b) with engine type [Data Collector](https://docs.streamsets.com/portal/#datacollector/latest/help/datacollector/UserGuide/Getting_Started/GettingStarted_Title.html#concept_sjz_rmx_3q)
    * Once a deployment has been successfully activated, the Data Collector engine should be up
and running before you can create pipelines and run jobs.
* Access to [DeltaLake](https://databricks.com/try-databricks) account
* Access to Oracle database
  * Check [versions of Oracle](https://docs.streamsets.com/portal/#datacollector/4.0.x/help/datacollector/UserGuide/Installation/SupportedSystemVersions.html#concept_k4l_5ft_v4b) supported for CDC
* Install the Oracle JDBC Driver
  * Download the correct JDBC driver from the web i.e. https://www.oracle.com/database/technologies/appdev/jdbc-downloads.html
  * Install the driver following these instructions **after you have imported the pipeline and from the Origin stage**: [Installing from Stage Properties](https://docs.streamsets.com/datacollector/latest/help/datacollector/UserGuide/Configuration/ExternalLibs.html#concept_skq_dh3_5mb)

Complete [Oracle CDC prerequisites](https://docs.streamsets.com/portal/platform-datacollector/latest/datacollector/UserGuide/Origins/OracleCDC.html#concept_xwg_33w_cx)

Complete [Delta Lake prerequisites](https://docs.streamsets.com/portal/platform-datacollector/latest/datacollector/UserGuide/Destinations/DeltaLake.html#concept_xnp_y5f_dlb)

## OVERVIEW

In order to implement a Change Data Capture (CDC) process, you will need to perform a BULK load in order to instantiate the data in your destination which will run once and then perform continuous CDC to keep the data in sync.

The first pipeline (**Oracle_DeltaLake[a]_BULK**) will be a batch pipeline that runs once and reads data from Oracle and loads data to DeltaLake.  This is in a separate folder located [here](https://github.com/streamsets/sample-pipelines/tree/master/Data%20Collector/Oracle%20to%20DeltaLake%20-%20BULK)

The second pipeline (**Oracle_DeltaLake[b]_CDC**) will be a streaming pipeline that will replicate changes from your source to the destination performing inserts, updates and deletes.

**Disclaimer:** *These pipelines are meant to serve as a templates.  Some of the parameters, tables and fields may be different for your environment and may need additional customizations.  Please consult the [StreamSets documentation](https://docs.streamsets.com/) for full information on configuration of each stage used below.*

## PIPELINE - CDC load

![Pipeline](images/OracletoDeltaLake_pipeline.png "Oracle CDC to DeltaLake")

## DOCUMENTATION

[Oracle CDC Client Origin](https://docs.streamsets.com/portal/platform-datacollector/latest/datacollector/UserGuide/Origins/OracleCDC.html#concept_rs5_hjj_tw)

[Expression Evaluator](https://docs.streamsets.com/portal/platform-datacollector/latest/datacollector/UserGuide/Processors/Expression.html#concept_zm2_pp3_wq)

[Field Renamer](https://docs.streamsets.com/portal/platform-datacollector/latest/datacollector/UserGuide/Processors/FieldRenamer.html#concept_vyv_zsg_ht)

[DeltaLake Destination](https://docs.streamsets.com/portal/platform-datacollector/latest/datacollector/UserGuide/Destinations/DeltaLake.html#concept_ddy_cdz_clb)

## STEP-BY-STEP

### Step 1: Download the pipeline

[Click Here](./Oracle_DeltaLake[b]_CDC.zip?raw=true) to download the pipeline and save it to your drive.

### Step 2: Import the pipeline

**For more information on Importing Pipelines, click [here](https://docs.streamsets.com/portal/platform-controlhub/controlhub/UserGuide/ExportImport/Importing.html#concept_gsm_tjx_bdb)**

Click the up arrow in the Pipelines list to start the import process.

![Step 2](images/OracletoDeltaLake_step2.png "Import the Pipeline")

Select 'Archive File', enter a Commit Message, then click "Browse File" and locate the pipeline file you just downloaded and select it. Click "Import"

![Step 2a](images/OracletoDeltaLake_step2a.png "Import the Pipeline")

### Step 3: Configure the parameters

Click on the pipeline you just imported to open it and click on the "Parameters" tab and fill in the appropriate information for your environment.

**Important:** *The pipeline template uses the most common default settings for things like the DeltaLake region, staging location, etc. All of these are configurable and if you need to change those, you can opt to not use the built-in parameters and choose the appropriate settings yourself. Please refer to the documentation listed in this document for all the available options.*

![Step 3](images/OracletoDeltaLake_step3.png "Configure the parameters")

The following parameters are set up for this pipeline:

| Parameter Name | Description |
| --- | --- |
| DeltaLake_JDBC_URL | JDBC URL used to connect to the Databricks cluster.<br>For example: jdbc:spark://[server_hostname]:443/default;transportMode=http :ssl=1;httpPath=sql/protocolv1/o/0/xxxx-xxxxxx-xxxxxxxx;AuthMech=3;<br>**Tip:** In Databricks, you can locate the JDBC URL for your cluster on the JDBC/ODBC tab in the cluster configuration details. As a best practice, remove the PWD parameter from the URL, and then enter the personal access token value in the Token property below. |
| DeltaLake_Token | Personal access token used to connect to the Databricks cluster.<br>**Tip:** To secure sensitive information such as access key pairs, you can use runtime resources or credential stores. |
| DeltaLake_Database | Name of the DeltaLake database |
| DeltaLake_Dir_Location | Directory for the Delta table location, specified as a path on Databricks File System (DBFS).<br>The destination adds the specified Table Name value as a subdirectory to create the final table location. For example, if you enter /mnt/deltalake as the directory for the table location and you enter sales.accounts as the table name, the final table location is /mnt/deltalake/sales.accounts.<br>When you specify a location, the destination creates an unmanaged Delta table. When you do not specify a location, the destination creates a managed Delta table. For more information, see the Delta Lake documentation.<br>Available when data drift and automatic table creation are enabled. |
| DeltaLake_S3_bucket | Bucket name or path to the existing Amazon S3 location to write the staged files.<br>Enter the bucket name or enter the full bucket path in the following format:<br>[bucket]/[prefix]<br>Available when using the Amazon S3 staging location. |
| Oracle_JDBC | Connection string used to connect to the database. Use the connection string format required by the database vendor.<br>For example, use the following formats for these database vendors:<br>Oracle - jdbc:oracle:[driver_type]:@[host]:[port]:[service_name]|
| Oracle_Username | User name for the JDBC connection.<br>The user account must have the correct permissions or privileges in the database.|
| Oracle_Password | Password for the JDBC user name.<br>Tip: To secure sensitive information such as user names and passwords, you can use [runtime resources](https://docs.streamsets.com/portal/platform-datacollector/latest/datacollector/UserGuide/Pipeline_Configuration/RuntimeValues.html#concept_bs4_5nm_2s) or [credential stores](https://docs.streamsets.com/portal/platform-datacollector/latest/datacollector/UserGuide/Configuration/CredentialStores.html#concept_bt1_bpj_r1b).

#### Other settings

You may want to check the following settings:
| Location | Setting |
| --- | --- |
| DeltaLake Destination --> Staging | Verify Staging location and access method |
| DeltaLake Destination --> Data | You **MUST** enter in the table and Key Columns in order to perform updates and deletes. |
| | |
![Step 2b](images/OracletoDeltaLake_step2b.png "Keys")
### Step 3a: Validate the pipeline

Once the parameters have been entered, you will want to validate the pipeline and address any issues.

![Step 3a](images/OracletoDeltaLake_step3a.png "Validate the pipeline")

![Step 3a](images/OracletoDeltaLake_step3a2.png "Validate the pipeline")

### Step 4: Run the pipeline

Click the "Draft Run" button and select Reset Origin & Start option to run the pipeline.

![Step 4](images/OracletoDeltaLake_step4.png "Run the pipeline")

![Step 4a](images/OracletoDeltaLake_step4a.png "Run the pipeline")

### Step 5: Make changes to the Oracle source table and see the pipeline process them

![Step 5](images/OracletoDeltaLake_step5.png "View the results")
