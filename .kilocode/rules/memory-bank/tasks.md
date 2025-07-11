# Telecom Fraud Detection AI Agent - Tasks

This file documents repetitive tasks that follow similar patterns and require editing the same files. These documented workflows help maintain consistency and efficiency when performing common operations.

## Add New Fraud Detection Model

**Files to modify:**
- `/src/models/[fraud_type]_model.py` - Create new model implementation
- `/src/features/[fraud_type]_features.py` - Create feature extractor
- `/config/model_config.yaml` - Add model configuration
- `/tests/unit/test_models.py` - Add unit tests for the model

**Steps:**
1. Create feature extractor class that inherits from BaseFeatureExtractor
2. Implement feature extraction logic specific to the fraud type
3. Create model trainer class that inherits from ModelTrainer
4. Implement model training and evaluation logic
5. Add configuration parameters to model_config.yaml
6. Create unit tests for feature extraction and model training
7. Update the decision engine to incorporate the new model

**Example:**
```python
# src/features/new_fraud_features.py
from src.features.feature_extractor import BaseFeatureExtractor

class NewFraudFeatureExtractor(BaseFeatureExtractor):
    def __init__(self, config_path: str):
        super().__init__(config_path)
        # Initialize specific parameters
        
    def extract_features(self, data, customer_history=None):
        features = {}
        # Extract specific features
        return features
```

## Add New Data Source Integration

**Files to modify:**
- `/src/data/[source_name]_consumer.py` - Create consumer for the new data source
- `/src/data/schemas.py` - Add schema for the new data type
- `/config/app_config.yaml` - Add configuration for the new data source
- `/tests/unit/test_data.py` - Add unit tests for the new consumer

**Steps:**
1. Define the schema for the new data type in schemas.py
2. Create a consumer class for the new data source
3. Implement data validation and processing logic
4. Add configuration parameters to app_config.yaml
5. Create unit tests for the new consumer
6. Update the data processor to handle the new data type

**Example:**
```python
# src/data/new_source_consumer.py
from src.data.base_consumer import BaseConsumer

class NewSourceConsumer(BaseConsumer):
    def __init__(self, config_path: str):
        super().__init__(config_path)
        # Initialize specific parameters
        
    def consume(self):
        # Implement consumption logic
        pass
```

## Deploy New Version to Kubernetes

**Files to modify:**
- `/docker/Dockerfile.[component]` - Update Docker image version
- `/k8s/[component]-deployment.yaml` - Update deployment configuration
- `/config/app_config.yaml` - Update configuration if needed

**Steps:**
1. Build and tag new Docker images for affected components
2. Push images to the container registry
3. Update Kubernetes deployment files with new image versions
4. Apply Kubernetes configuration changes
5. Verify deployment status and monitor logs
6. Run integration tests to verify functionality

**Example:**
```bash
# Build and push Docker image
docker build -t telecom-fraud/api:v1.2.0 -f docker/Dockerfile.api .
docker push telecom-fraud/api:v1.2.0

# Update Kubernetes deployment
kubectl apply -f k8s/api-deployment.yaml

# Verify deployment
kubectl get pods -n telecom-fraud
kubectl logs -n telecom-fraud deployment/fraud-api
```

## Add New Rule to Decision Engine

**Files to modify:**
- `/src/decision/rules_engine.py` - Add new rule implementation
- `/config/rules_config.yaml` - Add rule configuration
- `/tests/unit/test_decision.py` - Add unit tests for the new rule

**Steps:**
1. Define the new rule in rules_engine.py
2. Implement rule evaluation logic
3. Add rule parameters to rules_config.yaml
4. Create unit tests for the new rule
5. Update the decision engine to incorporate the new rule

**Example:**
```python
# src/decision/rules_engine.py
def evaluate_new_rule(cdr, config):
    """
    Evaluate a new fraud detection rule.
    
    Args:
        cdr: Call Detail Record
        config: Rule configuration
        
    Returns:
        RuleResult with triggered status and confidence
    """
    # Rule implementation
    threshold = config.get('threshold', 0.8)
    
    # Check conditions
    if some_condition:
        return RuleResult(
            rule_id="NEW_RULE_001",
            rule_name="New Fraud Detection Rule",
            triggered=True,
            confidence=0.9,
            reason="Specific reason for triggering"
        )
    
    return RuleResult(
        rule_id="NEW_RULE_001",
        rule_name="New Fraud Detection Rule",
        triggered=False,
        confidence=0.0,
        reason=""
    )