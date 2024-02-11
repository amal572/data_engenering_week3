## Data Engineering Zoomcamp - Week 3

In week 3 of Data Learning, We learned a lot of things and I want to explain about that 

## OLAP vs OLTP
In data engineering, there are two major concepts: Online analytical processing (OLAP) and Online Transaction Processing (OLTP), which serve different purposes in an organization. The table below summarises the difference between these two concept relatively well.

<div>
<img src="https://github.com/amal572/data_engenering_week3/blob/main/image/Capture.PNG">
</div>

In short, OLTP List day-to-day business transactions but OLAP for multi-dimension view of enterprise data 

<div>
<img src="https://github.com/amal572/data_engenering_week3/blob/main/image/1_FKW_m0LzTb2lEYmwg5_DNw.webp">
</div>

## What is a Data Warehouse?

The data warehouse is known to be an OLAP solution that is widely used for organizations where the data from the sources are cleaned and transformed in the staging area, before being loaded here for reporting and data analysis purposes. Taking a step further, the data processed in the data warehouse can be served into Data Marts, which a smaller database systems that are used for different teams within an organisation as shown in the flow chart below.

<li> Data Warehouse: Picture a neatly organized library with well-cataloged books. Examples include Google's BigQuery, Amazon's Redshift, and Azure's SQL Data Warehouse.</li>
<li> Data Lake: Imagine a vast storage room with books in boxes, requiring some digging to find what you need, akin to object storage. Example services include Google's Cloud Storage, Amazon's S3 (Simple Storage Service), and Azure Data Lake Storage.</li>
<li>Data Mart: Think of it as a specialized section of a library dedicated to a specific subject.</li>
<li>Data Lakehouse: A hybrid that combines the structured organization of a Data Warehouse with the expansive storage of a Data Lake.</li>


<div>
<img src="https://github.com/amal572/data_engenering_week3/blob/main/image/dataWarehouse.PNG">
</div>

## What is BigQuery?
BigQuery is a serverless data warehouse solution offered by Google Cloud Platform which organization doesn’t need to take care of the infrastructure that processes the data and offers a relatively fast service in reporting within an organization. Below are some key features of BigQuery

BigQuery supports external data sources, although the data is not stored in the BigQuery storage. For instance, it can be directly queried from data stored in Google Cloud Storage upon the creation of the external table.

An external table is a table that functions like a standard BigQuery table with table schema stored in BQ but the data itself is external. No preview of data is available in that sense. You might create an external table from a .csv or .parquet file from Google Cloud Storage in this way:

```bash

 -- Creating an external table referring to gcs path
CREATE OR REPLACE EXTERNAL TABLE `de-zoomcamp-412301.ny_taxi.external_yellow_tripdata` 
OPTIONS ( format = 'parquet',
    uris = ['gs://module-3-zoomcamp/yellow/yellow_tripdata_2020-*.parquet',
    'gs://module-3-zoomcamp/yellow/yellow_tripdata_2019-*.parquet']);

```
Be aware that the estimation of bytes of data for the query from the external table is not available. In regards to that, you can import an external table in BQ as a regular table. For example:

```bash
CREATE OR REPLACE TABLE `de-zoomcamp-412301.nytaxi.yellow_tripdata_non_partitoned` AS
SELECT * FROM `de-zoomcamp-412301.nytaxi.external_yellow_tripdata`;
```

## Partitioning
The BigQuery table can be partitioned into smaller tables which improve the query performance, thus saving cost. For instance, if we often query the data by date, we then can further partition our table by date, hence we can do a sub-query based on the date we are particularly interested in. In general, you can partition a table by:
Here is the example of partitioning data by datetime column,

```bash
-- Create a partitioned table from external table
CREATE OR REPLACE TABLE de-zoomcamp-412301.ny_taxi.yellow_tripdata_partitoned
PARTITION BY
  DATE(pickup_date) AS
SELECT * FROM de-zoomcamp-412301.ny_taxi.external_yellow_tripdata_corrected;
```
This partition have then cut down the bytes of data being scanned when we are doing the query, comparing between non-partitioned and partitioned table

```bash
-- Impact of partition
-- Scanning 1.6GB of data
SELECT DISTINCT(VendorID)
FROM de-zoomcamp-412301.ny_taxi.yellow_tripdata_non_partitoned
WHERE DATE(pickup_date) BETWEEN '2019-06-01' AND '2019-06-30';

-- Scanning ~106 MB of DATA
SELECT DISTINCT(VendorID)
FROM de-zoomcamp-412301.ny_taxi.yellow_tripdata_partitoned
WHERE DATE(pickup_date) BETWEEN '2019-06-01' AND '2019-06-30';
```

You may also check the rows of each partition with the query as follows

```bash
-- Let's look into the parties
SELECT table_name, partition_id, total_rows
FROM `ny_taxi.INFORMATION_SCHEMA.PARTITIONS`
WHERE table_name = 'yellow_tripdata_partitoned'
ORDER BY total_rows DESC;

```

The query above gives you a quick view of how the partitions look like your table. Note that the partition limit is 4000. The figure below shows how the partitioning in BQ looks like
<div>
<img src="https://github.com/amal572/data_engenering_week3/blob/main/image/perm.PNG">
</div>

## Clustering

Clustering is a technique used to enhance processing efficiency by querying specific subsets of data. It involves selecting columns that group related data together, with up to four columns available for clustering. The sequence of these columns is crucial as it determines the sorting order of the data.

For example, consider the dataset partitioned initially by the date column and then clustered by tags, as illustrated in the figure below. This approach significantly enhances query performance for both filtering and aggregation tasks. However, it's important to note that the benefits of partitioning and clustering may not be pronounced for tables smaller than 1GB in size.

Clustering columns must be top-level and non-repetitive. Supported data types for clustering include DATE, BOOL, GEOGRAPHY, INT64, NUMERIC, BIGNUMERIC, STRING, TIMESTAMP, and DATETIME.

<div>
<img src="https://github.com/amal572/data_engenering_week3/blob/main/image/cluster.PNG">
</div>

An example used in the module demonstrates how partitioning and clustering can be performed together as follows:

```bash
CREATE OR REPLACE TABLE de-zoomcamp-412301.ny_taxi.yellow_tripdata_partitoned_clustered
PARTITION BY DATE(pickup_date)
CLUSTER BY VendorID AS
SELECT * FROM de-zoomcamp-412301.ny_taxi.external_yellow_tripdata_corrected;

```
The total bytes of a query with permission and clustering, compared to one without, have been reduced from 1.1GB to 864.5MB of data in the table.
```bash
-- Query scans 1.1 GB
SELECT count(*) as trips
FROM de-zoomcamp-412301.ny_taxi.yellow_tripdata_partitoned
WHERE DATE(pickup_date) BETWEEN '2019-06-01' AND '2020-12-31'
  AND VendorID=1;


-- Query scans 864.5 MB
SELECT count(*) as trips
FROM de-zoomcamp-412301.ny_taxi.yellow_tripdata_partitoned_clustered
WHERE DATE(pickup_date) BETWEEN '2019-06-01' AND '2020-12-31'
  AND VendorID=1;
```







