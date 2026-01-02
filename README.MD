# Data source
Enrollments.xlsx

[Global Longitudinal University Enrolment Dataset](https://www.kaggle.com/datasets/ramazannuhbalayev/glued-institutional-data-from-orginal-source/data)

Countries_GDP_1960-2020.csv

Provided data from College Board

# EDA.ipynb
Exploring the 2 datasets

# ETL.ipynb
Pipeline for Ingesting both datasets and producing a conjoined output file.


# Production Thinking

## Incremental Updates:
Because we have the hash_key we can do a database merge on the hashkey and only take new ones.

If old values can be updated we could impliment something like a digest_key that is a hash of all the values that can be changed over time and upsert new data items

## Data Quality Monitoring:
The low quality nature of the enrollments.xlsx data gives some pause to metrics.

Easy low hanging fruit could be

- Incoming data counts vs inserted data counts vs total stored counts
- Watch the number of records pruned incoming to watch if incoming source data is degrading or lacking completeness

## Scalability:

- If the incoming data is truly massive there is always pySpark on a cluster

- Handle each data set individually and use the database merge and hash checking to determine records

- Breaking the job into multiple steps

    1. Download new files
    2. Break the file into parts
    3. Process each part
        - This would need a digest_key to make sure we are not ingesting the same data multiple times
        - Would want to prioritize the highest quality data and maybe add a "source" column to determine
- If data stores can handle the data load break the steps into ELT VS ETL
    - Job #1 ingests the data into a database
    - Job #2 uses more powerful database machines to do the transformation

## Failure & Recovery:

- S3 option:
    - Use file locations
    - New files drop into a bucket
    - Each step moves the file to a new bucket after preforming its action
- SQS option:
    - using message queues to move ingestions
    - triggers insert new messages for work to be done
    - as work is done messages are acknowledged
    - Requeue settings on the Queue free un-acknowledged messages for retry