﻿# Car Rental Data Pipeline

 This project implements an end-to-end data pipeline for processing car rental data using Airflow, Snowflake, and Google Cloud Dataproc. The pipeline integrates data from Google Cloud Storage (GCS), processes it with PySpark on Dataproc, and manages Slowly Changing Dimensions (SCD) in Snowflake.


 ## Project Overview
 The Car Rental Data Pipeline processes daily customer data from car rental services and ingests it into a Snowflake data warehouse. The data is staged in Google Cloud Storage and processed using PySpark jobs on Google Cloud Dataproc. The pipeline implements a Slowly Changing Dimension (SCD) Type 2 methodology for the customer_dim table, ensuring accurate tracking of historical changes to customer data.

 ![image]![image](https://github.com/user-attachments/assets/2ddac8cd-db2a-433a-86a0-ff4ee5c634df)

 ![image]![image](https://github.com/user-attachments/assets/c3854d44-9ac1-4c00-bba2-439c5eff5627)



 ## Technologies Used
- Airflow: Orchestration of the data pipeline.
- Snowflake: Data warehouse for storing dimension and fact tables.
- Google Cloud Dataproc: PySpark processing for large-scale data operations.
- Google Cloud Storage (GCS): Data staging area for incoming data files.
- PySpark: For data transformation and integration with Snowflake.
- Python: Custom scripts and Airflow operators.
## Airflow DAG Overview
The Airflow DAG is defined in car_rental_data_pipeline. It orchestrates several tasks that handle the extraction, transformation, and loading (ETL) of car rental data.

## Tasks
- Get Execution Date (get_execution_date):

  * Fetches the execution date passed as a parameter or defaults to the current date.
  * The execution date is used to locate the appropriate data file in GCS.
- Merge Customer Dimension (merge_customer_dim):

  * Implements SCD Type 2 for the customer_dim table by comparing the current records with the incoming data. If there are any changes, the existing record is marked as non-current, and a new record is inserted.
- Insert Customer Dimension (insert_customer_dim):

  * Inserts new or updated records into the customer_dim table, following the SCD Type 2 methodology.
- Submit PySpark Job (submit_pyspark_job):

  * Submits a PySpark job to Google Cloud Dataproc that processes car rental data, interacts with Snowflake, and performs additional transformations.
 
## Database Schema
## Dimension Tables

- location_dim

  * Stores details of car rental locations (e.g., airports).
- car_dim

  * Stores details about rental cars, including make, model, and year.
- customer_dim

  * Stores customer information, implementing SCD Type 2 to track historical changes.
- date_dim

  * Stores date information, such as year, month, day, and quarter.

## Fact Table
- rentals_fact
  * Stores transactional rental data, including references to the customer, car, and location dimensions, as well as rental-specific metrics (e.g., total rental amount, duration).


## Setup and Configuration
- Google Cloud Dataproc Configuration
    * Create a Dataproc Cluster:

  * You will need a Dataproc cluster to run the PySpark jobs.
  * Define cluster settings in cluster.json
```sql
{
  "CLUSTER_NAME": "your-cluster-name",
  "PROJECT_ID": "your-project-id",
  "REGION": "your-region"
}

```

- Submit PySpark Job:
  * The PySpark job (spark_job.py) is stored in a GCS bucket (gs://snowflake_projects/spark_job/spark_job.py).
  * The pipeline passes the execution date to this job as an argument.
 
- Snowflake Setup
    * Create Snowflake Tables:
      Run the SQL scripts in Snowflake to create the required dimension and fact tables as shown in the customer_dim, car_dim, date_dim, location_dim, and rentals_fact sections of the code.
    * Snowflake Integration:
       Ensure that the Snowflake connection snowflake_conn_v2 is properly configured in Airflow.
The pipeline queries and writes to the Snowflake database.


## Work Flow Summary


  - Airflow DAG:

Triggers a pipeline to extract customer data from Google Cloud Storage (GCS), ingest it into Snowflake, and run PySpark jobs on Dataproc.
Manages the execution date dynamically using a parameter.
  - Snowflake Integration:

Stages customer data from GCS to Snowflake.
Updates the customer_dim table with an SCD2 merge.
Inserts new data into customer_dim.
  - Dataproc Job:

A PySpark job is submitted to Dataproc to process additional data, using the execution date as an argument.
