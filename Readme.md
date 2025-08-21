# NetIntel-OCR Reference Architecture Guide

## Executive Summary

NetIntel-OCR is an intelligent document processing system that automatically extracts and converts network diagrams, tables, and text from PDF documents into structured, searchable data. The system leverages multimodal AI models to understand both visual and textual content, creating a comprehensive knowledge base with vector search capabilities.

## Quick Start

### Installation

```bash
# Install NetIntel-OCR from PyPI
pip install netintel-ocr

# Verify installation
netintel-ocr --version
```

### Prerequisites

```bash
# Install and start Ollama (required for AI processing)
curl -fsSL https://ollama.com/install.sh | sh
ollama serve

# Pull required models
ollama pull nanonets-ocr-s:latest
ollama pull qwen2.5vl:latest
```

### Initialize Project

```bash
# Create a new NetIntel-OCR project with Docker and configuration
netintel-ocr --init

# This creates:
# ├── docker/           # Docker configuration
# ├── helm/            # Kubernetes charts
# ├── config/          # Configuration files
# ├── input/           # PDF input directory
# └── output/          # Processing output
```

### Simple Extraction Example

```bash
# 1. Basic PDF processing (text + diagrams + tables)
netintel-ocr document.pdf

# 2. Process with specific options
netintel-ocr report.pdf --output ./results --keep-images

# 3. Fast text-only extraction
netintel-ocr document.pdf --text-only

# 4. Extract network diagrams
netintel-ocr network-topology.pdf --network-only

# 5. Batch process multiple PDFs
netintel-ocr --batch-ingest --input-pattern "*.pdf" --parallel 4

# 6. Query processed documents
netintel-ocr --query "firewall configuration"

# 7. View what was created
ls output/
# Output structure:
# output/
# ├── index.md                    # Document index
# └── abc123.../                  # Unique folder per document
#     ├── markdown/               # Extracted text
#     │   └── document.md        # Complete document
#     ├── lancedb/               # Vector database
#     │   └── chunks.jsonl       # Searchable chunks
#     └── tables/                # Extracted tables
```

### Complete Workflow Example

```bash
# Step 1: Install
pip install netintel-ocr

# Step 2: Initialize project
netintel-ocr --init
cd netintel-ocr

# Step 3: Process a document
netintel-ocr sample.pdf

# Step 4: Process multiple documents
netintel-ocr --batch-ingest --input-pattern "docs/*.pdf"

# Step 5: Merge to centralized database
netintel-ocr --merge-to-centralized

# Step 6: Search your documents
netintel-ocr --query "network security" --output-format markdown

# Step 7: View statistics
netintel-ocr --db-stats
```

### Docker Quick Start

```bash
# Using Docker (after --init)
docker-compose up -d
docker exec netintel-ocr netintel-ocr /input/document.pdf
```

## System Architecture

### High-Level Component Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         User Interface Layer                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │
│  │   CLI Entry  │  │ Web API      │  │ Batch Jobs   │  │ Monitoring │ │
│  │   Point      │  │ (Future)     │  │ Scheduler    │  │ Dashboard  │ │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └─────┬──────┘ │
│         │                  │                  │                │         │
└─────────┼──────────────────┼──────────────────┼────────────────┼─────────┘
          │                  │                  │                │
┌─────────▼──────────────────▼──────────────────▼────────────────▼─────────┐
│                        Processing Orchestration Layer                      │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                            │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐                │
│  │ Single Doc    │  │ Batch         │  │ Query         │                │
│  │ Processor     │  │ Processor     │  │ Engine        │                │
│  └───────┬───────┘  └───────┬───────┘  └───────┬───────┘                │
│          │                   │                   │                        │
│  ┌───────▼───────────────────▼───────────────────▼────────┐              │
│  │              Centralized Database Manager               │              │
│  └──────────────────────────┬──────────────────────────────┘              │
│                             │                                             │
└─────────────────────────────┼─────────────────────────────────────────────┘
                              │
┌─────────────────────────────▼─────────────────────────────────────────────┐
│                         Core Processing Engine                             │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
│  │ PDF Handler │  │ AI Model    │  │ Table       │  │ Network     │    │
│  │             │  │ Interface   │  │ Extractor   │  │ Diagram     │    │
│  │             │  │             │  │             │  │ Detector    │    │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘    │
│         │                 │                 │                 │           │
│  ┌──────▼─────────────────▼─────────────────▼─────────────────▼──────┐   │
│  │                    Content Processing Pipeline                     │   │
│  └────────────────────────────┬───────────────────────────────────────┘   │
│                               │                                           │
└───────────────────────────────┼───────────────────────────────────────────┘
                                │
┌───────────────────────────────▼───────────────────────────────────────────┐
│                         Data & Storage Layer                               │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
│  │ Vector DB   │  │ Document    │  │ Embedding   │  │ Cloud       │    │
│  │ (LanceDB)   │  │ Storage     │  │ Cache       │  │ Storage     │    │
│  │             │  │             │  │             │  │ (S3/MinIO)  │    │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘    │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
                                │
┌───────────────────────────────▼───────────────────────────────────────────┐
│                        External Services Layer                             │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
│  │ Ollama      │  │ OpenAI      │  │ HuggingFace │  │ MinIO/S3    │    │
│  │ Server      │  │ API         │  │ Models      │  │ Storage     │    │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘    │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

## Component Descriptions

### User Interface Layer

#### CLI Entry Point
The command-line interface provides access to all system functionality through a comprehensive set of commands and options. It serves as the primary interaction point for users, supporting both single document processing and batch operations.

**Key Responsibilities:**
- Command parsing and validation
- Configuration management
- Progress tracking and reporting
- Error handling and user feedback
- Mode selection (text-only, network-only, hybrid)

#### Web API (Future)
A planned RESTful API endpoint for programmatic access to the system, enabling integration with other applications and services.

#### Batch Jobs Scheduler
Manages scheduled and recurring processing tasks, enabling automated document ingestion from monitored directories or cloud storage locations.

#### Monitoring Dashboard
Provides real-time visibility into system performance, processing statistics, and health metrics.

### Processing Orchestration Layer

#### Single Document Processor
Handles individual PDF document processing with intelligent mode selection and error recovery.

**Processing Modes:**
- **Hybrid Mode**: Automatic detection and extraction of both text and diagrams
- **Text-Only Mode**: Fast text extraction without diagram detection
- **Network-Only Mode**: Exclusive focus on network diagram extraction

**Key Features:**
- MD5-based document identification
- Checkpoint/resume capability
- Multi-model support for optimized processing
- Comprehensive metrics and reporting

#### Batch Processor
Manages parallel processing of multiple documents with intelligent work distribution and progress tracking.

**Capabilities:**
- Parallel worker management (1-16 workers)
- Progress tracking with ETA calculation
- Checkpoint-based resume on interruption
- Auto-merge to centralized database
- Resource management and optimization

#### Query Engine
Provides advanced search capabilities across the document corpus with filtering and relevance ranking.

**Query Features:**
- Vector similarity search
- Multi-field filtering
- Result reranking strategies
- Multiple output formats
- Pagination and result limiting

### Core Processing Engine

#### PDF Handler
Manages PDF document operations including page extraction, image conversion, and metadata extraction.

**Functions:**
- PDF validation and page counting
- Page-to-image conversion
- Resolution optimization
- Page range selection
- Metadata extraction

#### AI Model Interface
Provides unified access to multiple AI model providers for vision, language, and embedding tasks.

**Supported Providers:**
- Ollama (local models)
- OpenAI API
- HuggingFace models
- Custom model endpoints

**Model Types:**
- Text extraction models
- Vision-language models
- Embedding models
- Reranking models

#### Table Extractor
Specialized module for detecting and extracting tabular data from documents.

**Extraction Methods:**
- Library-based extraction (fast)
- LLM-based extraction (accurate)
- Hybrid approach (optimal)

**Features:**
- Complex table support
- Type inference
- Data validation
- Multiple output formats

#### Network Diagram Detector
Identifies and extracts network topology diagrams with component and connection recognition.

**Detection Capabilities:**
- Confidence-based identification
- Multi-diagram support per page
- Component type recognition
- Connection mapping
- Protocol identification

### Data & Storage Layer

#### Vector Database (LanceDB)
High-performance vector storage for similarity search and retrieval.

**Structure:**
- Document metadata table
- Chunk vectors table
- Embedding indices
- Query optimization indices

**Features:**
- Scalable vector storage
- Fast similarity search
- Metadata filtering
- ACID compliance

#### Document Storage
Manages original documents and processed outputs with versioning and deduplication.

**Organization:**
- MD5-based folder structure
- Version control
- Deduplication logic
- Archive management

#### Embedding Cache
Intelligent caching system for embedding vectors to optimize performance and reduce costs.

**Cache Features:**
- TTL-based expiration
- LRU eviction policy
- Hit/miss tracking
- Size management

#### Cloud Storage
Provides distributed storage capabilities for scalability and collaboration.

**Supported Backends:**
- Amazon S3
- MinIO
- Azure Blob Storage
- Google Cloud Storage

**Features:**
- Bi-directional sync
- Backup and restore
- Version control
- Access control

## Command-Line Interface Reference

### Basic Usage

```bash
netintel-ocr [OPTIONS] <pdf_path>
```

### Document Processing Commands

#### Single Document Processing

```bash
# Basic processing with automatic detection
netintel-ocr document.pdf

# Specify output directory
netintel-ocr document.pdf --output /path/to/output

# Process specific page range
netintel-ocr document.pdf --start 10 --end 20

# Resume from checkpoint
netintel-ocr document.pdf --resume
```

#### Processing Modes

```bash
# Text-only mode (faster, skips diagram detection)
netintel-ocr document.pdf --text-only

# Network diagrams only
netintel-ocr document.pdf --network-only

# Fast extraction mode (50-70% faster)
netintel-ocr document.pdf --fast-extraction

# Force multi-diagram extraction
netintel-ocr document.pdf --multi-diagram
```

#### Model Selection

```bash
# Specify text extraction model
netintel-ocr document.pdf --model nanonets-ocr-s:latest

# Use different model for network diagrams
netintel-ocr document.pdf --network-model qwen2.5vl:latest

# Specify embedding model
netintel-ocr document.pdf --embedding-model nomic-embed-text
```

### Batch Processing Commands

#### Parallel Batch Processing

```bash
# Process all PDFs in directory
netintel-ocr --batch-ingest --input-pattern "*.pdf"

# Process with custom parallel workers
netintel-ocr --batch-ingest --input-pattern "docs/*.pdf" --parallel 8

# Batch processing with auto-merge
netintel-ocr --batch-ingest --input-pattern "*.pdf" --auto-merge

# Skip duplicate documents
netintel-ocr --batch-ingest --input-pattern "*.pdf" --dedupe
```

### Database Management Commands

#### Query Operations

```bash
# Basic query
netintel-ocr --query "network topology diagram"

# Query with filters and reranking
netintel-ocr --query "firewall configuration" --rerank --query-limit 20

# Query with similarity threshold
netintel-ocr --query "VPN setup" --similarity-threshold 0.8

# Export query results
netintel-ocr --query "security policies" --output-format markdown
```

#### Database Merging

```bash
# Merge per-document databases to centralized
netintel-ocr --merge-to-centralized

# Merge with embedding generation
netintel-ocr --merge-to-centralized --compute-embeddings

# Force merge (overwrite duplicates)
netintel-ocr --merge-to-centralized --force
```

#### Database Administration

```bash
# Show database statistics
netintel-ocr --db-stats

# Optimize database
netintel-ocr --db-optimize

# Export database
netintel-ocr --db-export output.json
netintel-ocr --db-export database.csv
```

### Infrastructure Commands

#### Project Initialization

```bash
# Initialize new project with Docker support
netintel-ocr --init

# Initialize with custom base directory
netintel-ocr --init --base-dir /path/to/project

# Force overwrite existing files
netintel-ocr --init --force
```

#### Configuration Management

```bash
# Use configuration file
netintel-ocr document.pdf --config config.yml

# Specify LanceDB paths
netintel-ocr --query "test" --lancedb-path /path/to/lancedb
netintel-ocr --query "test" --lancedb-uri s3://bucket/lancedb
```

### Advanced Options

#### Network Diagram Options

```bash
# Adjust detection confidence
netintel-ocr document.pdf --confidence 0.8

# Disable icons in Mermaid diagrams
netintel-ocr document.pdf --no-icons

# Extract diagram only (no text)
netintel-ocr document.pdf --diagram-only

# Custom timeout for complex diagrams
netintel-ocr document.pdf --timeout 120
```

#### Table Extraction Options

```bash
# Disable table extraction
netintel-ocr document.pdf --no-tables

# Adjust table detection confidence
netintel-ocr document.pdf --table-confidence 0.8

# Specify extraction method
netintel-ocr document.pdf --table-method hybrid
netintel-ocr document.pdf --table-method pdfplumber
netintel-ocr document.pdf --table-method llm

# Save tables as JSON
netintel-ocr document.pdf --save-table-json
```

#### Vector Generation Options

```bash
# Disable vector generation
netintel-ocr document.pdf --no-vector

# Vector-only mode (skip markdown)
netintel-ocr document.pdf --vector-only

# Specify vector format
netintel-ocr document.pdf --vector-format pinecone

# Configure chunking
netintel-ocr document.pdf --chunk-size 1500 --chunk-overlap 200

# Chunking strategy
netintel-ocr document.pdf --chunk-strategy semantic
netintel-ocr document.pdf --chunk-strategy fixed
netintel-ocr document.pdf --chunk-strategy sentence

# Include extended metadata
netintel-ocr document.pdf --embedding-metadata

# Regenerate vectors from existing markdown
netintel-ocr document.pdf --vector-regenerate
```

#### Cloud Storage Options

```bash
# Sync to S3/MinIO
netintel-ocr --s3-sync --endpoint http://minio:9000

# Backup to cloud
netintel-ocr --backup --bucket netintel-backups

# Restore from cloud
netintel-ocr --restore backup-2025-08-21.tar.gz
```

#### Debug and Monitoring

```bash
# Enable debug output
netintel-ocr document.pdf --debug

# Verbose output
netintel-ocr document.pdf --verbose

# Keep intermediate images
netintel-ocr document.pdf --keep-images

# Resize images for processing
netintel-ocr document.pdf --width 2000
```

## Configuration Examples

### Basic Configuration (YAML)

```yaml
processing:
  mode: hybrid
  parallel_workers: 4
  batch_size: 32
  timeouts:
    text_extraction: 120
    network_detection: 60

models:
  text:
    primary: nanonets-ocr-s:latest
    fallback: qwen2.5vl:latest
  network:
    primary: qwen2.5vl:latest
  embedding:
    model: nomic-embed-text
    provider: ollama

storage:
  output_dir: output
  cache_dir: .cache
  s3:
    enabled: true
    endpoint: http://minio:9000
    bucket: netintel-ocr

vector:
  enabled: true
  chunk_size: 1000
  chunk_overlap: 100
  lancedb:
    centralized: true
    centralized_path: output/lancedb
```

### Environment Variables

```bash
# Ollama configuration
export OLLAMA_HOST=http://localhost:11434

# Storage configuration
export NETINTEL_OUTPUT=/data/output
export MINIO_ENDPOINT=http://minio:9000
export AWS_ACCESS_KEY_ID=minioadmin
export AWS_SECRET_ACCESS_KEY=minioadmin

# Processing configuration
export NETINTEL_PARALLEL_WORKERS=8
export NETINTEL_AUTO_MERGE=true
export NETINTEL_DEDUPE=true

# Model configuration
export NETINTEL_EMBEDDING_MODEL=nomic-embed-text
export NETINTEL_EMBEDDING_PROVIDER=ollama
```

## Usage Patterns

### Pattern 1: Large-Scale Document Ingestion

```bash
# Step 1: Batch process all documents
netintel-ocr --batch-ingest --input-pattern "/documents/**/*.pdf" \
  --parallel 16 --auto-merge --dedupe

# Step 2: Optimize the database
netintel-ocr --db-optimize

# Step 3: Sync to cloud storage
netintel-ocr --s3-sync --endpoint $MINIO_ENDPOINT

# Step 4: Query the processed content
netintel-ocr --query "network security" --rerank --output-format json
```

### Pattern 2: Incremental Processing

```bash
# Process new documents daily
netintel-ocr --batch-ingest --input-pattern "/inbox/*.pdf" \
  --dedupe --auto-merge

# Check statistics
netintel-ocr --db-stats

# Export daily report
netintel-ocr --db-export "report-$(date +%Y%m%d).json"
```

### Pattern 3: Research and Analysis

```bash
# Query with advanced filtering
netintel-ocr --query "firewall rules" \
  --filter '{"document_type": "network_diagram", "confidence": {">": 0.8}}' \
  --rerank --query-limit 50 \
  --output-format markdown > firewall_analysis.md

# Export specific documents
netintel-ocr --query "VPN configuration" \
  --export-documents --output-dir ./vpn_docs
```

### Pattern 4: Development and Testing

```bash
# Initialize development environment
netintel-ocr --init --base-dir ./dev-env

# Process with debug output
netintel-ocr test.pdf --debug --keep-images

# Test different models
netintel-ocr sample.pdf --model llava:latest --network-model qwen2.5vl:latest

# Regenerate vectors with different settings
netintel-ocr sample.pdf --vector-regenerate \
  --chunk-size 500 --chunk-strategy sentence
```

## Performance Tuning

### Processing Speed Optimization

```bash
# Fast text-only processing
netintel-ocr document.pdf --text-only --parallel 8

# Optimized network extraction
netintel-ocr document.pdf --fast-extraction --timeout 30

# Skip unnecessary features
netintel-ocr document.pdf --no-tables --no-vector
```

### Memory Management

```bash
# Reduce image size for lower memory usage
netintel-ocr large.pdf --width 1500

# Process in smaller batches
netintel-ocr --batch-ingest --input-pattern "*.pdf" \
  --parallel 2 --batch-size 16
```

### Storage Optimization

```bash
# Clean up after processing
netintel-ocr document.pdf --no-keep-images

# Compress vector storage
netintel-ocr --db-optimize --compact

# Use cloud storage for archives
netintel-ocr --archive-to-s3 --older-than 30
```

## Deployment Scenarios

### Local Development

```bash
# Simple local processing
netintel-ocr document.pdf

# With local configuration
netintel-ocr document.pdf --config ./config.yml
```

### Docker Deployment

```bash
# Initialize Docker environment
netintel-ocr --init

# Build and run with Docker Compose
cd netintel-ocr
docker-compose up -d

# Process documents in container
docker exec netintel-ocr netintel-ocr /input/document.pdf
```

### Kubernetes Deployment

```bash
# Deploy with Helm
helm install netintel-ocr ./helm

# Scale workers
kubectl scale deployment netintel-ocr --replicas=5

# Monitor pods
kubectl get pods -n netintel-ocr
```

### Cloud Deployment

```bash
# Configure S3 backend
export AWS_ACCESS_KEY_ID=your_key
export AWS_SECRET_ACCESS_KEY=your_secret

# Process with cloud storage
netintel-ocr document.pdf \
  --lancedb-uri s3://your-bucket/lancedb \
  --s3-sync
```

## Troubleshooting Guide

### Common Issues and Solutions

#### Issue: Ollama Connection Failed

```bash
# Check Ollama status
curl http://localhost:11434/api/tags

# Specify custom Ollama host
export OLLAMA_HOST=http://your-server:11434
netintel-ocr document.pdf
```

#### Issue: Out of Memory

```bash
# Reduce parallel workers
netintel-ocr --batch-ingest --parallel 2

# Reduce image size
netintel-ocr document.pdf --width 1000
```

#### Issue: Slow Processing

```bash
# Use text-only mode for documents without diagrams
netintel-ocr document.pdf --text-only

# Enable fast extraction
netintel-ocr document.pdf --fast-extraction

# Use lighter models
netintel-ocr document.pdf --model tinyllama:latest
```

#### Issue: Database Corruption

```bash
# Validate database
netintel-ocr --db-validate

# Rebuild from source documents
netintel-ocr --db-rebuild

# Restore from backup
netintel-ocr --restore last-known-good.tar.gz
```

## Best Practices

### Document Organization

1. **Use consistent naming**: Organize input PDFs in logical directories
2. **Leverage deduplication**: Enable --dedupe to avoid reprocessing
3. **Regular maintenance**: Run --db-optimize weekly for large deployments
4. **Backup strategy**: Regular backups to cloud storage

### Performance Optimization

1. **Model selection**: Use appropriate models for content type
2. **Parallel processing**: Scale workers based on available resources
3. **Caching strategy**: Enable embedding cache for repeated queries
4. **Batch processing**: Process multiple documents together

### Security Considerations

1. **Access control**: Secure Ollama and MinIO endpoints
2. **Data encryption**: Enable encryption for cloud storage
3. **Credential management**: Use environment variables or secrets management
4. **Network isolation**: Deploy in isolated network segments

### Monitoring and Maintenance

1. **Track metrics**: Monitor processing times and success rates
2. **Log analysis**: Review logs for errors and warnings
3. **Resource monitoring**: Track CPU, memory, and storage usage
4. **Regular updates**: Keep models and dependencies updated

## Summary

NetIntel-OCR provides a comprehensive solution for intelligent document processing with advanced features for network diagram extraction, table processing, and vector search capabilities. The modular architecture supports various deployment scenarios from local development to cloud-scale production environments. The extensive command-line interface offers fine-grained control over all aspects of the system, while the configuration system provides flexibility for different use cases and requirements.