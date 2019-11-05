## Ingestion of Streaming Sources into Snowflake using Snowpipe - Workshop Steps

### Step 1 create a snowflake account
- Create a free Snowflake account (you can get a Snowflake account with limited free credits for AWS, Azure) from below instructions

- Please go ahead and create a free snowflake account if you do not have one at the link below. Laura R/Snowflake team has built a custom link for this event, [Meetup Snowflake Page](https://www.snowflake.com/event/snowflake-user-group-atlanta-11062019/)

- Click on the Right top corner of the SNOWFLAKE USER GROUP – ATLANTA page,  *Start for Free* and register.

### Step 2 Getting familiarized with Snowflake in 20 minutes
- Once you are registered, go to this link, [Snowflake in 20 minutes](https://docs.snowflake.net/manuals/user-guide/getting-started-tutorial.html) to go over a tutorial for getting started with snowflake. 

### Step 3 Download the code to generate streaming events to write to cloud file storage (below code writes to AWS S3 bucket) 
- Download the Data Streaming Code from this [github repo](https://github.com/hashmapinc/socket_el). Once the code is working, you should be able to stream json files to the AWS repository

### Step 4 Setup instructions for connecting s3 file streaming event notifications to snowpipe

### Step 4 Snowflake Snowpipe instructions for streaming the data into Snowflake from cloud (AWS in this case)

> — Create a database, warehouse and a schema for this exercise;
> CREATE OR REPLACE  WAREHOUSE SNFL_ATL_MEETUP_WH;
> CREATE OR REPLACE DATABASE SNFL_ATL_MEETUP_DB;
>  CREATE OR REPLACE  SCHEMA SNFL_ATL_MEETUP_SCHEMA;
> 
> --set the context of the worksheet to meetup warehouse and schema
> USE WAREHOUSE SNFL_ATL_MEETUP_WH;
> USE SCHEMA SNFL_ATL_MEETUP_DB.SNFL_ATL_MEETUP_SCHEMA;
> 
> -- Create a file format
> CREATE OR REPLACE FILE FORMAT SNFL_SP_JSON_FORMAT
>   TYPE = 'json';
> 
> --create an external stage to cloud storage eg., below is for AWS
> --please use unique stage name 
> 
> CREATE OR REPLACE STAGE SNFL_SP_DEMO_STAGE_trial 
>   url='s3://s3_bucketURL'
>   CREDENTIALS = (aws_role = 'awsroleKey')
>   FILE_FORMAT = SNFL_SP_JSON_FORMAT
>   encryption=(type='AWS_SSE_KMS' kms_key_id = 'aws/key');
> 
> DESC STAGE SNFL_ATL_MEETUP_DB.SNFL_ATL_MEETUP_SCHEMA.snfl_sp_demo_stage_trial;  
> 
> //get contents of the stage
> LIST @SNFL_ATL_MEETUP_DB.SNFL_ATL_MEETUP_SCHEMA.snfl_sp_demo_stage_trial;
> 
> //create table
> --please use unique table name 
> 
> CREATE OR REPLACE TABLE SNFL_ATL_MEETUP_DB.SNFL_ATL_MEETUP_SCHEMA..SP_DEMO_JSON_TABLE_trial  
> (RAW_DATA VARIANT,
> INGESTIONG_TIME TIMESTAMP_LTZ(9));
> 
> // create pipe
> --please use unique pipe name 
> 
> CREATE OR REPLACE PIPE
>   SNFL_ATL_MEETUP_DB.SNFL_ATL_MEETUP_SCHEMA..SP_DEMO_trial 
>   AUTO_INGEST=TRUE
> AS 
>   COPY INTO 
>     SNFL_ATL_MEETUP_DB.SNFL_ATL_MEETUP_SCHEMA.SP_DEMO_JSON_TABLE_trial
>   FROM (
>     SELECT 
>       $1				          AS RAW_DATA, 
>       CURRENT_TIMESTAMP()  	AS INGESTION_TIME
>     FROM 
>     @SNFL_ATL_MEETUP_DB.SNFL_ATL_MEETUP_SCHEMA.SNFL_SP_DEMO_STAGE_trial
>   );

> //list pipes  
> SHOW PIPES;
> 
> SELECT * FROM SNFL_ATL_MEETUP_DB.SNFL_ATL_MEETUP_SCHEMA.SP_DEMO_JSON_TABLE_trial;
> SELECT COUNT(*) FROM SNFL_ATL_MEETUP_DB.SNFL_ATL_MEETUP_SCHEMA.SP_DEMO_JSON_TABLE_trial;
> 
> //useful commands
> --see pipe status
> 
> SELECT SYSTEM$PIPE_STATUS('SP_DEMO_trial');
> --copy historical data into table
> COPY INTO 
>     SNFL_ATL_MEETUP_DB.SNFL_ATL_MEETUP_SCHEMA.SP_DEMO_JSON_TABLE_trial
>   FROM (
>     SELECT 
>       $1				        AS RAW_DATA, 
>       CURRENT_TIMESTAMP()   AS INGESTION_TIME
>     FROM 
>       @SNFL_ATL_MEETUP_DB.SNFL_ATL_MEETUP_SCHEMA.SNFL_SP_DEMO_STAGE_trial
>   );
