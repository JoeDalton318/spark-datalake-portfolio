# Architecture Documentation

## System Overview

This project implements a modern Data Lake architecture based on the Medallion pattern, integrating batch and stream processing capabilities through Apache Spark and Kafka.

## Design Principles

### 1. Medallion Architecture (Bronze-Silver-Gold)

**Bronze Layer - Raw Data Zone**
- Purpose: Immutable landing zone for all source data
- Format: Parquet (columnar storage for efficient querying)
- Schema: Source schema + technical metadata (_ingestion_timestamp, _source)
- Location: `s3a://bronze/{table_name}/{date}`
- Quality: No transformations, exact copy of source data

**Silver Layer - Cleansed Data Zone**
- Purpose: High-quality, conformed data ready for analytics
- Transformations:
  - Data type standardization
  - Null handling and validation
  - Business rule application
  - Denormalization for performance
- Format: Parquet with optimized schema
- Location: `s3a://silver/{entity}/{date}`
- Quality: Validated, cleaned, and enriched

**Gold Layer - Business Aggregations**
- Purpose: Pre-computed metrics and KPIs
- Transformations:
  - Aggregations and rollups
  - Customer segmentation (RFM)
  - Performance indicators
- Format: Parquet optimized for BI tools
- Location: `s3a://gold/{metric}/{date}`
- Quality: Analytics-ready, business-defined

### 2. Lambda Architecture Components

**Batch Processing (Spark SQL)**
- Historical data analysis
- Complex transformations and joins
- Scheduled ETL workflows
- Full table scans and aggregations

**Stream Processing (Spark Structured Streaming)**
- Real-time event ingestion
- Micro-batch processing
- Stateful computations
- Low-latency dashboards

**Serving Layer**
- In-memory tables for real-time queries
- Parquet files for historical analysis
- Combined views for unified analytics

## Data Flow Diagram

```
┌────────────────────────────────────────────────────────────────────────┐
│                         DATA SOURCES                                   │
├─────────────────────────────┬──────────────────────────────────────────┤
│   BATCH (PostgreSQL)        │    STREAM (Kafka)                        │
│   - Northwind OLTP DB       │    - Real-time sales events              │
│   - 6 tables (normalized)   │    - JSON payloads                       │
└─────────────┬───────────────┴──────────────┬───────────────────────────┘
              │                               │
              │  JDBC Read                    │  Kafka Consumer
              │                               │
┌─────────────▼───────────────────────────────▼───────────────────────────┐
│                         BRONZE LAYER (RAW)                              │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  Parquet Files: orders, order_details, products, customers,   │     │
│  │                 employees, categories                          │     │
│  │  + Metadata: _ingestion_timestamp, _source                     │     │
│  └────────────────────────────────────────────────────────────────┘     │
└─────────────────────────────┬───────────────────────────────────────────┘
                              │
                              │  PySpark Transformations
                              │
┌─────────────────────────────▼───────────────────────────────────────────┐
│                       SILVER LAYER (CLEANSED)                           │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  Master Sales View (Denormalized)                              │     │
│  │  - 5-table join (orders → details → products → categories      │     │
│  │                   → customers)                                  │     │
│  │  - Calculated fields: line_total, standardized country         │     │
│  │  - Quality checks: null handling, type casting                 │     │
│  └────────────────────────────────────────────────────────────────┘     │
└─────────────────────────────┬───────────────────────────────────────────┘
                              │
                              │  Aggregations & Analytics
                              │
┌─────────────────────────────▼───────────────────────────────────────────┐
│                        GOLD LAYER (ANALYTICS)                           │
│  ┌─────────────────────┐  ┌─────────────────────┐  ┌─────────────────┐ │
│  │   KPI Dashboard     │  │   RFM Analysis      │  │  Streaming View │ │
│  │                     │  │                     │  │                 │ │
│  │ - Total Revenue     │  │ - Recency           │  │ - Live Sales    │ │
│  │ - Top 3 Countries   │  │ - Frequency         │  │ - By Country    │ │
│  │ - Category Sales    │  │ - Monetary          │  │ - Real-time     │ │
│  │                     │  │ - Segmentation      │  │                 │ │
│  └─────────────────────┘  └─────────────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

## Technology Stack Details

### Apache Spark 4.0.1
- **Spark SQL**: Declarative query engine for batch processing
- **Spark Structured Streaming**: Scalable stream processing with SQL APIs
- **DataFrame API**: High-level abstraction for distributed data manipulation
- **Catalyst Optimizer**: Query optimization for performance
- **Tungsten Engine**: Memory and CPU efficiency improvements

### Apache Kafka 8.0 (Confluent Platform)
- **Topic**: `ventes-temps-reel` (real-time sales)
- **Partitions**: 1 (single-node setup)
- **Replication**: 1 (development mode)
- **Serialization**: JSON format for flexibility
- **Consumer Group**: Spark Structured Streaming

### MinIO (S3-Compatible Storage)
- **API**: AWS S3 compatible (s3a:// protocol)
- **Buckets**: bronze, silver, gold (logical separation)
- **Format**: Parquet (columnar, compressed)
- **Access**: Path-style addressing for Docker networking
- **Consistency**: Strong consistency for data integrity

### PostgreSQL 18.1 (Northwind Database)
- **Schema**: Normalized OLTP design (3NF)
- **Tables**: 6 core tables for e-commerce scenario
- **Access**: JDBC driver for Spark connectivity
- **Port**: 5433 (external), 5432 (internal Docker)

### Docker Compose Orchestration
- **Networks**: Custom bridge network for service discovery
- **Volumes**: Named volumes for data persistence
- **Health Checks**: Service dependency management
- **Resource Limits**: Configured for local development

## Data Engineering Patterns

### 1. Incremental Loading
```python
# Date partitioning for incremental updates
chemin_sortie = f"s3a://bronze/{table_name}/{DATE_JOUR}"
df_meta.write.mode("overwrite").parquet(chemin_sortie)
```

### 2. Idempotent Writes
- Overwrite mode ensures reruns produce consistent results
- Date partitioning allows historical reprocessing

### 3. Schema Evolution
- Parquet format supports schema changes
- Technical metadata fields for tracking

### 4. Denormalization Strategy
```python
# Silver layer creates wide table for analytics
df_silver_master = df_details \
    .join(df_orders, on="order_id") \
    .join(df_products, on="product_id") \
    .join(df_categories, on="category_id") \
    .join(df_customers, on="customer_id")
```

### 5. Real-time Aggregation
```python
# Micro-batch processing with watermarking
df_stream_agg = df_stream_parsed.groupBy("pays").agg(
    F.sum("montant").alias("total_ventes_live"),
    F.count("*").alias("nb_transactions")
)
```

## Performance Optimizations

### Storage Layer
- **Parquet Compression**: Snappy codec (default) for balance of speed/size
- **Columnar Format**: Efficient predicate pushdown for analytics queries
- **Partitioning**: Date-based partitions for efficient data pruning

### Processing Layer
- **Broadcast Joins**: Small dimension tables broadcasted to executors
- **Predicate Pushdown**: Filter operations pushed to storage layer
- **Column Pruning**: Only required columns read from Parquet

### Streaming Layer
- **Trigger**: `availableNow=True` for micro-batch processing
- **Output Mode**: `complete` for full aggregation updates
- **Memory Sink**: Fast in-memory table for dashboard queries

## Scalability Considerations

### Horizontal Scaling
- Add Spark workers for increased parallelism
- Kafka partitions for parallel consumption
- MinIO distributed mode for larger datasets

### Vertical Scaling
- Increase executor memory for larger aggregations
- Tune driver memory for complex transformations
- Adjust shuffle partitions for optimal performance

## Security Best Practices (Production)

1. **Credential Management**
   - Use environment variables for secrets
   - Implement AWS IAM roles for S3 access
   - Kafka SASL authentication

2. **Network Segmentation**
   - Separate VLANs for data tiers
   - Firewall rules for service isolation
   - TLS/SSL for data in transit

3. **Data Governance**
   - Column-level encryption for PII
   - Audit logging for access tracking
   - Data masking in non-production

## Monitoring & Observability

### Spark UI (Port 4040)
- Job execution metrics
- Stage-level performance
- Executor utilization

### Kafka UI (Port 7080)
- Topic lag monitoring
- Message throughput
- Consumer group health

### MinIO Console (Port 9001)
- Bucket statistics
- Object lifecycle
- Access patterns

## Future Enhancements

1. **Data Quality Framework**
   - Great Expectations integration
   - Automated validation rules
   - Data profiling reports

2. **Orchestration**
   - Apache Airflow for workflow scheduling
   - DAG-based pipeline dependencies
   - Retry and alerting mechanisms

3. **Advanced Analytics**
   - MLlib for machine learning models
   - Predictive customer churn
   - Recommendation engines

4. **Data Catalog**
   - Apache Atlas for metadata management
   - Data lineage tracking
   - Business glossary integration

## References

- [Medallion Architecture (Databricks)](https://www.databricks.com/glossary/medallion-architecture)
- [Spark Structured Streaming Guide](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)
- [S3A Hadoop Documentation](https://hadoop.apache.org/docs/stable/hadoop-aws/tools/hadoop-aws/index.html)
- [Kafka Documentation](https://kafka.apache.org/documentation/)
