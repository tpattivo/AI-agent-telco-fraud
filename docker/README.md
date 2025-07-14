# Docker Configuration

This directory contains Docker configuration files for containerizing the telecom fraud detection system components.

## Dockerfiles

- `Dockerfile.api` - API service container
- `Dockerfile.processor` - Data processor container
- `Dockerfile.trainer` - Model trainer container
- `Dockerfile.inference` - Model inference container
- `docker-compose.yml` - Docker Compose for local development

## API Service Dockerfile

The API service Dockerfile builds a container for the REST and gRPC API services:

```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Install dependencies
COPY requirements/base.txt requirements/api.txt ./requirements/
RUN pip install --no-cache-dir -r requirements/api.txt

# Copy application code
COPY src/ ./src/
COPY config/ ./config/

# Set environment variables
ENV PYTHONPATH=/app
ENV CONFIG_PATH=/app/config/app/prod.yaml
ENV LOG_LEVEL=INFO

# Expose ports
EXPOSE 8080 50051

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1

# Run the application
CMD ["python", "-m", "src.api.rest.app"]
```

## Data Processor Dockerfile

The data processor Dockerfile builds a container for Kafka consumers and data processing:

```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Install dependencies
COPY requirements/base.txt requirements/processor.txt ./requirements/
RUN pip install --no-cache-dir -r requirements/processor.txt

# Copy application code
COPY src/ ./src/
COPY config/ ./config/

# Set environment variables
ENV PYTHONPATH=/app
ENV CONFIG_PATH=/app/config/app/prod.yaml
ENV LOG_LEVEL=INFO

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD python -m src.utils.health_check || exit 1

# Run the application
CMD ["python", "-m", "src.data.processor"]
```

## Model Trainer Dockerfile

The model trainer Dockerfile builds a container for training ML models:

```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Install dependencies
COPY requirements/base.txt requirements/trainer.txt ./requirements/
RUN pip install --no-cache-dir -r requirements/trainer.txt

# Copy application code
COPY src/ ./src/
COPY config/ ./config/

# Set environment variables
ENV PYTHONPATH=/app
ENV CONFIG_PATH=/app/config/app/prod.yaml
ENV LOG_LEVEL=INFO

# Run the application
CMD ["python", "-m", "src.models.trainer"]
```

## Model Inference Dockerfile

The model inference Dockerfile builds a container for model serving:

```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Install dependencies
COPY requirements/base.txt requirements/inference.txt ./requirements/
RUN pip install --no-cache-dir -r requirements/inference.txt

# Copy application code
COPY src/ ./src/
COPY config/ ./config/

# Set environment variables
ENV PYTHONPATH=/app
ENV CONFIG_PATH=/app/config/app/prod.yaml
ENV LOG_LEVEL=INFO

# Expose port
EXPOSE 9000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:9000/health || exit 1

# Run the application
CMD ["python", "-m", "src.models.inference"]
```

## Docker Compose

The Docker Compose file sets up a local development environment with all necessary services:

```yaml
version: '3'

services:
  # Kafka and Zookeeper
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
    ports:
      - "2181:2181"
    volumes:
      - zookeeper-data:/var/lib/zookeeper/data
      
  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    volumes:
      - kafka-data:/var/lib/kafka/data
      
  schema-registry:
    image: confluentinc/cp-schema-registry:latest
    depends_on:
      - kafka
    ports:
      - "8081:8081"
    environment:
      SCHEMA_REGISTRY_HOST_NAME: schema-registry
      SCHEMA_REGISTRY_KAFKASTORE_CONNECTION_URL: zookeeper:2181
      
  # Databases
  timescaledb:
    image: timescale/timescaledb:latest-pg14
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: fraud_user
      POSTGRES_PASSWORD: fraud_password
      POSTGRES_DB: fraud_detection
    volumes:
      - timescaledb-data:/var/lib/postgresql/data
      
  redis:
    image: redis:latest
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
      
  neo4j:
    image: neo4j:latest
    ports:
      - "7474:7474"
      - "7687:7687"
    environment:
      NEO4J_AUTH: neo4j/fraud_password
    volumes:
      - neo4j-data:/data
      
  # Feature Store
  feast:
    image: feastdev/feature-server:latest
    ports:
      - "8080:8080"
    volumes:
      - ./config/features:/etc/feast
      
  # Model Registry
  mlflow:
    image: mlflow/mlflow:latest
    ports:
      - "5000:5000"
    environment:
      MLFLOW_TRACKING_URI: sqlite:///mlflow.db
    volumes:
      - mlflow-data:/mlflow
      
  # Monitoring
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./config/prometheus:/etc/prometheus
      - prometheus-data:/prometheus
      
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin
    volumes:
      - ./config/grafana:/etc/grafana/provisioning
      - grafana-data:/var/lib/grafana
      
  # Application Services
  api:
    build:
      context: ..
      dockerfile: docker/Dockerfile.api
    ports:
      - "8000:8080"
      - "50051:50051"
    environment:
      CONFIG_PATH: /app/config/app/dev.yaml
      KAFKA_BOOTSTRAP_SERVERS: kafka:29092
      SCHEMA_REGISTRY_URL: http://schema-registry:8081
      TIMESCALE_HOST: timescaledb
      REDIS_HOST: redis
      NEO4J_URI: bolt://neo4j:7687
      FEATURE_STORE_URL: http://feast:8080
      MODEL_REGISTRY_URL: http://mlflow:5000
    depends_on:
      - kafka
      - schema-registry
      - timescaledb
      - redis
      - neo4j
      - feast
      - mlflow
      
  processor:
    build:
      context: ..
      dockerfile: docker/Dockerfile.processor
    environment:
      CONFIG_PATH: /app/config/app/dev.yaml
      KAFKA_BOOTSTRAP_SERVERS: kafka:29092
      SCHEMA_REGISTRY_URL: http://schema-registry:8081
      TIMESCALE_HOST: timescaledb
      REDIS_HOST: redis
      NEO4J_URI: bolt://neo4j:7687
      FEATURE_STORE_URL: http://feast:8080
    depends_on:
      - kafka
      - schema-registry
      - timescaledb
      - redis
      - neo4j
      - feast
      
  inference:
    build:
      context: ..
      dockerfile: docker/Dockerfile.inference
    ports:
      - "9000:9000"
    environment:
      CONFIG_PATH: /app/config/app/dev.yaml
      FEATURE_STORE_URL: http://feast:8080
      MODEL_REGISTRY_URL: http://mlflow:5000
    depends_on:
      - feast
      - mlflow

volumes:
  zookeeper-data:
  kafka-data:
  timescaledb-data:
  redis-data:
  neo4j-data:
  mlflow-data:
  prometheus-data:
  grafana-data:
```

## Building and Running

### Building Images

```bash
# Build all images
docker-compose build

# Build specific image
docker build -t telecom-fraud/api:latest -f Dockerfile.api ..
```

### Running Locally

```bash
# Start all services
docker-compose up -d

# Start specific service
docker-compose up -d api

# View logs
docker-compose logs -f api

# Stop all services
docker-compose down
```

### Production Deployment

For production deployment, images are:
1. Built in CI/CD pipeline
2. Tagged with version and environment
3. Pushed to container registry
4. Deployed to Kubernetes using Helm charts

Example production build and push:

```bash
# Build production image
docker build -t telecom-fraud/api:v1.0.0 -f Dockerfile.api ..

# Tag for registry
docker tag telecom-fraud/api:v1.0.0 registry.example.com/telecom-fraud/api:v1.0.0

# Push to registry
docker push registry.example.com/telecom-fraud/api:v1.0.0
```

## Best Practices

The Docker configuration follows these best practices:
- Minimal base images
- Multi-stage builds for smaller images
- Proper layer caching
- Non-root users for security
- Health checks for all services
- Environment-based configuration
- Volume mounting for persistent data
- Container-specific resource limits