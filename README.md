
---

# **Logistics-Data-Lake-Analytics-Hub**  

## **Project Overview**  
This project automates the ingestion, transformation, and management of logistics data in a cloud-based data warehouse using **Google Cloud Platform (GCP)**. The pipeline integrates **Apache Airflow (Cloud Composer) and Apache Hive (GCP Dataproc)** to orchestrate an ETL workflow that processes incoming logistics data from **Google Cloud Storage (GCS)**.  
![Architecture](https://github.com/user-attachments/assets/4fe6d6db-bc37-49b1-ac27-e3c27e65b22a)

The workflow is triggered when new logistics data files arrive in GCS. The process includes:  
✅ **Creating a Hive Database** on GCP Dataproc.  
✅ **Defining External and Partitioned Tables** in Hive for structured storage.  
✅ **Processing and Ingesting Data** dynamically into partitioned tables.  
✅ **Archiving Processed Files** to a separate GCS bucket for long-term storage.  
✅ **Automated Scheduling** using Apache Airflow to run the pipeline daily.  

This setup leverages the **scalability** of GCP for big data processing and the **flexibility** of Apache Airflow for workflow automation.  

---

## **Architecture**  
The solution utilizes various GCP services for seamless data ingestion, transformation, and storage:  

1️⃣ **Google Cloud Storage (GCS):** Stores raw logistics data files.  
2️⃣ **Apache Hive (GCP Dataproc):** Manages structured data storage and querying.  
3️⃣ **Apache Airflow (Cloud Composer):** Orchestrates the ETL workflow.  
4️⃣ **Google Cloud Dataproc:** Executes Hive queries for data processing.  

📌 **Data Flow**  
🔹 **Raw Data:** Logistics files are uploaded to the GCS bucket (`logistics-raw-gds`).  
🔹 **Processing:** Apache Airflow detects new files and triggers Hive processing.  
🔹 **Structured Storage:** Data is partitioned and stored in Hive tables for optimized querying.  
🔹 **Archival:** Processed files are moved to an archive bucket (`logistics-archive-gds`).  

---

## **Tech Stack**  
- **Cloud Platform:** Google Cloud Platform (GCP)  
- **Orchestration:** Apache Airflow (Cloud Composer)  
- **Data Processing:** Apache Hive on GCP Dataproc  
- **Storage:** Google Cloud Storage (GCS)  
- **Scripting:** Python for Airflow DAG  

---

## **Implementation Details**  

### **1️⃣ File Detection in GCS**  
🔍 The Airflow DAG starts by sensing new files in the `logistics-raw-gds` bucket using **GCSObjectsWithPrefixExistenceSensor**.  
- **Trigger:** Files prefixed with `logistics_` in `input_data/`.  
- **Mode:** Polling every 30 seconds for 5 minutes.  

```python
sense_logistics_file = GCSObjectsWithPrefixExistenceSensor(
    task_id='sense_logistics_file',
    bucket='logistics-raw-gds',
    prefix='input_data/logistics_',
    mode='poke',
    timeout=300,
    poke_interval=30,
    dag=dag
)
```

---

### **2️⃣ Hive Database and Table Creation**  
🏗️ Once a new file is detected, the DAG initiates **Hive database creation** in GCP Dataproc.  

#### **🔹 Creating the Hive Database**
```sql
CREATE DATABASE IF NOT EXISTS logistics_db;
```
This ensures all logistics data is stored in a structured format.  

#### **🔹 Defining the External Hive Table**
```sql
CREATE EXTERNAL TABLE IF NOT EXISTS logistics_db.logistics_data (
    delivery_id INT,
    `date` STRING,
    origin STRING,
    destination STRING,
    vehicle_type STRING,
    delivery_status STRING,
    delivery_time STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION 'gs://logistics-raw-gds/input_data/'
tblproperties('skip.header.line.count'='1');
```
- Stores **raw logistics data** in an external table.  
- **Delimited by commas**, and header rows are skipped.  

---

### **3️⃣ Creating a Partitioned Table for Efficient Querying**  
📂 Data is moved into a **partitioned Hive table** for optimized querying.  
```sql
CREATE TABLE IF NOT EXISTS logistics_db.logistics_data_partitioned (
    delivery_id INT,
    origin STRING,
    destination STRING,
    vehicle_type STRING,
    delivery_status STRING,
    delivery_time STRING
)
PARTITIONED BY (`date` STRING)
STORED AS TEXTFILE;
```
- Data is **partitioned by date**, making queries faster and more efficient.  
- This improves **data retrieval speed** for large datasets.  

---

### **4️⃣ Data Loading with Dynamic Partitioning**  
🔥 Data is moved from the external table to the partitioned table using dynamic partitioning.  

```sql
SET hive.exec.dynamic.partition = true;
SET hive.exec.dynamic.partition.mode = nonstrict;

INSERT INTO logistics_db.logistics_data_partitioned PARTITION(`date`)
SELECT delivery_id, origin, destination, vehicle_type, delivery_status, delivery_time, `date`
FROM logistics_db.logistics_data;
```
💡 **Why dynamic partitioning?**  
✅ Optimized storage  
✅ Faster queries  
✅ Efficient data organization  

---

### **5️⃣ Archiving Processed Files**  
📁 Once the data is loaded successfully, the raw files are moved to an **archive bucket** for retention and auditing.  

```bash
gsutil -m mv gs://logistics-raw-gds/input_data/logistics_*.csv gs://logistics-archive-gds/
```
- Ensures **processed data** is stored safely.  
- Prevents redundant processing of the same files.  

---

## **🔄 Workflow Automation with Apache Airflow**  
The DAG defines dependencies between tasks, ensuring a **sequential execution flow**:  
1️⃣ **Sense New File** → 2️⃣ **Create Hive DB** → 3️⃣ **Define External Table**  
4️⃣ **Create Partitioned Table** → 5️⃣ **Load Data** → 6️⃣ **Archive Processed Files**  

```python
sense_logistics_file >> create_hive_database >> create_hive_table \
>> create_partitioned_table >> set_hive_properties_and_load_partitioned >> archive_processed_file
```
✅ **Automated Scheduling:** Runs daily to process new logistics data.  
✅ **Error Handling:** Logs failures for easy debugging.  
✅ **Cloud-Native:** Fully managed with GCP services.  

---

## **🚀 Key Takeaways**  
🔹 **End-to-End Automation:** The pipeline detects, processes, and archives data without manual intervention.  
🔹 **Optimized for Big Data:** Partitioning ensures efficient data retrieval.  
🔹 **Scalable & Cloud-Native:** Fully managed on GCP, allowing seamless expansion.  
🔹 **Robust Data Management:** Hive tables enable structured analysis for logistics insights.  

This solution **enhances logistics data processing efficiency**, **improves scalability**, and **enables advanced analytics** for better business decision-making. 🚀  

---
