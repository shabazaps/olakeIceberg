# Iceberg PostgreSQL Integration Project


This guide explains how to set up the required environment for running the Iceberg-based pipeline with PostgreSQL.

Clone the repo 

### bash code

git clone https://github.com/datazip-inc/olake.git

git clone https://github.com/apache/iceberg.git


1. PostgreSQL Setup

- Install PostgreSQL.
- Create a user named `admin`.
- Create a database named `olake_db`.
- Create a table `orders` and insert some sample data.

```sql
CREATE DATABASE olake_db;

\c olake_db

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(50),
    product VARCHAR(50),
    quantity INT,
    price DECIMAL(10, 2)
);

INSERT INTO orders (customer_name, product, quantity, price) VALUES
('Raj', 'Phone', 1, 10000.0),
('Sunil', 'Hedphone', 2, 2000.0),
('Aman', 'Keyboard', 3, 500.50),
('Abhiskeh', 'Pen', 5, 25.80),
('Guarav', 'Laptop', 1, 15000.05),
('Gautam', 'Tablet', 3, 8000.0),
('Riya', 'Watch', 4, 2000.45),
('Khushi', 'Charger', 5, 500.0),
('Sanjay', 'Microphone', 2, 2500.0),
('Sumit', 'Camera', 1, 10000.90),
('Aditya', 'Monitor', 3, 6000.60),
('Rakesh', 'Mouse', 3, 700.0);

2. Environment Installation
Install the following components:

Apache Spark: version 3.3.2

Python: version 3.12.3

Go: Version 1.22.2 (linux/amd64)

Apache Hadoop: version 3.3.6

OpenJDK: version 17.0.15 (release date: 2025-04-15)


3. Environment Variables
Ensure that the following environment variables are set correctly:

### bash code

export JAVA_HOME=/path/to/openjdk-17
export HADOOP_HOME=/path/to/hadoop
export SPARK_HOME=/path/to/spark
export PATH=$JAVA_HOME/bin:$HADOOP_HOME/bin:$SPARK_HOME/bin:$PATH

4. JDBC Driver Setup
Download the PostgreSQL JDBC driver and place it inside Spark’s jars directory:

### bash code

wget https://jdbc.postgresql.org/download/postgresql-42.7.1.jar -P $SPARK_HOME/jars/

5. Running the Spark Job

### bash code

pyspark --jars /home/shabaz/DE/iceberg-spark-runtime-3.2_2.12-1.4.2.jar

spark = SparkSession.builder \
    .appName("Iceberg_data") \
    .config("spark.sql.catalog.hadoop_catalog", "org.apache.iceberg.spark.SparkCatalog") \
    .config("spark.sql.catalog.hadoop_catalog.type", "hadoop") \
    .config("spark.sql.catalog.hadoop_catalog.warehouse", "hdfs:///user/hadoop/iceberg_warehouse") \
    .getOrCreate()

df = spark.sql("SELECT * FROM hadoop_catalog.my_db.my_table")
df.show()

Screenshots

Challenges & Improvements

1. I have previously installed Apache Spark verion 4.0.0 so I had to downgrade to version 3.3.2.
2. Identified and resolved missing JAR dependency (org.apache.iceberg.spark.SparkCatalog) issue.
3. wal2json plugin, was missing and logical replication failed due to unavailable decoding plugin in PostgreSQL.
4. Port conflicts, Docker containers failed to start due to ports (5432, 8181) already in use.
5. Wrong destination format caused repeated sync errors, Iceberg expected s3_path even when using HadoopFileIO
7. Version mismatches and unreachable REST endpoints.


