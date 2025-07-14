# Configuration

This directory contains configuration files for all components of the telecom fraud detection system. The configuration is organized by component and environment.

## Directory Structure

```
config/
├── app/                  # Application configuration
│   ├── dev.yaml          # Development environment
│   ├── test.yaml         # Testing environment
│   └── prod.yaml         # Production environment
├── models/               # Model configuration
│   ├── irsf_model.yaml   # IRSF model configuration
│   ├── wangiri_model.yaml # Wangiri model configuration
│   ├── bypass_model.yaml # Bypass model configuration
│   └── account_takeover_model.yaml # Account takeover model configuration
├── features/             # Feature configuration
│   └── feature_definitions.yaml # Feature definitions
└── rules/                # Rules configuration
    └── rule_definitions.yaml # Rule definitions
```

## Application Configuration

The application configuration (`app/*.yaml`) contains settings for:
- Kafka connection
- Database connections
- API settings
- Logging configuration
- Monitoring settings
- Service discovery

Example application configuration:

```yaml
# Development environment configuration
environment: development

# Kafka configuration
kafka:
  bootstrap_servers: "localhost:9092"
  schema_registry_url: "http://localhost:8081"
  consumer_group_id: "fraud-detection-dev"
  topics:
    cdr: "telecom-fraud-detection.raw-data.cdrs"
    signaling: "telecom-fraud-detection.raw-data.signaling"
    network_logs: "telecom-fraud-detection.raw-data.network-logs"
    customer_events: "telecom-fraud-detection.raw-data.customer-events"
    alerts: "telecom-fraud-detection.actions.alerts"
    blocks: "telecom-fraud-detection.actions.blocks"

# Database configuration
databases:
  timescale:
    host: "localhost"
    port: 5432
    database: "fraud_detection"
    username: "fraud_user"
    password: "${TIMESCALE_PASSWORD}"  # Environment variable
    pool_size: 10
    max_overflow: 20
  
  redis:
    host: "localhost"
    port: 6379
    db: 0
    password: "${REDIS_PASSWORD}"  # Environment variable
    ssl: false
  
  neo4j:
    uri: "bolt://localhost:7687"
    username: "neo4j"
    password: "${NEO4J_PASSWORD}"  # Environment variable

# API configuration
api:
  host: "0.0.0.0"
  port: 8080
  debug: true
  cors_origins: ["http://localhost:3000"]
  rate_limit: 100  # Requests per minute
  timeout: 30  # Seconds

# Logging configuration
logging:
  level: "DEBUG"
  format: "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
  file: "/var/log/fraud_detection/app.log"
  syslog: false
  json: false

# Monitoring configuration
monitoring:
  prometheus:
    enabled: true
    port: 9090
  health_check:
    enabled: true
    path: "/health"
  tracing:
    enabled: true
    exporter: "jaeger"
    jaeger_endpoint: "http://localhost:14268/api/traces"

# Feature store configuration
feature_store:
  online_store: "redis"
  offline_store: "timescale"
  registry: "http://localhost:8080/fs"

# Model serving configuration
model_serving:
  registry: "http://localhost:5000"
  serving_type: "seldon"
  namespace: "fraud-detection"
```

## Model Configuration

The model configuration (`models/*.yaml`) contains settings for:
- Model parameters
- Training configuration
- Evaluation metrics
- Deployment settings

Example model configuration:

```yaml
# IRSF model configuration
model:
  name: "irsf_detection"
  version: "1.0.0"
  type: "xgboost"
  description: "XGBoost model for IRSF detection"

# Model parameters
parameters:
  objective: "binary:logistic"
  eval_metric: "auc"
  eta: 0.1
  max_depth: 6
  min_child_weight: 1
  subsample: 0.8
  colsample_bytree: 0.8
  scale_pos_weight: 50
  tree_method: "hist"

# Training configuration
training:
  train_test_split: 0.2
  random_state: 42
  num_boost_round: 1000
  early_stopping_rounds: 50
  cross_validation: 5

# Features
features:
  - call_duration
  - destination_risk_score
  - time_of_day
  - day_of_week
  - call_frequency_1h
  - call_frequency_24h
  - avg_duration_7d
  - international_call_ratio
  - premium_number_ratio
  - destination_country_code
  - destination_network_type
  - customer_tenure
  - customer_segment
  - billing_type
  - historical_spend
  - deviation_from_average_spend

# Thresholds
thresholds:
  block: 0.9
  alert: 0.7
  monitor: 0.5

# Evaluation metrics
evaluation:
  primary_metric: "precision_recall_auc"
  metrics:
    - "roc_auc"
    - "precision"
    - "recall"
    - "f1"
    - "average_precision"

# Deployment
deployment:
  resources:
    requests:
      cpu: "100m"
      memory: "512Mi"
    limits:
      cpu: "500m"
      memory: "1Gi"
  replicas: 2
  autoscaling:
    enabled: true
    min_replicas: 2
    max_replicas: 10
    target_cpu_utilization: 70
```

## Feature Configuration

The feature configuration (`features/feature_definitions.yaml`) contains:
- Feature definitions
- Transformation logic
- Feature groups
- Online/offline store settings

Example feature configuration:

```yaml
# Feature definitions
features:
  - name: call_duration
    type: int
    description: "Duration of the call in seconds"
    transformation: "identity"
    
  - name: destination_risk_score
    type: float
    description: "Risk score for the destination country/number"
    transformation: "lookup_from_reference_data"
    reference_data: "country_risk_scores"
    
  - name: time_of_day
    type: int
    description: "Hour of the day (0-23)"
    transformation: "extract_hour_from_timestamp"
    
  - name: day_of_week
    type: int
    description: "Day of the week (0-6, 0=Monday)"
    transformation: "extract_day_from_timestamp"
    
  - name: call_frequency_1h
    type: int
    description: "Number of calls in the last hour"
    transformation: "count_events_in_window"
    window: "1h"
    entity_key: "caller_number"
    
  - name: international_call_ratio
    type: float
    description: "Ratio of international calls to total calls"
    transformation: "ratio_calculation"
    numerator: "count_international_calls_7d"
    denominator: "count_total_calls_7d"

# Feature groups
feature_groups:
  - name: "irsf_features"
    description: "Features for IRSF detection"
    features:
      - call_duration
      - destination_risk_score
      - time_of_day
      - day_of_week
      - call_frequency_1h
      - call_frequency_24h
      - avg_duration_7d
      - international_call_ratio
      - premium_number_ratio
      - destination_country_code
      - destination_network_type
      - customer_tenure
      - customer_segment
      - billing_type
      - historical_spend
      - deviation_from_average_spend

# Feature store settings
feature_store:
  online_ttl: 86400  # 24 hours in seconds
  offline_retention: "90d"  # 90 days
  batch_size: 1000
  update_frequency: "5m"  # 5 minutes
```

## Rules Configuration

The rules configuration (`rules/rule_definitions.yaml`) contains:
- Business rule definitions
- Rule parameters
- Rule groups
- Rule priorities

Example rules configuration:

```yaml
# Rule definitions
rules:
  - id: "IRSF_HIGH_RISK_COUNTRY"
    name: "High Risk Country Call"
    description: "Call to a country with high IRSF risk"
    condition: "destination_country in high_risk_countries"
    confidence: 0.8
    enabled: true
    fraud_type: "IRSF"
    priority: 1
    
  - id: "WANGIRI_SHORT_CALL"
    name: "Very Short Call"
    description: "Call duration less than 5 seconds"
    condition: "call_duration < 5"
    confidence: 0.6
    enabled: true
    fraud_type: "Wangiri"
    priority: 2
    
  - id: "BYPASS_QUALITY_ISSUE"
    name: "Low Call Quality"
    description: "Call with low MOS score indicating potential bypass"
    condition: "mos_score < 2.5"
    confidence: 0.7
    enabled: true
    fraud_type: "Bypass"
    priority: 3
    
  - id: "ACCOUNT_UNUSUAL_LOCATION"
    name: "Unusual Access Location"
    description: "Account accessed from unusual location"
    condition: "location_risk_score > 0.8"
    confidence: 0.75
    enabled: true
    fraud_type: "Account Takeover"
    priority: 1

# Reference data
reference_data:
  high_risk_countries: ["XYZ", "ABC", "DEF"]
  premium_number_prefixes: ["+123", "+456", "+789"]
  
# Rule groups
rule_groups:
  - name: "IRSF Rules"
    description: "Rules for detecting IRSF fraud"
    rules:
      - "IRSF_HIGH_RISK_COUNTRY"
      - "IRSF_PREMIUM_NUMBER"
      - "IRSF_UNUSUAL_PATTERN"
  
  - name: "Wangiri Rules"
    description: "Rules for detecting Wangiri fraud"
    rules:
      - "WANGIRI_SHORT_CALL"
      - "WANGIRI_CALLBACK_PATTERN"
```

## Configuration Management

Configuration is managed using:
- Environment-specific files
- Environment variable substitution
- Secrets management
- Version control
- Configuration validation

Configuration is loaded at startup and can be reloaded without service restart for certain components.