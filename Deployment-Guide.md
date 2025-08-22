# NetIntel-OCR Deployment Guide

## Overview

NetIntel-OCR can be deployed in multiple configurations ranging from personal use on a laptop to enterprise-scale deployments serving hundreds of users. This guide helps you choose the right deployment option and provides step-by-step instructions for each scenario.

## Table of Contents

1. [Quick Start](#quick-start)
2. [Deployment Options](#deployment-options)
3. [Prerequisites](#prerequisites)
4. [Installation Methods](#installation-methods)
5. [Deployment Configurations](#deployment-configurations)
6. [Production Deployment](#production-deployment)
7. [Scaling Guide](#scaling-guide)
8. [Troubleshooting](#troubleshooting)

## Quick Start

### Fastest Setup (5 minutes)

```bash
# Install NetIntel-OCR
pip install netintel-ocr

# Install Ollama (AI model server)
curl -fsSL https://ollama.com/install.sh | sh
ollama serve

# Pull required models
ollama pull nanonets-ocr-s:latest
ollama pull qwen2.5vl:latest

# Process your first PDF
netintel-ocr document.pdf
```

### Docker Setup (10 minutes)

```bash
# Initialize project with Docker support
netintel-ocr --init --deployment-scale minimal

# Start services
cd netintel-ocr/docker
docker-compose -f docker-compose.minimal.yml up

# Access the service
curl http://localhost:8000/health
```

## Deployment Options

### Choose Your Deployment Scale

| Scale | Best For | Users | PDFs/Day | Resources | Setup Time |
|-------|----------|-------|----------|-----------|------------|
| **Minimal** | Personal/Testing | 1-5 | <10 | 2GB RAM, 1 CPU | 5 min |
| **Small** | Team | 5-20 | 10-50 | 4GB RAM, 2 CPU | 15 min |
| **Medium** | Department | 20-100 | 50-200 | 8GB RAM, 4 CPU | 30 min |
| **Large** | Enterprise | 100+ | 200+ | 16GB+ RAM, 8+ CPU | 1 hour |

## Prerequisites

### System Requirements

#### Minimal Requirements
- **OS**: Linux, macOS, or Windows (with WSL2)
- **RAM**: 2GB minimum
- **Storage**: 10GB free space
- **Python**: 3.9 or higher
- **Docker**: Optional but recommended

#### Recommended Requirements
- **OS**: Ubuntu 22.04 LTS or RHEL 8+
- **RAM**: 8GB or more
- **Storage**: 50GB SSD
- **GPU**: Optional (speeds up processing)
- **Network**: Stable internet for initial model downloads

### Required Software

1. **Python Environment**
   ```bash
   # Check Python version
   python --version  # Should be 3.9+
   
   # Install pip if needed
   curl https://bootstrap.pypa.io/get-pip.py | python
   ```

2. **Ollama (AI Model Server)**
   ```bash
   # Linux/macOS
   curl -fsSL https://ollama.com/install.sh | sh
   
   # Start Ollama service
   ollama serve
   ```

3. **Docker (Optional)**
   ```bash
   # Install Docker
   curl -fsSL https://get.docker.com | sh
   
   # Install Docker Compose
   pip install docker-compose
   ```

## Installation Methods

### Method 1: Direct Installation (Simplest)

```bash
# Install from PyPI
pip install netintel-ocr

# Verify installation
netintel-ocr --version

# Download required AI models
ollama pull nanonets-ocr-s:latest
ollama pull qwen2.5vl:latest
```

### Method 2: Docker Installation (Recommended)

```bash
# Initialize project
netintel-ocr --init --deployment-scale small

# Navigate to project
cd netintel-ocr

# Start services
docker-compose up -d

# Verify services
docker-compose ps
```

### Method 3: Kubernetes Installation (Enterprise)

```bash
# Initialize with Kubernetes support
netintel-ocr --init --deployment-scale large --with-kubernetes

# Deploy with Helm
helm install netintel-ocr ./helm/netintel-ocr \
  --namespace netintel-ocr \
  --create-namespace

# Check deployment
kubectl get all -n netintel-ocr
```

## Deployment Configurations

### 1. Minimal Deployment (Personal Use)

Perfect for individual users, testing, or demos.

#### Setup Steps

```bash
# Step 1: Initialize minimal configuration
netintel-ocr --init --deployment-scale minimal

# Step 2: Update configuration
cd netintel-ocr
nano .env  # Update OLLAMA_HOST if needed

# Step 3: Start the all-in-one container
docker-compose -f docker/docker-compose.minimal.yml up

# Step 4: Process documents
# Place PDFs in the input/ folder
# Access API at http://localhost:8000
```

#### Features
- Single container with all services
- Local file storage
- SQLite database
- No external dependencies
- 1 concurrent PDF processor

### 2. Small Team Deployment

Ideal for small teams or departments.

#### Setup Steps

```bash
# Step 1: Initialize small-scale configuration
netintel-ocr --init --deployment-scale small

# Step 2: Configure environment
cd netintel-ocr
cp .env.example .env
# Edit .env to set:
# - OLLAMA_HOST (your Ollama server)
# - MINIO credentials
# - Resource limits

# Step 3: Start core services
docker-compose up -d

# Step 4: Optional - Add MCP server for LLM integration
docker-compose --profile with-mcp up -d

# Step 5: Access services
# API: http://localhost:8000
# MinIO: http://localhost:9001
```

#### Features
- API server with 2 embedded workers
- Redis queue management
- MinIO object storage
- Persistent data volumes
- 2-5 concurrent PDFs

### 3. Medium Department Deployment

For departments or growing teams.

#### Setup Steps

```bash
# Step 1: Initialize medium configuration
netintel-ocr --init --deployment-scale medium

# Step 2: Configure load balancing
cd netintel-ocr
cp docker/nginx.conf.example docker/nginx.conf
# Edit nginx.conf for your domain

# Step 3: Start all services
docker-compose -f docker/docker-compose.medium.yml up -d

# Step 4: Scale workers as needed
docker-compose -f docker/docker-compose.medium.yml \
  up -d --scale pdf-worker=4

# Step 5: Access services
# Load Balancer: http://localhost:8003
# API: http://localhost:8000
# MCP: http://localhost:8001-8002
```

#### Features
- 4 PDF processing workers
- 2 MCP server instances
- Nginx load balancer
- Redis with persistence
- Enhanced MinIO cluster
- 5-10 concurrent PDFs

### 4. Enterprise Deployment

Full-scale production deployment with monitoring.

#### Setup Steps

```bash
# Step 1: Initialize with Kubernetes
netintel-ocr --init --deployment-scale large --with-kubernetes

# Step 2: Prepare Kubernetes cluster
kubectl create namespace netintel-ocr
kubectl config set-context --current --namespace=netintel-ocr

# Step 3: Install dependencies
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Step 4: Deploy NetIntel-OCR
helm install netintel-ocr ./helm/netintel-ocr \
  --set global.domain=netintel.example.com \
  --set ollama.host=ollama.example.com \
  --set storage.size=100Gi

# Step 5: Configure autoscaling
kubectl apply -f kubernetes/hpa.yaml
kubectl apply -f kubernetes/keda-scaledobject.yaml

# Step 6: Setup monitoring
helm install prometheus bitnami/prometheus
helm install grafana bitnami/grafana
```

#### Features
- Horizontal Pod Autoscaling
- KEDA-based queue scaling
- Distributed MinIO (4+ nodes)
- Redis cluster with Sentinel
- Prometheus + Grafana monitoring
- HAProxy load balancing
- 20+ concurrent PDFs

## Production Deployment

### Pre-Production Checklist

- [ ] **Infrastructure**
  - [ ] Sufficient CPU/RAM allocated
  - [ ] SSD storage for databases
  - [ ] Network bandwidth verified
  - [ ] Backup storage configured

- [ ] **Security**
  - [ ] TLS certificates installed
  - [ ] Firewall rules configured
  - [ ] Access control implemented
  - [ ] Secrets management setup

- [ ] **Monitoring**
  - [ ] Prometheus metrics configured
  - [ ] Grafana dashboards imported
  - [ ] Alert rules defined
  - [ ] Log aggregation setup

- [ ] **High Availability**
  - [ ] Multiple API instances
  - [ ] Redis replication configured
  - [ ] MinIO cluster deployed
  - [ ] Load balancer configured

### Production Configuration

#### Environment Variables

```bash
# Production .env file
NODE_ENV=production
LOG_LEVEL=info

# API Configuration
API_HOST=0.0.0.0
API_PORT=8000
API_WORKERS=4
RATE_LIMIT=1000/minute
CORS_ORIGINS=https://your-domain.com

# Security
JWT_SECRET=<generate-strong-secret>
API_KEY=<generate-api-key>
ENCRYPT_AT_REST=true

# Database
LANCEDB_PATH=/data/lancedb
DB_BACKUP_ENABLED=true
DB_BACKUP_SCHEDULE="0 2 * * *"

# Storage
STORAGE_TYPE=s3
S3_ENDPOINT=https://s3.amazonaws.com
S3_BUCKET=netintel-ocr-prod
S3_REGION=us-east-1
AWS_ACCESS_KEY_ID=<your-key>
AWS_SECRET_ACCESS_KEY=<your-secret>

# Queue
REDIS_URL=redis://redis-cluster:6379
REDIS_PASSWORD=<strong-password>
REDIS_CLUSTER_MODE=true

# Monitoring
METRICS_ENABLED=true
METRICS_PORT=9090
TRACING_ENABLED=true
JAEGER_ENDPOINT=http://jaeger:14268
```

#### Resource Limits

```yaml
# kubernetes/resources.yaml
resources:
  api:
    requests:
      memory: "2Gi"
      cpu: "1000m"
    limits:
      memory: "4Gi"
      cpu: "2000m"
  
  worker:
    requests:
      memory: "4Gi"
      cpu: "2000m"
    limits:
      memory: "8Gi"
      cpu: "4000m"
  
  minio:
    requests:
      memory: "1Gi"
      cpu: "500m"
    limits:
      memory: "2Gi"
      cpu: "1000m"
```

## Scaling Guide

### When to Scale

#### API Servers
Scale when:
- Response time > 500ms (p99)
- CPU usage > 80%
- Request queue > 100

```bash
# Docker
docker-compose up -d --scale api=3

# Kubernetes
kubectl scale deployment netintel-api --replicas=5
```

#### PDF Workers
Scale when:
- Processing queue > 10 documents
- Average processing time increasing
- Memory usage > 80%

```bash
# Docker
docker-compose up -d --scale pdf-worker=8

# Kubernetes
kubectl scale deployment netintel-worker --replicas=10
```

#### Storage (MinIO)
Scale when:
- Storage usage > 80%
- IOPS consistently high
- Bandwidth saturation

```bash
# Add MinIO nodes
docker-compose -f docker-compose.large.yml \
  up -d --scale minio=6
```

### Auto-Scaling Configuration

#### Kubernetes HPA

```yaml
# kubernetes/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: netintel-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: netintel-api
  minReplicas: 2
  maxReplicas: 20
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

#### KEDA Queue Scaling

```yaml
# kubernetes/keda-scaledobject.yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: netintel-worker-scaler
spec:
  scaleTargetRef:
    name: netintel-worker
  minReplicaCount: 1
  maxReplicaCount: 50
  triggers:
  - type: redis
    metadata:
      address: redis:6379
      listName: pdf_processing_queue
      listLength: "5"  # Scale up when queue > 5
```

## Troubleshooting

### Common Issues

#### Issue: Cannot Connect to Ollama

```bash
# Check Ollama status
curl http://localhost:11434/api/tags

# Solution 1: Start Ollama
ollama serve

# Solution 2: Update OLLAMA_HOST
export OLLAMA_HOST=http://your-ollama-server:11434

# Solution 3: Use Docker Ollama
docker run -d -p 11434:11434 ollama/ollama
```

#### Issue: Out of Memory Errors

```bash
# Check memory usage
docker stats

# Solution 1: Reduce workers
docker-compose up -d --scale pdf-worker=2

# Solution 2: Increase memory limits
# Edit docker-compose.yml
services:
  pdf-worker:
    mem_limit: 8g

# Solution 3: Optimize processing
netintel-ocr document.pdf --width 1500  # Reduce image size
```

#### Issue: Slow Processing

```bash
# Check processing metrics
curl http://localhost:8000/metrics | grep processing

# Solution 1: Scale workers
docker-compose up -d --scale pdf-worker=8

# Solution 2: Use faster models
netintel-ocr document.pdf --fast-extraction

# Solution 3: Enable caching
export EMBEDDING_CACHE_ENABLED=true
```

#### Issue: Storage Full

```bash
# Check disk usage
df -h

# Solution 1: Clean old data
netintel-ocr --cleanup --older-than 30

# Solution 2: Move to cloud storage
netintel-ocr --s3-sync --delete-local

# Solution 3: Expand storage
# Add more volume to Docker/Kubernetes
```

### Health Checks

```bash
# Check service health
curl http://localhost:8000/health

# Check all services (Docker)
docker-compose ps

# Check all services (Kubernetes)
kubectl get pods -n netintel-ocr

# View logs
docker-compose logs -f api
kubectl logs -f deployment/netintel-api
```

### Performance Monitoring

```bash
# View real-time metrics
curl http://localhost:8000/metrics

# Monitor with Prometheus
# Access at http://localhost:9090

# View Grafana dashboards
# Access at http://localhost:3000
# Default login: admin/admin
```

## Migration Guide

### Upgrading from v0.1.12

```bash
# Backup existing data
netintel-ocr --db-export backup-v0.1.12.json

# Stop old services
docker-compose down

# Pull new version
docker pull netintel-ocr:0.1.13

# Update configuration
cp .env .env.backup
# Edit .env for new version

# Start new services
docker-compose up -d

# Restore data
netintel-ocr --db-import backup-v0.1.12.json
```

### Migrating Between Scales

```bash
# From Minimal to Small
docker-compose -f docker-compose.minimal.yml down
docker-compose up -d

# From Docker to Kubernetes
netintel-ocr --db-export docker-backup.tar.gz
helm install netintel-ocr ./helm
kubectl exec -it netintel-api-0 -- \
  netintel-ocr --db-import /data/docker-backup.tar.gz
```

## Support Resources

- **Documentation**: https://github.com/your-org/netintel-ocr/docs
- **Issues**: https://github.com/your-org/netintel-ocr/issues
- **Community**: Discord/Slack channels
- **Commercial Support**: support@netintel-ocr.com

## Next Steps

1. **After Deployment**:
   - Test with sample PDFs
   - Configure backup schedules
   - Set up monitoring alerts
   - Document your configuration

2. **Optimization**:
   - Review [Performance Guide](./Performance-Guide.md)
   - Tune model selection
   - Optimize storage settings
   - Configure caching

3. **Integration**:
   - Connect to existing systems
   - Set up API authentication
   - Configure webhooks
   - Implement workflow automation