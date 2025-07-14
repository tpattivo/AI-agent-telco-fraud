# Kubernetes Deployment

This directory contains Kubernetes manifests for deploying the telecom fraud detection system to a Kubernetes cluster.

## Directory Structure

```
k8s/
├── base/                          # Base Kubernetes configurations
│   ├── namespaces.yaml            # Namespace definitions
│   ├── rbac.yaml                  # RBAC configurations
│   └── storage.yaml               # Storage configurations
├── data/                          # Data component deployments
│   ├── kafka.yaml                 # Kafka deployment
│   └── databases.yaml             # Database deployments
├── processing/                    # Processing component deployments
│   ├── feature-extraction.yaml    # Feature extraction deployment
│   └── stream-processing.yaml     # Stream processing deployment
├── models/                        # Model component deployments
│   ├── model-serving.yaml         # Model serving deployment
│   └── training-jobs.yaml         # Training job definitions
├── api/                           # API component deployments
│   ├── rest-api.yaml              # REST API deployment
│   └── grpc-services.yaml         # gRPC services deployment
└── monitoring/                    # Monitoring component deployments
    ├── prometheus.yaml            # Prometheus deployment
    ├── grafana.yaml               # Grafana deployment
    └── elasticsearch.yaml         # Elasticsearch deployment
```

## Base Configuration

### Namespaces

```yaml
# namespaces.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: telecom-fraud
  labels:
    name: telecom-fraud
    environment: production
---
apiVersion: v1
kind: Namespace
metadata:
  name: telecom-fraud-dev
  labels:
    name: telecom-fraud
    environment: development
---
apiVersion: v1
kind: Namespace
metadata:
  name: telecom-fraud-test
  labels:
    name: telecom-fraud
    environment: testing
```

### RBAC

```yaml
# rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: fraud-detection-sa
  namespace: telecom-fraud
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: fraud-detection-role
  namespace: telecom-fraud
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps", "secrets"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "statefulsets"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: fraud-detection-rolebinding
  namespace: telecom-fraud
subjects:
- kind: ServiceAccount
  name: fraud-detection-sa
  namespace: telecom-fraud
roleRef:
  kind: Role
  name: fraud-detection-role
  apiGroup: rbac.authorization.k8s.io
```

### Storage

```yaml
# storage.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fraud-detection-fast
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
allowVolumeExpansion: true
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fraud-detection-standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: st1
reclaimPolicy: Retain
allowVolumeExpansion: true
```

## API Deployment

```yaml
# rest-api.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fraud-detection-api
  namespace: telecom-fraud
spec:
  replicas: 3
  selector:
    matchLabels:
      app: fraud-detection
      component: api
  template:
    metadata:
      labels:
        app: fraud-detection
        component: api
    spec:
      serviceAccountName: fraud-detection-sa
      containers:
      - name: api
        image: registry.example.com/telecom-fraud/api:v1.0.0
        ports:
        - containerPort: 8080
          name: http
        - containerPort: 50051
          name: grpc
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        env:
        - name: CONFIG_PATH
          value: "/app/config/app/prod.yaml"
        - name: KAFKA_BOOTSTRAP_SERVERS
          valueFrom:
            configMapKeyRef:
              name: fraud-detection-config
              key: kafka.bootstrap.servers
        - name: SCHEMA_REGISTRY_URL
          valueFrom:
            configMapKeyRef:
              name: fraud-detection-config
              key: schema.registry.url
        - name: TIMESCALE_HOST
          valueFrom:
            configMapKeyRef:
              name: fraud-detection-config
              key: timescale.host
        - name: TIMESCALE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: fraud-detection-secrets
              key: timescale.password
        volumeMounts:
        - name: config-volume
          mountPath: /app/config
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
      volumes:
      - name: config-volume
        configMap:
          name: fraud-detection-config-files
---
apiVersion: v1
kind: Service
metadata:
  name: fraud-detection-api
  namespace: telecom-fraud
spec:
  selector:
    app: fraud-detection
    component: api
  ports:
  - name: http
    port: 80
    targetPort: 8080
  - name: grpc
    port: 50051
    targetPort: 50051
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: fraud-detection-api
  namespace: telecom-fraud
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  rules:
  - host: api.fraud-detection.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: fraud-detection-api
            port:
              name: http
  tls:
  - hosts:
    - api.fraud-detection.example.com
    secretName: fraud-detection-tls
```

## Data Processor Deployment

```yaml
# stream-processing.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fraud-detection-processor
  namespace: telecom-fraud
spec:
  replicas: 3
  selector:
    matchLabels:
      app: fraud-detection
      component: processor
  template:
    metadata:
      labels:
        app: fraud-detection
        component: processor
    spec:
      serviceAccountName: fraud-detection-sa
      containers:
      - name: processor
        image: registry.example.com/telecom-fraud/processor:v1.0.0
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        env:
        - name: CONFIG_PATH
          value: "/app/config/app/prod.yaml"
        - name: KAFKA_BOOTSTRAP_SERVERS
          valueFrom:
            configMapKeyRef:
              name: fraud-detection-config
              key: kafka.bootstrap.servers
        - name: SCHEMA_REGISTRY_URL
          valueFrom:
            configMapKeyRef:
              name: fraud-detection-config
              key: schema.registry.url
        - name: TIMESCALE_HOST
          valueFrom:
            configMapKeyRef:
              name: fraud-detection-config
              key: timescale.host
        - name: TIMESCALE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: fraud-detection-secrets
              key: timescale.password
        volumeMounts:
        - name: config-volume
          mountPath: /app/config
        livenessProbe:
          exec:
            command:
            - python
            - -m
            - src.utils.health_check
          initialDelaySeconds: 30
          periodSeconds: 10
      volumes:
      - name: config-volume
        configMap:
          name: fraud-detection-config-files
```

## Model Serving Deployment

```yaml
# model-serving.yaml
apiVersion: machinelearning.seldon.io/v1
kind: SeldonDeployment
metadata:
  name: fraud-detection-models
  namespace: telecom-fraud
spec:
  name: fraud-detection-models
  predictors:
  - name: irsf-model
    replicas: 2
    graph:
      name: irsf-model
      implementation: SKLEARN_SERVER
      modelUri: s3://fraud-detection-models/irsf/v1.0.0
      envSecretRefName: fraud-detection-s3-credentials
    componentSpecs:
    - spec:
        containers:
        - name: irsf-model
          resources:
            requests:
              memory: "1Gi"
              cpu: "500m"
            limits:
              memory: "2Gi"
              cpu: "1000m"
  - name: wangiri-model
    replicas: 2
    graph:
      name: wangiri-model
      implementation: TENSORFLOW_SERVER
      modelUri: s3://fraud-detection-models/wangiri/v1.0.0
      envSecretRefName: fraud-detection-s3-credentials
    componentSpecs:
    - spec:
        containers:
        - name: wangiri-model
          resources:
            requests:
              memory: "1Gi"
              cpu: "500m"
            limits:
              memory: "2Gi"
              cpu: "1000m"
```

## Monitoring Deployment

```yaml
# prometheus.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  namespace: telecom-fraud
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      serviceAccountName: fraud-detection-sa
      containers:
      - name: prometheus
        image: prom/prometheus:latest
        ports:
        - containerPort: 9090
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        volumeMounts:
        - name: prometheus-config
          mountPath: /etc/prometheus
        - name: prometheus-data
          mountPath: /prometheus
      volumes:
      - name: prometheus-config
        configMap:
          name: prometheus-config
      - name: prometheus-data
        persistentVolumeClaim:
          claimName: prometheus-data
---
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  namespace: telecom-fraud
spec:
  selector:
    app: prometheus
  ports:
  - port: 9090
    targetPort: 9090
  type: ClusterIP
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: prometheus-data
  namespace: telecom-fraud
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fraud-detection-standard
  resources:
    requests:
      storage: 50Gi
```

## Deployment Process

### Prerequisites

- Kubernetes cluster (EKS, GKE, AKS, or on-premises)
- kubectl configured to access the cluster
- Container registry with authentication
- Helm (optional, for Helm-based deployments)

### Deployment Steps

1. Create namespaces and RBAC:
   ```bash
   kubectl apply -f k8s/base/namespaces.yaml
   kubectl apply -f k8s/base/rbac.yaml
   kubectl apply -f k8s/base/storage.yaml
   ```

2. Create ConfigMaps and Secrets:
   ```bash
   kubectl create configmap fraud-detection-config \
     --from-file=config/app/prod.yaml \
     --namespace telecom-fraud
     
   kubectl create secret generic fraud-detection-secrets \
     --from-literal=timescale.password=your-password \
     --from-literal=redis.password=your-password \
     --from-literal=neo4j.password=your-password \
     --namespace telecom-fraud
   ```

3. Deploy data infrastructure:
   ```bash
   kubectl apply -f k8s/data/kafka.yaml
   kubectl apply -f k8s/data/databases.yaml
   ```

4. Deploy processing components:
   ```bash
   kubectl apply -f k8s/processing/feature-extraction.yaml
   kubectl apply -f k8s/processing/stream-processing.yaml
   ```

5. Deploy model serving:
   ```bash
   kubectl apply -f k8s/models/model-serving.yaml
   ```

6. Deploy API services:
   ```bash
   kubectl apply -f k8s/api/rest-api.yaml
   kubectl apply -f k8s/api/grpc-services.yaml
   ```

7. Deploy monitoring:
   ```bash
   kubectl apply -f k8s/monitoring/prometheus.yaml
   kubectl apply -f k8s/monitoring/grafana.yaml
   kubectl apply -f k8s/monitoring/elasticsearch.yaml
   ```

### Helm Deployment (Alternative)

For Helm-based deployments:

```bash
# Add Helm repositories
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add elastic https://helm.elastic.co
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install Kafka
helm install kafka bitnami/kafka -n telecom-fraud -f helm/values/kafka-values.yaml

# Install TimescaleDB
helm install timescaledb timescale/timescaledb-single -n telecom-fraud -f helm/values/timescaledb-values.yaml

# Install Redis
helm install redis bitnami/redis -n telecom-fraud -f helm/values/redis-values.yaml

# Install Neo4j
helm install neo4j neo4j/neo4j -n telecom-fraud -f helm/values/neo4j-values.yaml

# Install Prometheus and Grafana
helm install prometheus prometheus-community/kube-prometheus-stack -n telecom-fraud -f helm/values/prometheus-values.yaml

# Install Elasticsearch and Kibana
helm install elasticsearch elastic/elasticsearch -n telecom-fraud -f helm/values/elasticsearch-values.yaml
helm install kibana elastic/kibana -n telecom-fraud -f helm/values/kibana-values.yaml

# Install fraud detection components
helm install fraud-detection ./helm/fraud-detection -n telecom-fraud
```

## Scaling and High Availability

The Kubernetes deployment is designed for high availability and scalability:

- **Horizontal Pod Autoscaling**: All deployments have HPA configured
- **Pod Disruption Budgets**: Ensure minimum availability during updates
- **Anti-affinity Rules**: Spread pods across nodes
- **Resource Requests/Limits**: Proper resource allocation
- **Readiness/Liveness Probes**: Health monitoring
- **Rolling Updates**: Zero-downtime deployments

Example HPA configuration:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: fraud-detection-api-hpa
  namespace: telecom-fraud
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: fraud-detection-api
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

## Monitoring and Logging

The Kubernetes deployment includes comprehensive monitoring and logging:

- **Prometheus**: Metrics collection
- **Grafana**: Visualization and dashboards
- **Elasticsearch**: Log storage and search
- **Kibana**: Log visualization
- **Jaeger**: Distributed tracing

## Security Considerations

The Kubernetes deployment follows security best practices:

- **RBAC**: Least privilege principle
- **Network Policies**: Pod-to-pod communication control
- **Secret Management**: Secure storage of credentials
- **Pod Security Policies**: Container security
- **TLS**: Encrypted communication
- **Image Scanning**: Vulnerability detection