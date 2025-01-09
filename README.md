# Mini-Project: Hadoop CoffeeShop

This project is the project in course "Road to Data Engineer 3.0 Bootcamp" by **Data TH**.
![alt text](image-1.png)
![alt text](image-2.png)

## Introduction
The objective of this mini-project is to simulate a complete big data pipeline for a fictional coffee shop, leveraging the Hadoop ecosystem and associated tools. The project aims to:

**1. Simulate Transactional Data:**<br>
Use shell scripts to mock real-time transactional data, representing coffee shop operations such as orders, payments, and inventory updates.

**2. Ingest Data into HDFS and HBase:**<br>
Employ Apache Flume to capture and transport the simulated transactional data into HDFS (Hadoop Distributed File System) and HBase, ensuring scalability and durability for large datasets.

**3. Data Cleaning and Processing:**<br>
Use Apache Spark for cleaning and transforming the ingested data, preparing it for efficient querying and analysis.

**4. Query and Analyze Data:**<br>
Utilize Apache Hive to query the cleaned data, enabling structured query execution for insights such as sales trends, popular products, and inventory usage.

**5. Workflow Scheduling:**<br>
Implement Apache Oozie to automate and schedule the end-to-end pipeline, ensuring timely execution of data ingestion, cleaning, and querying processes.

**6. Cloud Infrastructure:**<br>
Deploy the entire pipeline on Google Cloud Platform (GCP) using a VM Compute Engine, showcasing the power and flexibility of cloud-based big data infrastructure.

This project demonstrates the integration of various Hadoop ecosystem components for building a robust and scalable big data pipeline, emphasizing automation, data quality, and analytics.


## Project system design
![alt text](image.png)

## Methodology 
1. Set up GCP VM and Docker *[HadoopInstallation](./HadoopDockerInstallationGuide.pdf)*<br>
2. Start all HDFS Function via Cloudera<br>
![alt text](./pict/image-1.png)
3. Move the Folder *[data](./data/)* to Docker Container then execute the container
![alt text](./pict/image-2.png)
![alt text](./pict/image-3.png)
4. **Batch Layer.**<br>
Put File from source to HDFS sink 
    - create /tmp/file/sink in HDFS to store csv File<br>
    ` hadoop fs -mkdir /tmp/file`
    ` hadoop fs -mkdir /tmp/file/sink`
    - Put File from client to HDFS
    `hadoop fs -put /data/file/source//customer.csv /tmp/file/sink`
    ![alt text](./pict/image-4.png) 

5. **Speed Layer**.<br>
- use src_sys.sh to mocking transactional data of coffeeshop *[mocking_script.sh](./data/flume/src_sys.sh)*<br>
- create source folder of HDFS and HBASE relate with flume config file.<br> 
*[flume_hbase.conf](./data/flume/source/flume_hbase.conf)* <br> *[flume_hdfs.conf](./data/flume/source/flume_hdfs.conf)*
![alt text](./pict/image-5.png)
![alt text](./pict/image-6.png)

- create sink on HDFS side to kept file via config
![alt text](./pict/image-7.png)
- create hbase a table to keep file 
![alt text](./pict/image-8.png)
**note** This is from hbase config file
![alt text](./pict/image-9.png)
![alt text](./pict/image-10.png)

- Run Flume config with nohub command to run in the background.<br>
![alt text](./pict/image-13.png)
![alt text](./pict/image-12.png)

- Run Transactional Script to feed Flume
![alt text](./pict/image-17.png)
![alt text](./pict/image-15.png)

- Checks Running jobs
![alt text](./pict/image-16.png)

- Check Sink side of HDFS and table of HBASE
![alt text](./pict/image-18.png)
![alt text](./pict/image-19.png)

6. **Query Part**<br>
- Open Hive from HUE 
![alt text](./pict/image-20.png)
- Create External Table for customer.csv on HDFS
*[create_hive_customers.sql](./data/sql/create_hive_customers.sql)*
![alt text](./pict/image-21.png)
![alt text](./pict/image-22.png)
- Create External Table for HBASE data
*[create_hive_transactions](./data/sql/create_hive_transactions.sql)*
![alt text](./pict/image-23.png)
![alt text](./pict/image-24.png)

7. **Clening Data with Apache Spark**<br>
7.1 **Spark**
- `spark-submit /data/spark/spark.py`<br>
![alt text](./pict/image-25.png)
- The Spark file locate *[here](./data/spark/spark.py)*
After running keep file at `/tmp/default/customers_cln`
![alt text](./pict/image-26.png)
- Then Create External Table on Hive for Cleaned customers data again with *[create_hive_customers_cln](./data/sql/create_hive_customers_cln.sql)*.
![alt text](./pict/image-27.png)
![alt text](./pict/image-28.png)
----------------------------
7.2 **Spark Streaming**
- The spark streaming file locate *[spark_streaming.py](./data/spark_streaming/spark_streaming.py)*
![alt text](./pict/image-30.png)
- Must run with nohup because of streaming file type must continue running
![alt text](./pict/image-29.png)
- Then Create External Table on HBASE Table for Cleaned transactional data again with *[create_hive_transactions_cln.sql](./data/sql/create_hive_transactions_cln.sql)*.
![alt text](./pict/image-31.png)

8. **Query The data**<br>
- Create Loyalty Table to wait for insert data from oozie, can execute the script via root user by<br>
`hive -f /data/sql/create_hive_loyalty.sql`
![alt text](./pict/image-32.png) 

- OOZIE 
    - use *[insert_hive_loyalty.sql](./data/sql/insert_hive_loyalty.sql)* to put on oozie and set parameter to data_dt
    ![alt text](./pict/image-33.png)
    ![alt text](./pict/image-34.png)
    ![alt text](./pict/image-35.png)
    - Insert Script To HDFS 
    ![alt text](./pict/image-36.png)
    - Put the Script to OOZIE <br>
    ![alt text](./pict/image-37.png)
    - Create Coordinator <br>
    ![alt text](./pict/image-38.png)
    - After Save Must Commit again!!! <br>
    ![alt text](./pict/image-39.png)
    - Result!!! <br>
    ![alt text](./pict/image-40.png)
    ![alt text](./pict/image-41.png)


## Technologies
**Cloud Infrastructure**
- Google Cloud Platform (GCP)
Apache Hadoop
- HDFS (Hadoop Distributed File System)
- HBase (NoSQL database)
- Apache Flume (Data ingestion)
- Apache Spark (Batch and Streaming data processing)
- Apache Hive (Data querying and analytics)
- Apache Oozie (Workflow orchestration and scheduling)

**Development Tools**
- Docker (Hadoop ecosystem containerization)
- Hue (Web interface for Hive and HDFS operations)

--------------------------

## Languages
- `Shell Scripting`: For mocking data and operational scripts.
- `Python`: For Spark batch and streaming processing scripts.
- `SQL`: For creating and managing Hive tables and querying data
