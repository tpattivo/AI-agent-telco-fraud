# IRSF Detection Models

This directory contains models for detecting International Revenue Share Fraud (IRSF). IRSF is a type of fraud where fraudsters artificially inflate traffic to premium rate numbers, often in countries with high termination rates.

## Model Overview

The IRSF detection system uses a combination of:
- Gradient boosting models (XGBoost/LightGBM) for classification
- Anomaly detection for identifying unusual patterns
- Rule-based systems for known fraud patterns

## Key Files

- `trainer.py` - Model training code
- `predictor.py` - Model prediction code
- `evaluation.py` - Model evaluation code
- `feature_importance.py` - Feature importance analysis
- `hyperparameter_tuning.py` - Hyperparameter optimization

## Model Architecture

### XGBoost Classifier

The primary model is an XGBoost classifier with the following characteristics:

```python
params = {
    'objective': 'binary:logistic',
    'eval_metric': 'auc',
    'eta': 0.1,
    'max_depth': 6,
    'min_child_weight': 1,
    'subsample': 0.8,
    'colsample_bytree': 0.8,
    'scale_pos_weight': 50,  # Adjusted for class imbalance
    'tree_method': 'hist'
}
```

### Feature Importance

The most important features for IRSF detection are:
1. Destination country risk score
2. Call duration
3. International call ratio
4. Premium number ratio
5. Deviation from average spend

## Training Process

The model is trained using:
- Historical CDR data with labeled fraud cases
- Features extracted by the IRSF feature extractor
- Class weighting to handle imbalanced data
- Cross-validation for robust evaluation

```python
# Example training process
def train_irsf_model(training_data, config):
    # Prepare features and labels
    X = training_data[feature_columns]
    y = training_data['is_fraud']
    
    # Split data
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=42
    )
    
    # Train model
    dtrain = xgb.DMatrix(X_train, label=y_train)
    dtest = xgb.DMatrix(X_test, label=y_test)
    
    model = xgb.train(
        params,
        dtrain,
        num_boost_round=1000,
        evals=[(dtrain, 'train'), (dtest, 'test')],
        early_stopping_rounds=50,
        verbose_eval=100
    )
    
    # Evaluate model
    y_pred = model.predict(dtest)
    precision, recall, thresholds = precision_recall_curve(y_test, y_pred)
    pr_auc = auc(recall, precision)
    
    return model, pr_auc
```

## Model Evaluation

The model is evaluated using:
- Precision-Recall AUC (primary metric due to class imbalance)
- ROC AUC
- Precision at different thresholds
- Recall at different thresholds
- F1 score
- Business impact metrics (potential fraud prevented, false positive cost)

## Deployment

The trained model is:
1. Registered in MLflow with all metadata
2. Versioned and tagged
3. Packaged as a Docker container
4. Deployed using Seldon Core in Kubernetes
5. Monitored for performance and drift

## Real-time Scoring

The model provides real-time scoring through:
- Feature store integration for low-latency feature serving
- Optimized inference using ONNX runtime
- Caching of frequent predictions
- Batch prediction capabilities for efficiency

## Model Updating

The model is regularly updated based on:
- New confirmed fraud cases
- Feedback from fraud analysts
- Performance monitoring
- Concept drift detection

## Integration

The IRSF model integrates with:
- Feature store for feature serving
- Decision engine for risk scoring
- Explanation system for interpretability
- Monitoring system for performance tracking