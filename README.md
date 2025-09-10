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
      
<h3 id="copy-into-file-location">  CopyInto - Source Data </h3>

**InferSchema-true**

![image](https://github.com/user-attachments/assets/2f549fe3-648f-4132-bcdf-2ab92f6554bd)

**InferSchema-false**

![image](https://github.com/user-attachments/assets/c1071dff-da18-45fc-9e43-1dd5cdd03329)

##### Internal or External Stage

| **Setting** | **Description** |
|---------|-------------|
| **Coalesce Storage Location of Stage** | A storage location in Coalesce where the stage is located.|
| **Stage Name (Required)** | Internal or External stage where the files containing data to be loaded are staged|
| **File Name(s)(Ex:a.csv,b.csv)** | Specifies a list of one or more files names (separated by commas) to be loaded |
| **Path or subfolder** | Not mandatory.Specifies the path or subfolders inside the stage where the file is located.Ensure that '/' is not pre-fixed before or after the subfolder name|
| **File Pattern (Ex:'.*hea.*[.]csv')**| A regular expression pattern string, enclosed in single quotes, specifying the file names or paths to match |

<h3 id="copy-into-file-format"> CopyInto - File Format </h3>

**File format definition-File format name**

![Copy-Into format](https://github.com/user-attachments/assets/9e4e8f7e-29be-4209-a08b-cb19fadccf05)

<h3 id="copy-into-copy-options"> CopyInto - Copy Options </h3>

| **Setting** | **Description** |
|-------------|-----------------|
|**Enforce Length**|-Boolean that specifies whether to truncate text strings that exceed the target column length|
|**Truncate Columns**|-Boolean that specifies whether to truncate text strings that exceed the target column length|

### CopyInto - System Columns

The set of columns which has source data and file metadata information.

* **SRC** - The data from the file is loaded into this variant column.
* **LOAD_TIMESTAMP** - Current timestamp when the file gets loaded.
* **FILENAME** - Name of the staged data file the current row belongs to. Includes the full path to the data file.
* **FILE_ROW_NUMBER** - Row number for each record in the staged data file.
* **FILE_LAST_MODIFIED** - Last modified timestamp of the staged data file the current row belongs to
* **SCAN_TIME** - Start timestamp of operation for each record in the staged data file. Returned as TIMESTAMP_LTZ.

### CopyInto Deployment

#### CopyInto Deployment Parameters

The CopyInto node typeincludes an environment parameter that allows you to specify if you want to perform a full load or a reload based on the load type when you are performing a Copy-Into operation.

The parameter name is `loadType` and the default value is ``.

```json
{
    "loadType": ""
}
```

When the parameter value is set to `Reload`, the data is reloaded into the table regardless of whether they’ve been loaded previously and have not changed since they were loaded.As a alternate option we can use TruncateBefore option in node config

#### CopyInto Initial Deployment
When deployed for the first time into an environment the Copy-into node of materialization type table will execute the below stage:

| Deployment Behavior  | Load Type | Stages Executed |
|--|--|--|
| Initial Deployment | ``|Create Table/Transient Table
| Initial Deployment | Reload|Create Table/Transient Table

### CopyInto Redeployment

#### Altering the CopyInto Tables

There are few column or table changes like Change in table name,Dropping existing column, Alter Column data type,Adding a new column if made in isolation or all-together will result in an ALTER statement to modify the Work Table in the target environment.

The following stages are executed:

* **Rename Table| Alter Column | Delete Column | Add Column | Edit table description**: Alter table statement is executed to perform the alter operation.

#### Copy-Into change in materialization type

When the materialization type of Copy-Into node is changed from table to transient table or viceversa,the below stages are executed:

* **Drop table/transient table**
* **Create transient table/table**
* 
### Redeployment with no changes 

If the nodes are redeployed with no changes compared to previous deployment,then no stages are executed

### CopyInto Undeployment

If the CopyInto node is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher-level environment then the target table in the target environment will be dropped.

* **Drop table/transient table**: Target table in Snowflake is dropped
