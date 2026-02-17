#  Banking Modern Data Stack

![Snowflake](https://img.shields.io/badge/Snowflake-29B5E8?logo=snowflake&logoColor=white)
![DBT](https://img.shields.io/badge/dbt-FF694B?logo=dbt&logoColor=white)
![Apache Airflow](https://img.shields.io/badge/Apache%20Airflow-017CEE?logo=apacheairflow&logoColor=white)
![Apache Kafka](https://img.shields.io/badge/Apache%20Kafka-231F20?logo=apachekafka&logoColor=white)
![Debezium](https://img.shields.io/badge/Debezium-EF3B2D?logo=apache&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![Git](https://img.shields.io/badge/Git-F05032?logo=git&logoColor=white)
![CI/CD](https://img.shields.io/badge/CI%2FCD-000000?logo=githubactions&logoColor=white)

---

##  Project Overview
This project demonstrates an **end-to-end modern data stack pipeline** for a **Banking domain**.  
We simulate **customer, account, and transaction data**, stream changes in real time, transform them into analytics-ready models, and visualize insights — following **best practices of CI/CD and data warehousing**.

 

---

##  Architecture  

<img width="5647" height="3107" alt="Architecture" src="https://github.com/user-attachments/assets/7521ea8a-451e-46ff-9db0-71dd6ddf8181" />


**Pipeline Flow:**
1. **Data Generator** → Simulates banking transactions, accounts & customers (via Faker).  
2. **Kafka + Debezium** → Streams change data (CDC) into MinIO (S3-compatible storage).  
3. **Airflow** → Orchestrates data ingestion & snapshots into Snowflake.  
4. **Snowflake** → Cloud Data Warehouse (Bronze → Silver → Gold).  
5. **DBT** → Applies transformations, builds marts & snapshots (SCD Type-2).  
6. **CI/CD with GitHub Actions** → Automated tests, build & deployment.  

---

##  Tech Stack
- **Snowflake** → Cloud Data Warehouse  
- **DBT** → Transformations, testing, snapshots (SCD Type-2)  
- **Apache Airflow** → Orchestration & DAG scheduling  
- **Apache Kafka + Debezium** → Real-time streaming & CDC  
- **MinIO** → S3-compatible object storage  
- **Postgres** → Source OLTP system  
- **Python (Faker)** → Data simulation  
- **Docker & docker-compose** → Containerized setup  
- **Git & GitHub Actions** → CI/CD workflows  

---

##  Key Features
- **PostgreSQL OLTP**: Source relational database with ACID guarantees (customers, accounts, transactions)  
- **Simulated banking system**: customers, accounts, and transactions  
- **Change Data Capture (CDC)** via Kafka + Debezium (capturing Postgres WAL)  
- **Raw → Staging → Fact/Dimension** models in DBT  
- **Snapshots for history tracking** (slowly changing dimensions)  
- **Automated pipeline orchestration** using Airflow  
- **CI/CD pipeline** with dbt tests + GitHub Actions  

---

##  Repository Structure
```text
banking-modern-datastack/
├── .github/workflows/         # CI/CD pipelines (ci.yml, cd.yml)
├── banking_dbt/              # DBT project
│   ├── models/
│   │   ├── staging/           # Staging models
│   │   ├── marts/             # Facts & dimensions
│   │   └── sources.yml
│   ├── snapshots/             # SCD2 snapshots
│   └── dbt_project.yml
├── consumer
│   └── kafka_to_minio.py
├── data-generator/            # Faker-based data simulator
│   └── faker_generator.py
├── docker/                    # Airflow DAGs, plugins, etc.
│   ├── dags/                  # DAGs (minio_to_snowflake, scd_snapshots)
├── kafka-debezium/            # Kafka connectors & CDC logic
│   └── generate_and_post_connector.py
├── postgres/                  # Postgres schema (OLTP DDL & seeds)
│   └── schema.sql
├── .gitignore
├── docker-compose.yml         # Containerized infra
├── dockerfile-airflow.dockerfile
├── requirements.txt
└── README.md
```

---

##  Step-by-Step Implementation  

1️⃣ Data Simulation – Banking OLTP System
Purpose

Simulate a real-world transactional banking system.

1.1 Synthetic Data Generation

Generated synthetic banking datasets using Faker

customers

accounts

transactions

Inserted into PostgreSQL (OLTP) system

Maintained relational integrity (foreign keys, constraints)

Configurable generation via config.yaml

1.2 Problems Addressed
Challenge	Solution
No real production data	Synthetic generation using Faker
Need realistic relationships	Foreign key constraints
Controlled testing volume	Config-driven generation
OLTP simulation	PostgreSQL transactional database

Postgres acts as a real banking transactional system.

2️⃣ Change Data Capture – Kafka + Debezium
Purpose

Capture real-time database changes without polling.

2.1 CDC Implementation

Configured Kafka Connect

Deployed Debezium Postgres Connector

Captured:

INSERT

UPDATE

DELETE

Published events to Kafka topics:

customers

accounts

transactions

2.2 Streaming to Data Lake

Python Kafka Consumer:

Extracted after payload

Buffered records (batch size = 50)

Converted to Parquet

Uploaded to MinIO (S3-compatible storage)

2.3 Problems Addressed
Challenge	Solution
Database polling inefficiency	Log-based CDC
High data latency	Event streaming
Small file problem	Batch buffering
Complex Debezium JSON	Payload extraction
Storage scalability	S3-compatible MinIO
Analytics inefficiency	Columnar Parquet format

This establishes the Raw Lake (Bronze equivalent) layer.

3️⃣ Airflow Orchestration
Purpose

Automate and manage pipeline execution.

3.1 DAG Responsibilities

Ingest MinIO Parquet → Snowflake (Bronze schema)

Trigger dbt transformations

Manage task dependencies

Schedule incremental loads

Retry failed tasks

3.2 Problems Addressed
Challenge	Solution
Manual execution	Scheduled DAGs
Dependency mismanagement	Directed Acyclic Graph
Silent pipeline failures	Retry & logging
Lack of visibility	Airflow monitoring UI

Airflow acts as the orchestration layer.

4️⃣ Snowflake Cloud Data Warehouse
Purpose

Store scalable, analytics-ready datasets.

4.1 Layered Architecture
Bronze

Raw ingestion from MinIO

Minimal transformation

Silver

Cleaned & standardized datasets

Deduplicated records using ROW_NUMBER()

Type casting & null handling

Gold

Fact & Dimension models

Analytics-ready marts

4.2 Problems Addressed
Challenge	Solution
Raw unstructured ingestion	Bronze staging schema
Duplicate change records	Window-based deduplication
Dirty attributes	Silver cleansing logic
Business analytics need	Gold dimensional modeling
Scalability limits	Elastic Snowflake warehouse

Snowflake provides elastic compute & storage separation.

5️⃣ DBT Transformation Framework
Purpose

Apply structured, version-controlled transformations.

5.1 Implementation

Source definitions for raw Snowflake tables

Staging models:

Type casting

Deduplication

Latest-record filtering

Dimension models:

dim_customers

dim_accounts

Fact models:

fact_transactions

Snapshots:

Track historical changes (accounts & customers)

5.2 Problems Addressed
Challenge	Solution
Manual SQL management	dbt version-controlled models
No historical tracking	Snapshots
Duplicate CDC records	ROW_NUMBER filtering
Schema drift	Explicit casting
Lack of lineage	dbt dependency graph

dbt enforces transformation governance.

6️⃣ CI/CD with GitHub Actions
Purpose

Ensure reliability and production-readiness.

6.1 CI Pipeline (ci.yml)

Trigger on push / pull request

Setup Python

Install dependencies

Linting (Ruff)

Unit tests (Pytest)

dbt compile validation

6.2 CD Pipeline (cd.yml)

Deploy Airflow DAGs

Deploy dbt models

Validate Snowflake connection

6.3 Problems Addressed
Challenge	Solution
Broken commits	Automated validation
SQL syntax errors	dbt compile
Deployment risk	Controlled CI/CD
Code quality issues	Linting enforcement

CI/CD ensures pipeline stability.

📦 Final Deliverables

-Automated CDC pipeline (Postgres → Kafka → MinIO → Snowflake)

-Structured Lakehouse architecture (Bronze → Silver → Gold)

-dbt transformation framework (staging, marts, snapshots)

-Airflow DAG orchestration

-Synthetic banking dataset for simulation

-Containerized infrastructure (Docker)

-CI/CD workflows for reliability

---

**Author**: *Jaya Chandra Kadiveti*  
**LinkedIn**: [jayachandrakadiveti](https://www.linkedin.com/in/jayachandrakadiveti/)  
**Contact**: [datawithjay1@gmail.com](mailto:datawithjay1@gmail.com)  


