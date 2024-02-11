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

Clustering columns must be top-level, non-repeated columns
<li>DATE</li>
<li>BOOL</li>
<li>GEOGRAPHY</li>
<li>INT64</li>
<li>NUMERIC</li>
<li>BIGNUMERIC</li>
<li>STRING</li>
<li>TIMESTAMP</li>
<li>DATETIME</li>


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
## Partitioning vs Clustering
As mentioned before, you may combine both partitioning and clustering in a table, but there are important differences between both techniques that you need to be aware of to decide what to use for your specific scenario:
<div>
<img src="https://github.com/amal572/data_engenering_week3/blob/main/image/clus_per.PNG">
</div>

## Machine Learning with BigQuery
BigQuery ML is a BQ feature that allows us to create and execute Machine Learning models using standard SQL queries, without additional knowledge of Python or any other programming languages and without the need to export data into a different system.

The pricing for BigQuery ML is slightly different and more complex than regular BigQuery. Some resources are free of charge up to a specific limit as part of the Google Cloud Free Tier. You may check the current pricing at this link.

BQ ML offers a variety of ML models depending on the use case, as the image below shows:
<div>
<img src="https://github.com/amal572/data_engenering_week3/blob/main/image/machine.PNG">
</div>
We will now create a few example queries to show how BQ ML works. Let's begin with creating a custom table:

```bash
CREATE OR REPLACE TABLE `taxi-rides-ny.nytaxi.yellow_tripdata_ml` (
  `passenger_count` INTEGER,
  `trip_distance` FLOAT64,
  `PULocationID` STRING,
  `DOLocationID` STRING,
  `payment_type` STRING,
  `fare_amount` FLOAT64,
  `tolls_amount` FLOAT64,
  `tip_amount` FLOAT64
) AS (
  SELECT passenger_count, trip_distance, CAST(PULocationID AS STRING), CAST(DOLocationID AS STRING), CAST(payment_type AS STRING), fare_amount, tolls_amount, tip_amount
  FROM `taxi-rides-ny.nytaxi.yellow_tripdata_partitoned`
  WHERE fare_amount != 0
);
```
BQ supports feature preprocessing, both manual and automatic.
A few columns such as PULocationID are categorical but are represented with integer numbers in the original table. We cast them as strings to get BQ to automatically preprocess them as categorical features that will be one-hot encoded.
Our target feature for the model will be tip_amount. We drop all records where tip_amount equals zero to improve training.

Let's now create a simple linear regression model with default settings:

```bash
-- CREATE MODEL WITH DEFAULT SETTING
CREATE OR REPLACE MODEL `de-zoomcamp-412301.ny_taxi.tip_model`
OPTIONS
(model_type='linear_reg',
input_label_cols=['tip_amount'],
DATA_SPLIT_METHOD='AUTO_SPLIT') AS
SELECT
*
FROM
`de-zoomcamp-412301.ny_taxi.yellow_tripdata_ml`
WHERE
tip_amount IS NOT NULL;

```

The CREATE MODEL clause will create the taxi-rides-ny. nytaxi.tip_model model. The OPTIONS() clause contains all the necessary arguments to create our model. model_type='linear_reg' specifies that we will create a linear regression model. input_label_cols=['tip_amount'] informs BQ that our target feature is tip_amount. For linear regression models, target features must be real numbers. DATA_SPLIT_METHOD='AUTO_SPLIT' automatically splits the dataset into train/test datasets. The SELECT statement indicates which features need to be considered for training the model. Since we already created a dedicated table with all the needed features, we simply select them all. Running this query may take several minutes. After the query runs successfully, the BQ explorer in the side panel will display all available models (just one in our case) with a special icon. Selecting a model will open a new tab with additional information such as model details, 
training graphs, and evaluation metrics. We can also get a description of the features with the following query:

```bash
SELECT * FROM ML.FEATURE_INFO(MODEL `taxi-rides-ny.nytaxi.tip_model`);
```
The output will be similar to describe() in Pandas.

Model evaluation against a separate dataset is as follows:

```bash
-- EVALUATE THE MODEL
SELECT
*
FROM
ML.EVALUATE(MODEL `de-zoomcamp-412301.ny_taxi.tip_model`,
(
SELECT
*
FROM
`de-zoomcamp-412301.ny_taxi.yellow_tripdata_ml`
WHERE
tip_amount IS NOT NULL
));
```
where the root means square and Pearson correlation will be calculated. The next step in ML is to predict the target using a dataset. In our modules, we will be using back the same dataset used for training for demonstration purposes. In real cases, it is normally tested with a dataset that is not previously used for training.
This could be achieved via

```bash
-- PREDICT THE MODEL
SELECT
*
FROM
ML.PREDICT(MODEL `de-zoomcamp-412301.ny_taxi.tip_model`,
(
SELECT
*
FROM
`de-zoomcamp-412301.ny_taxi.yellow_tripdata_ml`
WHERE
tip_amount IS NOT NULL
));
```
Upon the prediction, you might wonder how to explain the model using the key features that influence the most in our model. 
To extract those features, we can run an SQL query like
```bash
-- PREDICT AND EXPLAIN
SELECT
*
FROM
ML.EXPLAIN_PREDICT(MODEL `de-zoomcamp-412301.ny_taxi.tip_model`,
(
SELECT
*
FROM
`de-zoomcamp-412301.ny_taxi.yellow_tripdata_ml`
WHERE
tip_amount IS NOT NULL
), STRUCT(3 as top_k_features));

```
In ML, the improvement of model performance can be achieved by hyperparameter tuning. In BigQuery, a similar strategy can be done via
```bash
-- HYPER PARAM TUNNING
CREATE OR REPLACE MODEL `de-zoomcamp-412301.ny_taxi.tip_hyperparam_model`
OPTIONS
(model_type='linear_reg',
input_label_cols=['tip_amount'],
DATA_SPLIT_METHOD='AUTO_SPLIT',
num_trials=5,
max_parallel_trials=2,
l1_reg=hparam_range(0, 20),
l2_reg=hparam_candidates([0, 0.1, 1, 10])) AS
SELECT
*
FROM
`de-zoomcamp-412301.ny_taxi.yellow_tripdata_ml`
WHERE
tip_amount IS NOT NULL;
```
