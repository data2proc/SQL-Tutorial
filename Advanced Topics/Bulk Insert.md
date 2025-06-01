# What is *Bulk Insert* and How Does It Work in SQL Server?

## Introduction
One of the most common operations related to databases is importing and exporting data in a specific format to/from SQL Server. SQL Server provides several tools for this purpose, including:

**SQL Server 2019 Import and Export Data (64-bit)**

This tool helps in the data transfer process. However, as you may know, there are other methods as well. SQL Server has also introduced the `BULK INSERT` command for this purpose, which we will explain below.

**Let's first examine the command itself:**   

```sql  
BULK INSERT  
   { database_name.schema_name.table_or_view_name | schema_name.table_or_view_name | table_or_view_name }  
      FROM 'data_file'  
     [ WITH  
    (  
   [ [ , ] BATCHSIZE = batch_size ]  
   [ [ , ] CHECK_CONSTRAINTS ]  
   [ [ , ] CODEPAGE = { 'ACP' | 'OEM' | 'RAW' | 'code_page' } ]  
   [ [ , ] DATAFILETYPE =  
      { 'char' | 'native' | 'widechar' | 'widenative' } ]  
   [ [ , ] DATA_SOURCE = 'data_source_name' ]  
   [ [ , ] ERRORFILE = 'file_name' ]  
   [ [ , ] ERRORFILE_DATA_SOURCE = 'errorfile_data_source_name' ]  
   [ [ , ] FIRSTROW = first_row ]  
   [ [ , ] FIRE_TRIGGERS ]  
   [ [ , ] FORMATFILE_DATA_SOURCE = 'data_source_name' ]  
   [ [ , ] KEEPIDENTITY ]  
   [ [ , ] KEEPNULLS ]  
   [ [ , ] KILOBYTES_PER_BATCH = kilobytes_per_batch ]
   [ [ , ] LASTROW = last_row ]
   [ [ , ] MAXERRORS = max_errors ]
   [ [ , ] ORDER ( { column [ ASC | DESC ] } [ ,...n ] ) ]
   [ [ , ] ROWS_PER_BATCH = rows_per_batch ]
   [ [ , ] ROWTERMINATOR = 'row_terminator' ]
   [ [ , ] TABLOCK ]

   -- input file format options
   [ [ , ] FORMAT = 'CSV' ]
   [ [ , ] FIELDQUOTE = 'quote_characters']
   [ [ , ] FORMATFILE = 'format_file_path' ]
   [ [ , ] FIELDTERMINATOR = 'field_terminator' ]
   [ [ , ] ROWTERMINATOR = 'row_terminator' ]
    )]

```
# Arguments of Bulk Insert

## Database_name 
The name of the database where the table or view is located. If no name is provided, the current database is assumed.

## Schema_name
The schema name of the table. If the specified schema is the default schema, setting or not setting this value will not affect the execution of the command. However, if the provided schema differs from the existing schema, an error will be raised.

## Table_name
The name of the table where the bulk data should be inserted.

## From ‘Data_file’
The full path of the file to be imported into the table. The file extension must be explicitly mentioned. Example path:  
\\SystemX\DiskZ\Sales\data\orders.dat  

## Batchsize = batch_size
The batch size of the data that should be inserted into the table in each transaction. If an error occurs, SQL Server commits the successfully processed data or rolls back if needed. This setting significantly affects performance.

## Check_constraints
Defines all considerations and constraints that must be enforced on the destination table. If this option is not specified, primary and foreign key constraints defined in the source data will not be considered. If a field does not have a defined value, the `Bulk Insert` command inserts a null value into the table.

## CODEPAGE = { 'ACP' | 'OEM' | 'RAW' | 'code_page' }
Defines the data `CodePage`. This option is relevant when handling `CHAR`, `VARCHAR`, or `TEXT` data with lengths greater than 127 or less than 32.

# CODEPAGE Values in SQL Server

| CODEPAGE Value  | Description |
|---------------|-------------|
| **ACP**        | Columns of `char`, `varchar`, or `text` data type are converted from the ANSI/Microsoft Windows code page (ISO 1252) to the SQL Server code page. |
| **OEM (default)** | Columns of `char`, `varchar`, or `text` data type are converted from the system OEM code page to the SQL Server code page. |
| **RAW**        | No conversion from one code page to another occurs. `RAW` is the fastest option. |
| **code_page**  | Specific code page number, for example, `850`. |

> **Note:** Versions prior to SQL Server 2016 (13.x) do not support code page **65001** (UTF-8 encoding).
DATAFILETYPE = { 'char' | 'native' | 'widechar' | 'widenative' }
This setting defines the type of data file.

DATA_SOURCE = 'data_source_name'
Specifies the name of the data source.

ERRORFILE = 'error_file_path'
This file stores rows that failed to import into the destination due to errors. It is created during execution, and if it already exists, an error will occur.

ERRORFILE_DATA_SOURCE = 'errorfile_data_source_name'
Defines the data source where erroneous records should be stored. This setting was introduced in SQL Server 2017.

FIRSTROW = first_row
Specifies the first row to be loaded. `Bulk Insert` does not have an option to ignore headers.

FIRE_TRIGGERS
Determines whether an `INSERT` trigger should fire for each inserted row in the destination table. If not set, triggers will not execute.

FORMATFILE_DATA_SOURCE = 'data_source_name'
Defines the format of the data source.

KEEPIDENTITY  
KEEPNULLS  
Specifies whether empty values should be treated as `NULL` during import.

KILOBYTES_PER_BATCH = kilobytes_per_batch
Defines the approximate number of kilobytes per batch.

LASTROW = last_row
Sets the number of last rows to be loaded. The default is `0`, meaning all rows will be imported.

MAXERRORS = max_errors
Defines the maximum number of errors that can be encountered before the `Bulk Insert` process is canceled. Every failed row increases the error count, and reaching the max limit stops execution. The default value is `10`.

ORDER ( { column [ ASC | DESC ] } [ ,... n ] )
Specifies the sorting order. If the imported data is sorted by the primary key, performance improves significantly.

ROWS_PER_BATCH = rows_per_batch
Estimates the approximate number of rows in the data file.

## Input File Settings

FORMAT = 'CSV'
Defines the format of the input file.

FIELDQUOTE = 'field_quote'
Specifies the character used for quoting in a CSV file. The default is `"`.

FORMATFILE = 'format_file_path'
Used when the number of columns in the file differs from the designed table, when ordering varies, or when a different delimiter is used.

FIELDTERMINATOR = 'field_terminator'
Defines the field separator, defaulting to `\t` (Tab).

ROWTERMINATOR = 'row_terminator'
Defines the row separator, defaulting to `\r\n` (Newline).

## Bulk Insert Example

To demonstrate `Bulk Insert`, a large dataset is needed. You can download sample data from [Click here](https://www.microsoft.com).  

First, a table must be created to store the imported data. The structure of this table should match the input data model. It is recommended to include control columns such as `ImportDate` to track historical records.

Create a CSV file named **bulkInsert.csv** and save the following data:
9;John;Mackay;58 Lamberts Branch Road;123777898 10;Aaron;Gallagher;2218 Smith Road;556814332  


To transfer the CSV data into the database, use:

```sql
BULK INSERT Customers
FROM 'C:\Users\user\Desktop\WEB\bulkInsert.csv'
WITH (
    FORMAT = 'CSV',
    FIELDTERMINATOR= ';',
    ROWTERMINATOR = '\n',
    KEEPIDENTITY,
    MAXERRORS = 2
);
```
After execution, all records in the CSV file will be imported into the Customers table.  

