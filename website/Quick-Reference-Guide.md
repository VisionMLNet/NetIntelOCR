# NetIntel-OCR Quick Reference Guide

## Installation & Setup

### Quick Install
```bash
# Install NetIntel-OCR
pip install netintel-ocr

# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh
ollama serve

# Pull required models
ollama pull nanonets-ocr-s:latest
ollama pull qwen2.5vl:latest

# Process first PDF
netintel-ocr document.pdf
```

### Docker Setup
```bash
# Initialize project
netintel-ocr --init --deployment-scale minimal

# Start services
cd netintel-ocr/docker
docker-compose -f docker-compose.minimal.yml up
```

## Common Commands

### Basic Processing

| Command | Description |
|---------|-------------|
| `netintel-ocr document.pdf` | Process PDF with auto-detection |
| `netintel-ocr doc.pdf --text-only` | Extract text only (faster) |
| `netintel-ocr doc.pdf --network-only` | Extract network diagrams only |
| `netintel-ocr doc.pdf --fast-extraction` | Fast mode (50-70% faster) |
| `netintel-ocr doc.pdf --resume` | Resume from checkpoint |
| `netintel-ocr doc.pdf --output ./results` | Specify output directory |

### Page Range Processing
```bash
# Process specific pages
netintel-ocr document.pdf --start 10 --end 20

# Process with custom timeout
netintel-ocr document.pdf --timeout 120
```

### Model Selection
```bash
# Use specific text model
netintel-ocr doc.pdf --model nanonets-ocr-s:latest

# Use different network model
netintel-ocr doc.pdf --network-model qwen2.5vl:latest

# Specify embedding model
netintel-ocr doc.pdf --embedding-model nomic-embed-text
```

## Batch Processing

### Basic Batch Commands
```bash
# Process all PDFs in directory
netintel-ocr --batch-ingest --input-pattern "*.pdf"

# Parallel processing with 8 workers
netintel-ocr --batch-ingest --input-pattern "docs/*.pdf" --parallel 8

# Auto-merge to centralized database
netintel-ocr --batch-ingest --input-pattern "*.pdf" --auto-merge

# Skip duplicates
netintel-ocr --batch-ingest --input-pattern "*.pdf" --dedupe
```

## Database Operations

### Query Commands
```bash
# Basic search
netintel-ocr --query "network topology"

# Advanced search with filters
netintel-ocr --query "firewall" --rerank --query-limit 20

# Export results
netintel-ocr --query "VPN" --output-format markdown > results.md
netintel-ocr --query "security" --output-format json > results.json
netintel-ocr --query "config" --output-format csv > results.csv
```

### Database Management
```bash
# Show statistics
netintel-ocr --db-stats

# Merge databases
netintel-ocr --merge-to-centralized

# Optimize database
netintel-ocr --db-optimize

# Export database
netintel-ocr --db-export backup.json
```

## Configuration Options

### Network Diagram Options
```bash
# Adjust detection confidence
netintel-ocr doc.pdf --confidence 0.8

# Disable icons in diagrams
netintel-ocr doc.pdf --no-icons

# Force multi-diagram extraction
netintel-ocr doc.pdf --multi-diagram

# Extract diagram without text
netintel-ocr doc.pdf --diagram-only
```

### Table Extraction Options
```bash
# Disable table extraction
netintel-ocr doc.pdf --no-tables

# Specify extraction method
netintel-ocr doc.pdf --table-method hybrid    # Best of both
netintel-ocr doc.pdf --table-method pdfplumber # Fast
netintel-ocr doc.pdf --table-method llm       # Accurate

# Adjust table confidence
netintel-ocr doc.pdf --table-confidence 0.8

# Save tables as JSON
netintel-ocr doc.pdf --save-table-json
```

### Vector Generation Options
```bash
# Disable vector generation
netintel-ocr doc.pdf --no-vector

# Configure chunking
netintel-ocr doc.pdf --chunk-size 1500 --chunk-overlap 200

# Chunking strategies
netintel-ocr doc.pdf --chunk-strategy semantic  # Default
netintel-ocr doc.pdf --chunk-strategy fixed     # Fixed size
netintel-ocr doc.pdf --chunk-strategy sentence  # Sentence boundary

# Regenerate vectors
netintel-ocr doc.pdf --vector-regenerate
```

## Server Modes

### API Server
```bash
# Start API server
netintel-ocr --api

# API with embedded workers
netintel-ocr --api --embedded-workers --max-workers 2

# Custom host/port
netintel-ocr --api --api-host 0.0.0.0 --api-port 8080
```

### MCP Server
```bash
# Start MCP server
netintel-ocr --mcp

# Custom configuration
netintel-ocr --mcp --mcp-host 0.0.0.0 --mcp-port 8001
```

### All-in-One Mode
```bash
# Single process with all services
netintel-ocr --all-in-one

# Minimal resources
netintel-ocr --all-in-one --local-storage --sqlite-queue
```

## Cloud Storage

### S3/MinIO Integration
```bash
# Sync to cloud storage
netintel-ocr --s3-sync --endpoint http://minio:9000

# Backup to S3
netintel-ocr --backup --bucket netintel-backups

# Restore from backup
netintel-ocr --restore backup-2025-08-21.tar.gz

# Use S3 for LanceDB
netintel-ocr --query "test" --lancedb-uri s3://bucket/lancedb
```

## Performance Optimization

### Speed Optimization
```bash
# Fast text-only processing
netintel-ocr doc.pdf --text-only --parallel 8

# Fast extraction mode
netintel-ocr doc.pdf --fast-extraction --timeout 30

# Skip unnecessary features
netintel-ocr doc.pdf --no-tables --no-vector

# Reduce image size
netintel-ocr large.pdf --width 1500
```

### Memory Optimization
```bash
# Reduce parallel workers
netintel-ocr --batch-ingest --parallel 2

# Smaller batch size
netintel-ocr --batch-ingest --batch-size 16

# Clean up after processing
netintel-ocr doc.pdf --no-keep-images
```

## Deployment Scales

### Quick Deployment Reference

| Scale | Command | Users | Resources |
|-------|---------|-------|-----------|
| **Minimal** | `--deployment-scale minimal` | 1-5 | 2GB RAM |
| **Small** | `--deployment-scale small` | 5-20 | 4GB RAM |
| **Medium** | `--deployment-scale medium` | 20-100 | 8GB RAM |
| **Large** | `--deployment-scale large` | 100+ | 16GB+ RAM |

### Initialize Deployment
```bash
# Minimal (personal use)
netintel-ocr --init --deployment-scale minimal

# Small team
netintel-ocr --init --deployment-scale small

# Department
netintel-ocr --init --deployment-scale medium

# Enterprise with Kubernetes
netintel-ocr --init --deployment-scale large --with-kubernetes
```

## Docker Commands

### Container Management
```bash
# Start services
docker-compose up -d

# View logs
docker-compose logs -f netintel-api

# Scale workers
docker-compose up -d --scale pdf-worker=5

# Stop services
docker-compose down

# Remove everything
docker-compose down -v
```

## Kubernetes Commands

### Helm Deployment
```bash
# Install
helm install netintel-ocr ./helm/netintel-ocr \
  --namespace netintel-ocr \
  --create-namespace

# Upgrade
helm upgrade netintel-ocr ./helm/netintel-ocr

# Scale
kubectl scale deployment netintel-worker --replicas=10

# Check status
kubectl get all -n netintel-ocr
```

## Environment Variables

### Essential Variables
```bash
# Ollama configuration
export OLLAMA_HOST=http://localhost:11434

# Output directory
export NETINTEL_OUTPUT=/data/output

# Parallel workers
export NETINTEL_PARALLEL_WORKERS=8

# Auto-merge
export NETINTEL_AUTO_MERGE=true

# Deduplication
export NETINTEL_DEDUPE=true
```

### Cloud Storage
```bash
# MinIO/S3
export MINIO_ENDPOINT=http://minio:9000
export AWS_ACCESS_KEY_ID=minioadmin
export AWS_SECRET_ACCESS_KEY=minioadmin

# Embedding provider
export NETINTEL_EMBEDDING_PROVIDER=ollama
export NETINTEL_EMBEDDING_MODEL=nomic-embed-text
```

## Troubleshooting

### Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| **Ollama connection failed** | Check `ollama serve` is running, verify `OLLAMA_HOST` |
| **Out of memory** | Reduce workers: `--parallel 2`, reduce image size: `--width 1000` |
| **Slow processing** | Use `--text-only` for text docs, enable `--fast-extraction` |
| **Model not found** | Pull model: `ollama pull model-name:latest` |
| **Permission denied** | Check file permissions, run with appropriate user |
| **Port in use** | Change port: `--api-port 8001` or check existing services |

### Debug Commands
```bash
# Enable debug output
netintel-ocr doc.pdf --debug

# Verbose logging
netintel-ocr doc.pdf --verbose

# Keep intermediate files
netintel-ocr doc.pdf --keep-images

# Check service health
curl http://localhost:8000/health

# View metrics
curl http://localhost:8000/metrics
```

## File Structure

### Output Directory Structure
```
output/
├── index.md                      # Master index
└── <md5_checksum>/              # Per-document folder
    ├── markdown/                # Extracted text
    │   ├── page_001.md         # Individual pages
    │   └── document.md         # Merged document
    ├── lancedb/                # Vector database
    │   ├── chunks.jsonl        # Document chunks
    │   └── metadata.json       # Metadata
    ├── tables/                 # Extracted tables
    │   └── tables.json        # Table data
    └── images/                # Original images (if --keep-images)
```

## API Endpoints

### REST API Reference
```bash
# Upload document
curl -X POST http://localhost:8000/api/v1/documents/upload \
  -F "file=@document.pdf"

# Get job status
curl http://localhost:8000/api/v1/jobs/{job_id}

# Query documents
curl -X POST http://localhost:8000/api/v1/query \
  -H "Content-Type: application/json" \
  -d '{"query": "network diagram", "limit": 10}'

# Health check
curl http://localhost:8000/health

# Metrics
curl http://localhost:8000/metrics
```

## Configuration File

### Sample config.yaml
```yaml
processing:
  mode: hybrid
  parallel_workers: 4
  timeout: 300

models:
  text: nanonets-ocr-s:latest
  network: qwen2.5vl:latest
  embedding: nomic-embed-text

storage:
  output_dir: output
  s3:
    enabled: true
    endpoint: http://minio:9000
    bucket: netintel-ocr

vector:
  enabled: true
  chunk_size: 1000
  chunk_overlap: 100
```

### Using Configuration
```bash
# Use config file
netintel-ocr doc.pdf --config config.yaml

# Override config settings
netintel-ocr doc.pdf --config config.yaml --parallel 8
```

## Performance Benchmarks

### Processing Speed

| Mode | Pages/Min | Use Case |
|------|-----------|----------|
| **Text-only** | 4-6 | Documents without diagrams |
| **Fast extraction** | 2-3 | Quick processing |
| **Standard** | 1-2 | Balanced accuracy |
| **High accuracy** | 0.5-1 | Complex diagrams |

### Resource Usage

| Component | RAM | CPU | Storage |
|-----------|-----|-----|---------|
| **API Server** | 1-2GB | 0.5-1 core | Minimal |
| **Worker** | 2-4GB | 1-2 cores | 1GB/PDF |
| **Ollama** | 4-8GB | 2-4 cores | Model size |
| **Database** | 1-2GB | 0.5 core | Data size |

## Quick Tips

### Best Practices
- ✅ Use `--text-only` for documents without diagrams
- ✅ Enable `--dedupe` to avoid reprocessing
- ✅ Use `--auto-merge` for batch processing
- ✅ Set appropriate `--parallel` based on resources
- ✅ Use `--resume` for long documents
- ✅ Enable `--fast-extraction` for initial testing

### Performance Tips
- 🚀 Use lighter models for simple documents
- 🚀 Reduce image size with `--width` for large PDFs
- 🚀 Use `--chunk-strategy fixed` for consistent chunks
- 🚀 Enable caching for repeated queries
- 🚀 Use SSD storage for better I/O performance

### Security Tips
- 🔒 Use `--no-cloud-sync` for sensitive documents
- 🔒 Set strong passwords for services
- 🔒 Enable TLS for production deployments
- 🔒 Restrict network access to services
- 🔒 Regular backups with `--db-export`

## Support & Resources

- **GitHub**: https://github.com/your-org/netintel-ocr
- **Documentation**: Full guides and API reference
- **Issues**: Report bugs and request features
- **Community**: Discord/Slack for discussions
- **Commercial**: support@netintel-ocr.com