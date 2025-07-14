# Data Schemas

This directory contains the schema definitions for all data types used in the telecom fraud detection system.

## Schema Files

- `cdr.avsc` - Avro schema for Call Detail Records (CDRs)
- `signaling.avsc` - Avro schema for signaling data
- `network_logs.avsc` - Avro schema for network logs
- `customer_events.avsc` - Avro schema for customer events
- `alerts.avsc` - Avro schema for fraud alerts

## CDR Schema Example

```json
{
  "type": "record",
  "name": "CDR",
  "namespace": "com.telecom.fraud.detection",
  "fields": [
    {"name": "call_id", "type": "string"},
    {"name": "timestamp", "type": "long"},
    {"name": "caller_number", "type": "string"},
    {"name": "called_number", "type": "string"},
    {"name": "call_duration", "type": "int"},
    {"name": "call_type", "type": "string"},
    {"name": "call_direction", "type": "string"},
    {"name": "switch_id", "type": "string"},
    {"name": "trunk_id", "type": "string"},
    {"name": "billing_type", "type": "string"},
    {"name": "call_status", "type": "string"},
    {"name": "disconnect_reason", "type": "string"},
    {"name": "source_country", "type": "string"},
    {"name": "destination_country", "type": "string"},
    {"name": "roaming_status", "type": "boolean", "default": false},
    {"name": "service_type", "type": "string"}
  ]
}
```

## Signaling Schema Example

```json
{
  "type": "record",
  "name": "SignalingEvent",
  "namespace": "com.telecom.fraud.detection",
  "fields": [
    {"name": "event_id", "type": "string"},
    {"name": "timestamp", "type": "long"},
    {"name": "protocol", "type": "string"},
    {"name": "message_type", "type": "string"},
    {"name": "source_address", "type": "string"},
    {"name": "destination_address", "type": "string"},
    {"name": "calling_party", "type": "string"},
    {"name": "called_party", "type": "string"},
    {"name": "network_id", "type": "string"},
    {"name": "gateway_id", "type": "string"},
    {"name": "status_code", "type": "int"},
    {"name": "error_code", "type": ["null", "int"], "default": null},
    {"name": "error_message", "type": ["null", "string"], "default": null},
    {"name": "call_id", "type": ["null", "string"], "default": null},
    {"name": "session_id", "type": ["null", "string"], "default": null},
    {"name": "payload", "type": ["null", "bytes"], "default": null}
  ]
}
```

## Schema Registry

All schemas are registered in the Confluent Schema Registry to ensure compatibility and versioning. The schema registry is available at:

- Development: `http://localhost:8081`
- Testing: `http://schema-registry.test.telecom-fraud.internal:8081`
- Production: `http://schema-registry.prod.telecom-fraud.internal:8081`

## Schema Evolution

When evolving schemas, follow these guidelines:
1. Only add optional fields (with defaults)
2. Never remove fields
3. Never change field types
4. Register new schema versions in the schema registry before deploying consumers/producers