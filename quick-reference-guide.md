# NetIntel-OCR Quick Reference (v0.1.12)

**Note**: Default mode is now hybrid with automatic network diagram detection. Vector generation is enabled by default.

## Quick Start

### Basic Usage
```bash
# Process entire PDF with automatic detection (default)
netintel-ocr document.pdf
# Creates: output/<md5_checksum>/
# Includes: Text, diagrams, tables, vectors

# Process specific pages
netintel-ocr --start 5 --end 10 document.pdf

# Resume interrupted processing
netintel-ocr document.pdf --resume

# Keep intermediate images
netintel-ocr --keep-images document.pdf
```

### Multi-Model Support (v0.1.4+)
```bash
# Use different models for text and network processing
netintel-ocr document.pdf --model nanonets-ocr-s --network-model qwen2.5vl

# Fast processing with optimized models
netintel-ocr document.pdf --model moondream --network-model bakllava --fast-extraction

# Heavy processing for complex diagrams
netintel-ocr document.pdf --network-model llava:34b --timeout 120
```

## Processing Modes

### Default Hybrid Mode
```bash
# Automatic detection with icons (default)
netintel-ocr document.pdf

# Disable icons
netintel-ocr document.pdf --no-icons

# Extract only diagrams without text
netintel-ocr document.pdf --diagram-only
```

### Specialized Modes
```bash
# Text only (fastest, skip detection)
netintel-ocr --text-only document.pdf

# Network diagrams only
netintel-ocr --network-only document.pdf

# Fast extraction (50-70% faster)
netintel-ocr --fast-extraction document.pdf

# Force multi-diagram extraction
netintel-ocr --multi-diagram document.pdf
```

### Table Extraction (v0.1.6+)
```bash
# Automatic table detection (enabled by default)
netintel-ocr document.pdf

# Control table extraction method
netintel-ocr document.pdf --table-method hybrid  # library, llm, or hybrid

# Skip table extraction
netintel-ocr document.pdf --no-tables
```

## Advanced Features (v0.1.12)

### Centralized Database Management
```bash
# Merge documents into centralized database
netintel-ocr --merge-to-centralized ./output ./centralized.lancedb

# Query centralized database
netintel-ocr --query "network security architecture" \
  --centralized-db ./centralized.lancedb \
  --filters '{"content_type": "diagram", "confidence": {"$gte": 0.8}}' \
  --rerank-strategy hybrid \
  --limit 20

# Database statistics
netintel-ocr --db-stats ./centralized.lancedb --format json

# Optimize database
netintel-ocr --db-optimize ./centralized.lancedb --vacuum --reindex
```

### Batch Processing
```bash
# Process multiple PDFs in parallel
netintel-ocr --batch-ingest ./documents \
  --output ./batch_output \
  --parallel-workers 8 \
  --auto-merge

# Batch with S3 sync
netintel-ocr --batch-ingest ./documents \
  --s3-sync \
  --s3-bucket my-bucket \
  --s3-endpoint https://minio.example.com
```

### S3/MinIO Integration
```bash
# Sync to S3/MinIO storage
netintel-ocr document.pdf --s3-sync \
  --s3-bucket netintel-docs \
  --s3-endpoint http://localhost:9000

# Download from S3 and process
netintel-ocr --s3-download s3://my-bucket/document.pdf \
  --s3-endpoint https://minio.example.com

# Backup to S3
netintel-ocr --s3-backup ./output \
  --s3-bucket backup-bucket \
  --retention-days 30
```

## Vector Database Integration (v0.1.7+)

### Vector Generation (Enabled by Default)
```bash
# Default: Generates vector files automatically
netintel-ocr document.pdf

# Disable vector generation
netintel-ocr document.pdf --no-vector

# Custom chunking settings
netintel-ocr document.pdf \
  --chunk-size 500 \
  --chunk-overlap 50 \
  --chunk-strategy semantic

# Different vector format
netintel-ocr document.pdf --vector-format pinecone
```

### Vector Regeneration (v0.1.10+)
```bash
# Regenerate vectors with new settings
netintel-ocr document.pdf --vector-regenerate \
  --chunk-size 750 \
  --chunk-strategy sentence

# Change vector format without reprocessing PDF
netintel-ocr document.pdf --vector-regenerate \
  --vector-format weaviate
```

## Infrastructure & Deployment (v0.1.11+)

### Project Initialization
```bash
# Initialize complete project structure
netintel-ocr --init ./my-project

# With custom configuration
netintel-ocr --init ./my-project \
  --with-docker \
  --with-helm \
  --with-minio
```

### Configuration Management
```bash
# Use YAML configuration
netintel-ocr --config config.yml document.pdf

# Override with environment variables
export NETINTEL_OCR_MODEL="qwen2.5vl"
export NETINTEL_OCR_PARALLEL_WORKERS=16
netintel-ocr --config config.yml document.pdf
```

### Docker Operations
```bash
# Build Docker image
docker build -t netintel-ocr .

# Run with Docker Compose
docker-compose up -d

# Process with containerized version
docker run -v $(pwd)/input:/data/input \
  -v $(pwd)/output:/data/output \
  netintel-ocr document.pdf
```

### Kubernetes Deployment
```bash
# Deploy with Helm
helm install netintel-ocr ./helm/netintel-ocr

# Upgrade deployment
helm upgrade netintel-ocr ./helm/netintel-ocr \
  --set image.tag=v0.1.12

# Scale workers
kubectl scale deployment netintel-ocr --replicas=5
```

## Command Line Options Reference

### Core Options
| Option | Short | Description | Default |
|--------|-------|-------------|---------|
| `--output` | `-o` | Base output directory | `output` |
| `--model` | `-m` | Text extraction model | `nanonets-ocr-s` |
| `--network-model` | | Network processing model | `qwen2.5vl` |
| `--keep-images` | `-k` | Keep intermediate images | `False` |
| `--width` | `-w` | Resize images to width | `0` (no resize) |
| `--start` | `-s` | Start page number | `1` |
| `--end` | `-e` | End page number | Last page |
| `--resume` | | Resume from checkpoint | `False` |

### Processing Modes
| Option | Description | Default |
|--------|-------------|---------|
| `--text-only` | Skip network detection | `False` |
| `--network-only` | Process diagrams only | `False` |
| `--fast-extraction` | Use optimized prompts | `False` |
| `--multi-diagram` | Force multi-diagram mode | `False` |
| `--diagram-only` | Extract diagram without text | `False` |

### Network & Table Options
| Option | Short | Description | Default |
|--------|-------|-------------|---------|
| `--confidence` | `-c` | Detection confidence | `0.7` |
| `--timeout` | | LLM timeout (seconds) | `60` |
| `--no-icons` | | Disable Mermaid icons | `False` |
| `--table-method` | | Table extraction method | `hybrid` |
| `--no-tables` | | Skip table extraction | `False` |

### Vector Database Options
| Option | Description | Default |
|--------|-------------|---------|
| `--no-vector` | Disable vector generation | `False` |
| `--vector-format` | Target database format | `lancedb` |
| `--chunk-size` | Tokens per chunk | `1000` |
| `--chunk-overlap` | Token overlap | `100` |
| `--chunk-strategy` | Chunking strategy | `semantic` |
| `--embedding-metadata` | Include extended metadata | `False` |
| `--vector-regenerate` | Regenerate vectors from markdown | `False` |

### Advanced Options (v0.1.12)
| Option | Description |
|--------|-------------|
| `--batch-ingest` | Process directory of PDFs |
| `--parallel-workers` | Number of parallel workers (1-16) |
| `--auto-merge` | Auto-merge to centralized DB |
| `--merge-to-centralized` | Merge databases |
| `--query` | Query centralized database |
| `--filters` | JSON query filters |
| `--rerank-strategy` | Reranking strategy |
| `--db-stats` | Show database statistics |
| `--db-optimize` | Optimize database |
| `--s3-sync` | Sync to S3/MinIO |
| `--s3-bucket` | S3 bucket name |
| `--s3-endpoint` | S3 endpoint URL |

### Output Control
| Option | Short | Description | Default |
|--------|-------|-------------|---------|
| `--verbose` | `-v` | Show progress information | `False` |
| `--debug` | `-d` | Show detailed processing | `False` |

## Output Structure (v0.1.4+)

```
output/
├── index.md                              # Master index of all documents
├── <md5_checksum>/                       # Unique folder per document
│   ├── markdown/
│   │   ├── page_001.md                  # Individual pages
│   │   ├── page_002.md
│   │   ├── document.md                  # Merged with metrics
│   │   └── document-vector.md           # Vector-optimized
│   ├── lancedb/                         # Vector DB files
│   │   ├── chunks.jsonl                 # Pre-chunked content
│   │   ├── metadata.json                # Document metadata
│   │   └── schema.json                  # LanceDB schema
│   ├── tables/                          # Extracted tables
│   │   ├── table_page_001.json
│   │   └── tables_flat.jsonl           # Flattened for vector DB
│   ├── images/                          # If --keep-images
│   ├── .checkpoint/                     # Resume data
│   └── summary.md                       # Processing summary
└── centralized.lancedb/                 # Unified database (v0.1.12)
```

## Environment Variables

```bash
# Ollama configuration
export OLLAMA_HOST="http://localhost:11434"

# Default models
export NETINTEL_OCR_MODEL="nanonets-ocr-s"
export NETINTEL_OCR_NETWORK_MODEL="qwen2.5vl"

# Processing settings
export NETINTEL_OCR_PARALLEL_WORKERS=8
export NETINTEL_OCR_CHUNK_SIZE=1000

# S3 configuration
export AWS_ACCESS_KEY_ID="your-key"
export AWS_SECRET_ACCESS_KEY="your-secret"
export S3_ENDPOINT_URL="https://minio.example.com"
```

## Examples by Use Case

### Fast Text Extraction
```bash
netintel-ocr --text-only --model nanonets-ocr-s document.pdf
```

### High-Quality Network Diagram Extraction
```bash
netintel-ocr --network-model cogvlm --timeout 120 --confidence 0.9 document.pdf
```

### Process Large Document with Resume
```bash
# Initial run (may be interrupted)
netintel-ocr large-document.pdf

# Resume from checkpoint
netintel-ocr large-document.pdf --resume
```

### Batch Process with Database
```bash
# Process all PDFs and create centralized DB
netintel-ocr --batch-ingest ./pdfs \
  --parallel-workers 8 \
  --auto-merge \
  --output ./processed

# Query the database
netintel-ocr --query "firewall configuration" \
  --centralized-db ./processed/centralized.lancedb \
  --limit 10
```

### Extract and Regenerate Vectors
```bash
# Initial processing
netintel-ocr document.pdf

# Later: regenerate with different settings
netintel-ocr document.pdf --vector-regenerate \
  --chunk-size 500 \
  --vector-format pinecone
```

## Performance Optimization Tips

### Model Selection for Speed
```bash
# Fastest combination
netintel-ocr --model nanonets-ocr-s \
  --network-model bakllava \
  --fast-extraction \
  --timeout 30 document.pdf
```

### Model Selection for Accuracy
```bash
# Most accurate combination
netintel-ocr --model qwen2.5vl \
  --network-model cogvlm \
  --timeout 120 \
  --confidence 0.9 document.pdf
```

### Resource-Constrained Environment
```bash
# Minimal resource usage
netintel-ocr --model llava-phi3 \
  --network-model llava-phi3 \
  --width 1024 \
  --chunk-size 500 document.pdf
```

### Large-Scale Processing
```bash
# Optimize for many documents
netintel-ocr --batch-ingest ./documents \
  --parallel-workers 16 \
  --fast-extraction \
  --auto-merge \
  --checkpoint-interval 10
```

## Troubleshooting Commands

### Check Ollama Connection
```bash
curl http://localhost:11434/api/tags
```

### List Available Models
```bash
ollama list
```

### Test Minimal Processing
```bash
netintel-ocr --text-only --start 1 --end 1 test.pdf --debug
```

### Verify Database Health
```bash
netintel-ocr --db-stats ./centralized.lancedb --include-performance-metrics
```

### Debug Vector Generation
```bash
netintel-ocr document.pdf --debug --chunk-size 100 --keep-images
# Check lancedb/chunks.jsonl for output
```

## Version History Highlights

- **v0.1.12**: Centralized DB, advanced query engine, batch processing, S3 integration
- **v0.1.11**: Docker/Kubernetes support, configuration management, project init
- **v0.1.10**: Vector regeneration, smart ToC handling
- **v0.1.7**: Automatic vector generation for RAG
- **v0.1.6**: Table extraction with library and LLM methods
- **v0.1.5**: Checkpoint/resume for long documents
- **v0.1.4**: Multi-model support, MD5-based output structure

## Quick Performance Reference

| Mode | Speed | Use Case |
|------|-------|----------|
| Text-only | 15-30 sec/page | Documents without diagrams |
| Fast extraction | 10-20 sec/page | Quick processing |
| Default hybrid | 30-60 sec/page | Mixed content |
| Network-only | 30-60 sec/diagram | Architecture docs |
| Heavy models | 60-90 sec/page | Complex diagrams |
| Batch (8 workers) | ~5 sec/page/worker | Large collections |

## Model Recommendations

| Task | Recommended | Alternative | Fast Option |
|------|-------------|-------------|-------------|
| Text OCR | nanonets-ocr-s | moondream | llava-phi3 |
| Network Detection | qwen2.5vl | llava | bakllava |
| Complex Diagrams | cogvlm | llava:34b | qwen2.5vl |
| Tables | qwen2.5vl | llava | (uses pdfplumber) |