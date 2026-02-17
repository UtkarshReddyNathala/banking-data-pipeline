# Real-Time Banking Data Pipeline 
### Change Data Capture | Streaming Architecture | Lakehouse Design | Orchestration | CI/CD

---

##  Project Overview
This project is an **end-to-end modern real time data stack pipeline** for a **Banking domain**.  
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

# Step-by-Step Implementation

---

# 1️. Data Simulation – Banking OLTP System

## Purpose  

Simulate a real-world transactional banking system.

---

## 1.1 Synthetic Data Generation

* Generated synthetic banking datasets using **Faker**
  * customers
  * accounts
  * transactions
* Inserted into **PostgreSQL (OLTP)** system
* Maintained relational integrity (foreign keys, constraints)
* Configurable generation via `config.yaml`

---

## 1.2 Problems Addressed

| Challenge | Solution |
|------------|----------|
| No real production data | Synthetic generation using Faker |
| Need realistic relationships | Foreign key constraints |
| Controlled testing volume | Config-driven generation |
| OLTP simulation | PostgreSQL transactional database |

Postgres acts as a real banking transactional system.

---

# 2️. Change Data Capture – Kafka + Debezium

## Purpose  

Capture real-time database changes without polling.

---

## 2.1 CDC Implementation

* Configured **Kafka Connect**
* Deployed **Debezium Postgres Connector**
* Captured:
  * INSERT
  * UPDATE
  * DELETE
* Published events to Kafka topics:
  * customers
  * accounts
  * transactions

---

## 2.2 Streaming to Data Lake

* Python Kafka Consumer:
  * Extracted `after` payload
  * Buffered records (batch size = 50)
  * Converted to Parquet
  * Uploaded to MinIO (S3-compatible storage)

---

## 2.3 Problems Addressed

| Challenge | Solution |
|------------|----------|
| Database polling inefficiency | Log-based CDC |
| High data latency | Event streaming |
| Small file problem | Batch buffering |
| Complex Debezium JSON | Payload extraction |
| Storage scalability | S3-compatible MinIO |
| Analytics inefficiency | Columnar Parquet format |

This establishes the **Raw Lake (Bronze equivalent)** layer.

---

# 3️.Airflow Orchestration

## Purpose  

Automate and manage pipeline execution.

---

## 3.1 DAG Responsibilities

* Ingest MinIO Parquet → Snowflake (Bronze schema)
* Trigger dbt transformations
* Manage task dependencies
* Schedule incremental loads
* Retry failed tasks

---

## 3.2 Problems Addressed

| Challenge | Solution |
|------------|----------|
| Manual execution | Scheduled DAGs |
| Dependency mismanagement | Directed Acyclic Graph |
| Silent pipeline failures | Retry & logging |
| Lack of visibility | Airflow monitoring UI |

Airflow acts as the orchestration layer.

---

# 4️.Snowflake Cloud Data Warehouse

## Purpose  

Store scalable, analytics-ready datasets.

---

## 4.1 Layered Architecture

### Bronze
* Raw ingestion from MinIO
* Minimal transformation

### Silver
* Cleaned & standardized datasets
* Deduplicated records using `ROW_NUMBER()`
* Type casting & null handling

### Gold
* Fact & Dimension models
* Analytics-ready marts

---

## 4.2 Problems Addressed

| Challenge | Solution |
|------------|----------|
| Raw unstructured ingestion | Bronze staging schema |
| Duplicate change records | Window-based deduplication |
| Dirty attributes | Silver cleansing logic |
| Business analytics need | Gold dimensional modeling |
| Scalability limits | Elastic Snowflake warehouse |

Snowflake provides elastic compute & storage separation.

---

# 5️.DBT Transformation Framework

## Purpose  

Apply structured, version-controlled transformations.

---

## 5.1 Implementation

* Source definitions for raw Snowflake tables
* Staging models:
  * Type casting
  * Deduplication
  * Latest-record filtering
* Dimension models:
  * dim_customers
  * dim_accounts
* Fact models:
  * fact_transactions
* Snapshots:
  * Track historical changes (accounts & customers)

---

## 5.2 Problems Addressed

| Challenge | Solution |
|------------|----------|
| Manual SQL management | dbt version-controlled models |
| No historical tracking | Snapshots |
| Duplicate CDC records | ROW_NUMBER filtering |
| Schema drift | Explicit casting |
| Lack of lineage | dbt dependency graph |

dbt enforces transformation governance.

---

# 6️.CI/CD with GitHub Actions

## Purpose  

Ensure reliability and production-readiness.

---

## 6.1 CI Pipeline (`ci.yml`)

* Trigger on push / pull request
* Setup Python
* Install dependencies
* Linting (Ruff)
* Unit tests (Pytest)
* dbt compile validation

---

## 6.2 CD Pipeline (`cd.yml`)

* Deploy Airflow DAGs
* Deploy dbt models
* Validate Snowflake connection

---

## 6.3 Problems Addressed

| Challenge | Solution |
|------------|----------|
| Broken commits | Automated validation |
| SQL syntax errors | dbt compile |
| Deployment risk | Controlled CI/CD |
| Code quality issues | Linting enforcement |

CI/CD ensures pipeline stability.

---

# Final Deliverables

* Automated CDC pipeline (Postgres → Kafka → MinIO → Snowflake)
* Structured Lakehouse architecture (Bronze → Silver → Gold)
* dbt transformation framework (staging, marts, snapshots)
* Airflow DAG orchestration
* Synthetic banking dataset for simulation
* Containerized infrastructure (Docker)
* CI/CD workflows for reliability

---

### Analytics Powerbi Dashbaord

![Powerbi Dashboard](banking-data-pipeline/postgres/Powerbi_Dashboard.png)


---

# Banking Business Insights Report

## Executive Summary

This dashboard provides a comprehensive overview of the bank’s customer base, account activity, and transaction volumes. It offers actionable insights for business managers and decision-makers to understand customer behavior, monitor account activity, and optimize banking operations. The dashboard aggregates data across **198 customers**, **400 accounts**, and **1050 transactions**, highlighting trends in customer and account-level engagement.

---

## Key Metrics

| Metric                   | Value   | Insight                                                  |
| ------------------------ | ------- | -------------------------------------------------------- |
| Total Customers          | 198     | The bank currently serves 198 unique customers.          |
| Total Accounts           | 400     | Each customer holds, on average, 2 accounts.             |
| Total Transactions       | 1050    | Indicates active customer engagement across accounts.    |
| Average Balance          | $515.24 | Provides an overview of typical customer deposit levels. |
| Total Transaction Amount | $541.9K | Represents total monetary movement across all accounts.  |

---

## Analysis

### 1. Transaction Distribution by Type

* **Deposit, Withdrawal, Transfer** amounts are fairly evenly distributed:

  * Highest: $186.48K (34.41%)
  * Medium: $178.45K (32.93%)
  * Lowest: $176.97K (32.66%)
* Insight: The bank maintains a balanced transactional mix, reducing dependency on a single transaction type.

### 2. High-Value Customers

* Top 10 customers contribute significantly to total transaction value, ranging from **$4K to $6K** per customer.
* Insight: Identifying these customers allows the bank to target premium services, loyalty programs, and personalized offers.

### 3. High-Activity Accounts

* Certain accounts, such as Account_ID 143, 209, and 324, dominate transaction volumes.
* Insight: These accounts may belong to business clients or high-net-worth individuals, signaling the need for dedicated account management.

---

## Business Insights

1. **Customer Segmentation:** High-value customers can be targeted with personalized banking products to increase retention and cross-selling opportunities.
2. **Resource Allocation:** Bank operations can prioritize support for high-transaction accounts, improving service efficiency and customer satisfaction.
3. **Risk Management:** Balanced transaction types reduce exposure to liquidity risks from a single transaction type.
4. **Strategic Marketing:** Average balance and transaction volumes provide baseline metrics for marketing campaigns, promotions, and savings plans.
5. **Growth Opportunities:** Customers with fewer transactions may benefit from engagement programs to encourage higher activity and adoption of new services.

---

## Conclusion

The dashboard highlights that the bank has a **diverse and active customer base** with well-distributed transaction types and notable high-value clients. Leveraging this data can drive **customer retention, operational efficiency, and targeted growth strategies**. Continuous monitoring of transaction patterns and account activity will enable proactive decision-making for business success.

---














