# Feature Extractors

This directory contains feature extraction components for different fraud types. These extractors transform raw telecom data into features that can be used by machine learning models.

## Feature Extractors

- `irsf_features.py` - Features for International Revenue Share Fraud detection
- `wangiri_features.py` - Features for Wangiri fraud detection
- `bypass_features.py` - Features for Interconnect Bypass fraud detection
- `account_takeover_features.py` - Features for Account Takeover detection

## IRSF Feature Extractor

The IRSF feature extractor generates features related to international calling patterns and high-risk destinations.

### Key Features

- **Destination Risk Score**: Risk score for the destination country/number
- **Call Duration Features**: Statistics on call duration
- **Time-based Features**: Time of day, day of week patterns
- **Volume Features**: Call frequency, total duration
- **Destination Diversity**: Variety of international destinations called
- **Premium Number Ratio**: Ratio of calls to premium rate numbers
- **Historical Comparison**: Deviation from historical patterns

### Example Usage

```python
from fraud_detection.features.extractors import IRSFFeatureExtractor

# Initialize extractor with configuration
extractor = IRSFFeatureExtractor(config_path="config/features/irsf_features.yaml")

# Extract features from a CDR
features = extractor.extract_features(cdr)

# Extract features from a batch of CDRs
batch_features = extractor.extract_batch_features(cdr_batch)

# Extract features with customer history context
features_with_context = extractor.extract_features(
    cdr, 
    customer_history=customer_history
)
```

## Wangiri Feature Extractor

The Wangiri feature extractor focuses on detecting "one ring and cut" patterns designed to entice callbacks.

### Key Features

- **Call Duration**: Very short call durations
- **Callback Patterns**: History of callbacks to the number
- **Time Patterns**: Calls during unusual hours
- **Frequency**: Multiple short calls from the same source
- **International Origin**: Calls from high-risk international locations
- **Number Intelligence**: Information about the calling number

## Bypass Feature Extractor

The Bypass feature extractor identifies patterns related to interconnect bypass fraud.

### Key Features

- **Call Quality Metrics**: MOS scores, jitter, packet loss
- **Routing Anomalies**: Unusual routing paths
- **SIM Box Indicators**: Multiple calls from sequential IMEIs/IMSIs
- **Traffic Patterns**: Unusual traffic distribution
- **Signaling Anomalies**: Inconsistencies in signaling data

## Account Takeover Feature Extractor

The Account Takeover feature extractor identifies suspicious account activities.

### Key Features

- **Behavioral Biometrics**: Typing patterns, navigation behavior
- **Device Intelligence**: Device fingerprinting, new devices
- **Location Anomalies**: Unusual access locations
- **Activity Timing**: Unusual times of activity
- **Transaction Patterns**: Changes in transaction behavior
- **Setting Changes**: Password changes, notification settings

## Feature Store Integration

All extractors integrate with the Feast feature store for:
- Feature versioning
- Point-in-time correctness
- Online/offline feature serving
- Feature monitoring