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





