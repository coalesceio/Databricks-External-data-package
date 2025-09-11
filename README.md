# External Data Package

The Coalesce Databricks External Data Package includes:

* [CopyInto](#CopyInto)
* [Code](#code)
  
<h2 id="CopyInto"> CopyInto </h2>

### CopyInto Node Configuration

The Copy-Into node type the following configurations available:

* [Node Properties](#copy-into-node-properties)
* [General Options](#copy-into-general-options)
* [Source Data](#copy-into-file-location)
* [File Format OPtions](#copy-into-file-format)
* [Copy Options](#copy-into-copy-options)

<h3 id="copy-into-node-properties">CopyInto - Node Properties</h3>

There are four configs within the **Node Properties** group.

| **Property** | **Description** |
|-------------|-----------------|
| **Storage Location** | Storage Location where the Materialized View will be created |
| **Node Type** | Name of template used to create node objects |
| **Description** | A description of the node's purpose |
| **Deploy Enabled** | If TRUE the node will be deployed / redeployed when changes are detected<br/>If FALSE the node will not be deployed or will be dropped during redeployment |

### Key points to use CopyInto Node

* CopyInto node can be created by just clicking on Create node from browser if we want the data from the file to be loaded into single variant column in target table.
* The data can be reloaded into the table by truncating the data in the table before load using the TruncateBefore option in node config or reload parameter
* The path or subfolder name inside stage where the file is located can be specified using config 'Path or subfolder'.Do not prefix or suffix '/' in path name.Example,one level 'SUBFOLDER',two levels 'SUBFOLDER/INNERFOLDER'.

### Use CopyInto node with InferSchema option
* Set Infer Schema toggle to true
* Hit Create button to Infer Schema
* To choose the file format configs,[refer link](#file-format-config-inferschema)
* Click on Re-Sync Columns button
* If all looks good, set Infer Schema button to false
* Hit Create button to execute create table based on inferred schema
* This is mainly a test to make sure create will work
* Hit Run button to execute DML

![Resync](https://github.com/user-attachments/assets/de3c4ce2-370e-43d0-a010-fc6131cd8669)

If the above works, it should be deployable as is.  Deploy will simply take the columns and execute a create table.

<h3 id="copy-into-general-options">  CopyInto - General Options </h3>

**InferSchema-True**

![CopyInto](https://github.com/user-attachments/assets/a537faac-bb91-4a00-b055-4612934d95b9)

**InferSchema-False**

![Copy-Into](https://github.com/user-attachments/assets/60b01ca8-2b99-46ec-8e5c-1491266f2333)

| **Option** | **Description** |
|------------|----------------|
| **Create As** | Select from the options to create as Table or Transient Table<br/>- **Transient table**<br/>  -**Table** |
| **TruncateBefore(Disabled when Inferschema is true)** | True / False toggle that determines whether or not a table is to be truncated before reloading <br/>- **True**: Table is truncated and Copy-Into statement is executed to reload the data into target table<br/>- **False**: Data is loaded directly into target table and no truncate action takes place. |
| **InferSchema** | True / False toggle that determines whether or not to infer the columns of file before loading <br/>- **True**: The node is created with the inferred columns<br/>- **False**: No infer table step is executed |
      
##### Internal or External Stage

| **Setting** | **Description** |
|---------|-------------|
| **Path (Ex:gs://bucket/base/path )** | A path can be an internal volume,external volume in databricks or an external location pointing to S3 bucket/gcp container .|
| **File Name(Ex:a.csv,b.csv)** | Specifies a files name to be loaded.**Note:** It is advised to add either the filename or file pattern file loads |
| **Path or subfolder** | Not mandatory.Specifies the path or subfolders inside the stage where the file is located.Ensure that '/' is not pre-fixed before or after the subfolder name|
| **File Pattern (Ex:'.*hea.*[.]csv')**| A regular expression pattern string, enclosed in single quotes, specifying the file names or paths to match.**Note:** It is advised to add either the filename or file pattern file loads |

<h3 id="copy-into-file-format"> CopyInto - File Format Options </h3>


<h3 id="copy-into-copy-options"> CopyInto - Copy Options </h3>

| **Setting** | **Description** |
|-------------|-----------------|
|**Force**|- Boolean, default false. If set to true, idempotency is disabled and files are loaded regardless of whether they've been loaded before.|
|**Mergeschema**|-Boolean, default false. If set to true, the schema can be evolved according to the incoming data.|

### CopyInto - System Columns

The set of columns which has source data and file metadata information.

* **LOAD_TIMESTAMP** - Current timestamp when the file gets loaded.
* **FILENAME** - Name of the file the current row belongs to. 
* **FILEPATH** - Full path of the file in storage
* **FILEBLOCKSTART** - Start offset of the file split being read
* **FILEBLOCKEND** - Length of the file split being read
* **FILESIZE** - Size of the file in bytes
* **FILE_LAST_MODIFIED** - Last modified timestamp of the file

### CopyInto Deployment

#### CopyInto Initial Deployment
When deployed for the first time into an environment the Copy-into node of materialization type table will execute the below stage:

| Deployment Behavior | Stages Executed |
|--|--|
| Initial Deployment |Create Table|

### CopyInto Redeployment

#### Altering the CopyInto Tables

There are few column or table changes like Change in table name,Dropping existing column, Alter Column data type,Adding a new column if made in isolation or all-together will result in an ALTER statement to modify the Work Table in the target environment.

The following stages are executed:

* **Rename Table| Alter Column | Delete Column | Add Column | Edit table description**: Alter table statement is executed to perform the alter operation.

### Redeployment with no changes 

If the nodes are redeployed with no changes compared to previous deployment,then no stages are executed

### CopyInto Undeployment

If the CopyInto node is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher-level environment then the target table in the target environment will be dropped.

* **Drop table**: Target table in Snowflake is dropped
