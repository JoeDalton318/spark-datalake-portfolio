# Modern Data Lake Architecture with Spark, Kafka & MinIO

![Apache Spark](https://img.shields.io/badge/Apache%20Spark-4.0.1-E25A1C?style=for-the-badge&logo=apachespark&logoColor=white)
![Kafka](https://img.shields.io/badge/Apache%20Kafka-8.0-231F20?style=for-the-badge&logo=apachekafka&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-18.1-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![MinIO](https://img.shields.io/badge/MinIO-Latest-C72E49?style=for-the-badge&logo=minio&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Lab-F37626?style=for-the-badge&logo=jupyter&logoColor=white)

## Overview

Production-ready Data Lake infrastructure implementing the **Medallion Architecture** (Bronze/Silver/Gold) for modern data engineering workflows. This project demonstrates enterprise-level data processing patterns using industry-standard tools.

### Key Features

- **Medallion Architecture**: Bronze (Raw), Silver (Cleansed), Gold (Aggregated) layers
- **Real-time Streaming**: Apache Kafka integration with Spark Structured Streaming
- **Batch Processing**: PySpark for large-scale ETL/ELT operations
- **Object Storage**: MinIO S3-compatible storage for data lake persistence
- **Containerized**: Full Docker Compose orchestration for reproducible environments
- **Analytics-Ready**: Pre-loaded Northwind database for business intelligence scenarios

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    MODERN DATA LAKE ARCHITECTURE                            │
└─────────────────────────────────────────────────────────────────────────────┘

    ┌──────────────┐        ┌──────────────┐        ┌──────────────┐
    │  PostgreSQL  │        │ Kafka Broker │        │    MinIO     │
    │   (OLTP)     │        │  (Streaming) │        │ (Data Lake)  │
    │    :5433     │        │    :9092     │        │  :9000/:9001 │
    └──────┬───────┘        └──────┬───────┘        └──────┬───────┘
           │                       │                       │
           │    Ingestion          │   Real-time           │
           │                       │                       │
    ┌──────▼───────────────────────▼───────────────────────▼───────┐
    │                     SPARK PROCESSING LAYER                   │
    │                          (JupyterLab)                        │
    │  ┌────────────────────────────────────────────────────────┐  │
    │  │  Bronze Layer  →  Silver Layer  →  Gold Layer          │  │
    │  │  (Raw Data)      (Cleansed)        (Business KPIs)     │  │
    │  └────────────────────────────────────────────────────────┘  │
    └──────────────────────────────────────────────────────────────┘
```

### Data Flow

1. **Bronze Layer**: Raw data ingestion from PostgreSQL (JDBC) and Kafka streams
2. **Silver Layer**: Data cleansing, deduplication, and denormalization (star schema)
3. **Gold Layer**: Business aggregations, KPIs, and RFM analysis for analytics

## Quick Start

### Prerequisites

- Docker Desktop 20.10+
- Docker Compose 2.0+
- 8GB RAM minimum (16GB recommended)
- 20GB free disk space

### Launch Environment

```bash
# Clone repository
git clone https://github.com/your-username/spark-datalake-portfolio.git
cd spark-datalake-portfolio

# Start all services
docker compose up -d

# Verify services are running
docker compose ps

# Access JupyterLab
# Open http://localhost:8888 in your browser
```

### Run the Complete Pipeline

Open the notebook `notebooks/TP_Groupe3_DataLake.ipynb` in JupyterLab and execute all cells sequentially. The pipeline will:

1. Ingest Northwind database tables to Bronze layer (Parquet format)
2. Apply transformations and create master view in Silver layer
3. Produce real-time events to Kafka and consume with Spark Streaming
4. Generate business KPIs and customer segmentation in Gold layer
5. Display interactive dashboard with matplotlib/seaborn visualizations

## Services & Access Points

| Service | URL | Credentials | Purpose |
|---------|-----|-------------|---------|
| **JupyterLab** | http://localhost:8888 | No token required | Spark development environment |
| **MinIO Console** | http://localhost:9001 | `minioadmin` / `minioadmin123` | S3 storage management |
| **Kafka UI** | http://localhost:7080 | - | Stream monitoring & topic management |
| **Adminer** | http://localhost:9080 | `postgres` / `postgres` | Database client (DB: `app`) |
| **PostgreSQL** | localhost:5433 | `postgres` / `postgres` | Source database |
| **Kafka Broker** | localhost:9092 | - | Event streaming |
| **MinIO API** | localhost:9000 | - | S3-compatible object storage |

## Project Structure

```
spark-datalake-portfolio/
├── docker-compose.yml              # Infrastructure orchestration
├── notebooks/
│   ├── TP_Groupe3_DataLake.ipynb  # Complete pipeline implementation
│   ├── exemples/                   # Starter examples
│   └── exercices/                  # Practice exercises
├── vol/
│   ├── jupyter/                    # Jupyter configuration
│   └── postgresql/
│       └── northwind.sql           # Sample database schema
└── README.md                       # This file
```

## Technical Stack

### Core Technologies

- **Apache Spark 4.0.1**: Distributed data processing engine
- **PySpark**: Python API for Spark
- **Apache Kafka 8.0**: Real-time event streaming platform
- **MinIO**: High-performance S3-compatible object storage
- **PostgreSQL 18.1**: OLTP source database with Northwind dataset
- **JupyterLab**: Interactive development environment

### Python Libraries

- **kafka-python**: Kafka producer/consumer clients
- **pyspark**: Spark SQL, DataFrame, and Streaming APIs
- **matplotlib/seaborn**: Data visualization
- **pandas**: Data manipulation and analysis

### Data Processing Features

- JDBC connectivity (PostgreSQL)
- S3A file system integration (Hadoop AWS)
- Spark Structured Streaming
- Kafka integration (spark-sql-kafka)
- Parquet columnar storage format

## Use Cases Demonstrated

### 1. Batch Data Ingestion
- Full table extraction from OLTP database
- Metadata tracking (timestamp, source)
- Partitioned writes to object storage

### 2. Data Transformation & Quality
- Null handling and data type conversions
- Denormalization for analytical queries
- Calculated fields and business logic

### 3. Real-time Stream Processing
- Kafka producer simulation (Python)
- Spark Structured Streaming consumer
- Micro-batch aggregations
- In-memory query tables

### 4. Business Intelligence
- Customer segmentation (RFM analysis)
- Sales performance by region
- Product category analysis
- Real-time revenue monitoring

## Pipeline Implementation Details

The main notebook (`TP_Groupe3_DataLake.ipynb`) implements a complete data engineering workflow:

### Phase 1: Bronze Layer (Data Ingestion)
- Reads 6 tables from Northwind database via JDBC
- Adds technical metadata columns
- Writes raw Parquet files to MinIO (`s3a://bronze/`)

### Phase 2: Silver Layer (Data Transformation)
- Cleans and standardizes data types
- Calculates derived metrics (line totals, discounts)
- Creates denormalized master view with 5-table join
- Stores cleansed data (`s3a://silver/`)

### Phase 3: Streaming Integration
- Python Kafka producer generates synthetic sales events
- Spark Structured Streaming consumes from topic
- Real-time aggregations by country
- Writes to in-memory table for dashboard queries

### Phase 4: Gold Layer (Business Analytics)
- Computes total revenue KPI
- Top 3 countries by sales volume
- RFM customer segmentation (Recency, Frequency, Monetary)
- VIP classification logic
- Stores final analytics (`s3a://gold/`)

### Phase 5: Visualization Dashboard
- Matplotlib/Seaborn charts for insights
- Top 10 countries bar chart
- Customer segment pie chart
- Product category performance
- Real-time streaming comparison

## Configuration

### Spark Session Configuration

```python
spark = SparkSession.builder \
    .appName("Modern Data Lake Pipeline") \
    .config("spark.jars.packages", 
            "org.postgresql:postgresql:42.6.0,"
            "org.apache.hadoop:hadoop-aws:3.4.1,"
            "com.amazonaws:aws-java-sdk-bundle:1.12.262,"
            "org.apache.spark:spark-sql-kafka-0-10_2.13:3.5.0") \
    .config("spark.hadoop.fs.s3a.endpoint", "http://minio:9000") \
    .config("spark.hadoop.fs.s3a.access.key", "minioadmin") \
    .config("spark.hadoop.fs.s3a.secret.key", "minioadmin123") \
    .config("spark.hadoop.fs.s3a.path.style.access", "true") \
    .getOrCreate()
```

### MinIO Buckets

The pipeline automatically uses the following buckets:
- `bronze/` - Raw ingested data with technical metadata
- `silver/` - Cleansed and transformed dimensional data
- `gold/` - Aggregated business metrics and KPIs

Create buckets via MinIO Console (http://localhost:9001) or use S3 CLI.

## Stopping the Environment

```bash
# Stop services (keeps data)
docker compose down

# Stop and remove all data (CAUTION)
docker compose down -v
```

## Learning Outcomes

This project demonstrates competency in:

- Modern data lake architecture patterns
- Medallion (Bronze/Silver/Gold) data organization
- Distributed data processing with Apache Spark
- Real-time streaming with Kafka and Spark Structured Streaming
- S3-compatible object storage integration
- ETL/ELT pipeline development
- Data quality and transformation techniques
- Business intelligence and analytics
- Docker containerization for data platforms
- Python data engineering ecosystem

## Contributing

This project was developed as an academic assignment. Feel free to fork and adapt for your own learning purposes.

## Authors

- Gills Daryl KETCHA
- Narcisse Cabrel TSAFACK
- Frédéric FERNANDES DA COSTA
- Jennifer HOUNGBEDJI

## License

Educational use only. Not licensed for commercial applications.

## Acknowledgments

Built with best practices from industry-standard data engineering frameworks and inspired by real-world data lake architectures.
