# Data Ingestion and Processing

This chart details the flow of data from various sources through the ingestion and processing pipeline to the storage layer.

```mermaid
flowchart LR
    subgraph "Real-time Ingestion"
        CDR[CDRs] --> KAFKA[Kafka/Pulsar]
        SIG[Signaling] --> KAFKA
        NET[Network Logs] --> KAFKA
    end

    subgraph "Batch Ingestion"
        CUST[Customer Profiles] --> HDFS[HDFS/S3]
        HIST[Historical Fraud] --> HDFS
    end

    subgraph "Processing"
        KAFKA --> FLINK[Flink/Spark Streaming]
        HDFS --> SPARK[Spark Batch]
        FLINK & SPARK --> CLEAN[Data Cleaning]
        CLEAN --> NORM[Normalization]
        NORM --> ENRICH[Enrichment]
        ENRICH --> FEAT[Feature Extraction]
    end

    subgraph "Storage"
        FEAT --> TS[Time-Series DB]
        FEAT --> GRAPH_DB[Graph Database]
        FEAT --> FEATURE_STORE[Feature Store]
    end
```

## Description

This diagram illustrates the detailed data flow from sources to storage:

1. **Real-time Ingestion**: 
   - Call Detail Records (CDRs), signaling data, and network logs are ingested in real-time through Kafka/Pulsar
   - These data sources require immediate processing for fraud detection

2. **Batch Ingestion**:
   - Customer profiles and historical fraud data are processed in batches
   - These data sources are stored in HDFS or S3-compatible storage for periodic processing

3. **Processing Pipeline**:
   - Real-time data is processed through Flink/Spark Streaming
   - Batch data is processed through Spark Batch jobs
   - All data goes through cleaning, normalization, and enrichment stages
   - Feature extraction transforms raw data into ML-ready features

4. **Storage Layer**:
   - Time-Series DB (TimescaleDB) stores temporal data for trend analysis
   - Graph Database (Neo4j) stores relationship data for network analysis
   - Feature Store (Feast) maintains features for ML model training and inference

This architecture enables both real-time fraud detection and deeper batch analysis, combining the strengths of both approaches.