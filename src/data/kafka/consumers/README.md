# Kafka Consumers

This directory contains Kafka consumer implementations for ingesting various data types from telecom systems.

## Consumers

- `cdr_consumer.py` - Consumer for Call Detail Records (CDRs)
- `signaling_consumer.py` - Consumer for signaling data (SS7, Diameter, SIP)
- `network_logs_consumer.py` - Consumer for network logs
- `customer_events_consumer.py` - Consumer for customer events

## Usage

Each consumer follows a similar pattern:

```python
from fraud_detection.data.kafka.consumers import CDRConsumer

# Initialize consumer with configuration
consumer = CDRConsumer(config_path="config/app/dev.yaml")

# Start consuming
consumer.start()
```

## Configuration

Consumers are configured via YAML files in the `config/app/` directory. Example configuration:

```yaml
kafka:
  bootstrap_servers: "localhost:9092"
  group_id: "fraud-detection-cdr-consumer"
  auto_offset_reset: "earliest"
  enable_auto_commit: true
  schema_registry_url: "http://localhost:8081"
  
topics:
  cdr: "telecom-fraud-detection.raw-data.cdrs"
  signaling: "telecom-fraud-detection.raw-data.signaling"
  network_logs: "telecom-fraud-detection.raw-data.network-logs"
  customer_events: "telecom-fraud-detection.raw-data.customer-events"
```

## Error Handling

All consumers implement robust error handling with:
- Automatic retries for transient errors
- Dead-letter queues for messages that cannot be processed
- Comprehensive logging
- Monitoring integration