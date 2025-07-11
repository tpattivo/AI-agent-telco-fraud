# System Architecture Overview

This chart illustrates the high-level architecture of the telecom fraud detection system, showing the main components and data flow.

```mermaid
flowchart TD
    subgraph "Data Sources"
        CDR[Call Detail Records]
        SIG[Signaling Data]
        NET[Network Logs]
        CUST[Customer Profiles]
        HIST[Historical Fraud Cases]
    end

    subgraph "Data Ingestion Layer"
        KAFKA[Kafka/Pulsar Streaming]
        BATCH[Batch Processing]
    end

    subgraph "Data Processing Layer"
        SPARK[Spark/Flink Processing]
        FEAT[Feature Engineering]
        STORE[Feature Store]
    end

    subgraph "AI/ML Layer"
        ANOM[Anomaly Detection]
        PRED[Predictive Models]
        GRAPH[Graph Analysis]
        DL[Deep Learning]
        RL[Reinforcement Learning]
    end

    subgraph "Decision Layer"
        RULES[Rule Engine]
        SCORE[Risk Scoring]
        ALERT[Alert Generation]
    end

    subgraph "Action Layer"
        BLOCK[Call/Transaction Blocking]
        FLAG[Account Flagging]
        NOTIFY[Notification System]
        CASE[Case Management]
    end

    subgraph "Monitoring & Feedback"
        DASH[Dashboards]
        METRICS[Performance Metrics]
        FEEDBACK[Feedback Loop]
    end

    CDR & SIG & NET & CUST & HIST --> KAFKA & BATCH
    KAFKA & BATCH --> SPARK
    SPARK --> FEAT
    FEAT --> STORE
    STORE --> ANOM & PRED & GRAPH & DL & RL
    ANOM & PRED & GRAPH & DL & RL --> RULES & SCORE
    RULES & SCORE --> ALERT
    ALERT --> BLOCK & FLAG & NOTIFY & CASE
    BLOCK & FLAG & NOTIFY & CASE --> DASH & METRICS
    DASH & METRICS --> FEEDBACK
    FEEDBACK --> FEAT
```

## Description

This diagram shows the complete data flow through the system:

1. **Data Sources**: Various telecom data sources feed into the system
2. **Data Ingestion Layer**: Handles real-time and batch data ingestion
3. **Data Processing Layer**: Processes and transforms raw data into features
4. **AI/ML Layer**: Various ML techniques analyze the data for fraud patterns
5. **Decision Layer**: Combines ML outputs with rules to make decisions
6. **Action Layer**: Takes appropriate actions based on decisions
7. **Monitoring & Feedback**: Monitors system performance and provides feedback

The arrows indicate data flow between components, showing how information moves through the system and how the feedback loop continuously improves the system's performance.