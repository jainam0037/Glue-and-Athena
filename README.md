# AWS Glue and Athena ETL Process: COVID-19 Data Transformation and Querying

This tutorial demonstrates how to use **AWS Glue** to run an ETL (Extract, Transform, Load) job on COVID-19 data stored in an **S3 bucket** and how to query the transformed data using **AWS Athena**.

## Steps Overview:
1. [Downloading the Data](#step-1-downloading-the-data)
2. [Create IAM Role for AWS Glue](#step-2-create-iam-role-for-aws-glue)
3. [Setup S3 Buckets](#step-3-setup-s3-buckets)
4. [Create Glue Database and Crawler](#step-4-create-glue-database-and-crawler)
5. [Create and Run Glue ETL Job](#step-5-create-and-run-glue-etl-job)
6. [Monitor the Job and Retrieve Output](#step-6-monitor-the-job-and-retrieve-output)
7. [Running Crawler to Generate Data Catalog](#step-7-running-crawler-to-generate-the-data-catalog)
8. [Viewing Data in AWS Athena](#step-8-viewing-data-in-aws-athena)
9. [Filtering Data Using Athena Queries](#step-9-filtering-data-using-athena-queries)
10. [Transform Data with AWS Glue ETL Job](#step-10-transform-data-with-aws-glue-etl-job)
11. [Running the Job and Storing Output](#step-11-running-the-job-and-storing-output)
12. [Creating a Crawler for Transformed Data](#step-12-creating-a-crawler-for-transformed-data)
13. [Querying the Transformed Data in Athena](#step-13-querying-the-transformed-data-in-athena)

---

## Step 1: Downloading the Data
- Download the **World Health Organization's COVID-19** dataset as a CSV file.
- Convert the CSV to JSON using an online tool.
- Remove any unnecessary array brackets in the JSON file using a text editor (e.g., VS Code).

## Step 2: Create IAM Role for AWS Glue
- Go to the **IAM dashboard** on AWS and create a new role for Glue.
  - **Role Type**: AWS Glue can call other AWS services on your behalf.
  - **Permissions**: Assign PowerUserAccess to the role.
  - **Role Name**: `covid-glue-role`.

## Step 3: Setup S3 Buckets
Create the following S3 buckets:
1. **Input Data Bucket**: `covid-input-data` (store the original JSON file).
2. **Output Data Bucket**: `covid-output-data` (store the transformed data).
3. **Script Bucket**: `covid-scripts` (store Glue scripts).

Upload the JSON file to the input data bucket.

## Step 4: Create Glue Database and Crawler
1. **Create Database**: Go to the Glue dashboard and create a new database (e.g., `covid-db`).
2. **Create a Crawler**:
   - Set the data source as the S3 input data bucket.
   - Select the IAM role (`covid-glue-role`).
   - Set the target database to `covid-db`.
   - Run the crawler to inspect the JSON data and create a table in the Glue data catalog (e.g., `covid-input-table`).

## Step 5: Create and Run Glue ETL Job
1. Go to Glue Studio and create an **ETL job**.
   - **Source**: Select the Glue Data Catalog and choose the table created by the crawler.
   - **Action**: Add transformations (e.g., dropping unwanted columns).
   - **Target**: Set the output to an S3 bucket (`covid-output-data`) and choose CSV format with gzip compression.
   - **Job Details**:
     - Select the IAM role (`covid-glue-role`).
     - Use the **Spark** engine with **Python 3**.
     - Set the script path to the script in `covid-scripts`.
2. Run the job.

## Step 6: Monitor the Job and Retrieve Output
- Monitor the ETL job from the Monitoring tab.
- Check the S3 output bucket to verify that a CSV file has been generated.

## Step 7: Running Crawler to Generate the Data Catalog
1. Create a new crawler for the input data in Glue.
   - Set the source as the `covid-input-data` S3 bucket.
   - Use the `covid-db` database and the `covid-glue-role`.
2. Run the crawler to generate the schema for the input data.

## Step 8: Viewing Data in AWS Athena
1. Navigate to the newly created table in Glue.
2. Select **"View Data"** to open Athena.
3. Configure the results storage location by setting up a new S3 bucket (e.g., `covid-query-results-bucket`).
4. Run a query in Athena to fetch the data.

## Step 9: Filtering Data Using Athena Queries
1. Modify the Athena query to filter rows where `new_cases > 0` and `new_deaths > 0`.
2. Run the query to retrieve the filtered data and download the results as a CSV.

## Step 10: Transform Data with AWS Glue ETL Job
1. Create a new Glue ETL job to remove unnecessary columns from the dataset.
   - Set the input as the S3 bucket (`covid-input-data`) with CSV format.
   - Perform transformations to remove unwanted columns (e.g., cumulative cases, cumulative deaths).
   - Set the output as the S3 bucket (`covid-output-data`).
2. Save and run the job.

## Step 11: Running the Job and Storing Output
1. After the ETL job completes, download the transformed CSV file from the S3 bucket.
2. Verify that it contains only the necessary columns (e.g., `new_cases`, `new_deaths`).

## Step 12: Creating a Crawler for Transformed Data
1. Create a new crawler in Glue for the transformed data.
   - Set the source as the S3 bucket (`covid-output-data`).
   - Use the existing database (`covid-db`).
2. Run the crawler to generate the new table schema.

## Step 13: Querying the Transformed Data in Athena
1. In Athena, view the new table created by the crawler.
2. Run queries to filter and analyze the transformed data:
   ```sql
   SELECT * 
   FROM "covid-db"."covid-output-data-table"
   WHERE "new_cases" > 0 AND "new_deaths" > 0;
