# NetIntel-OCR Architecture Documentation v0.1.12

## Overview

NetIntel-OCR is a specialized Python-based tool that intelligently processes PDF documents, automatically detecting and converting network diagrams into Mermaid.js format while preserving text content. It uses multimodal AI models via Ollama to process PDFs by converting pages to images and then using vision-language models to transcribe content and identify network-specific diagrams for enhanced extraction. 

**Version 0.1.12 delivers the complete centralized database architecture with advanced query engine, parallel batch processing, S3/MinIO cloud storage integration, and comprehensive embedding management system.**

## System Architecture

### Component Diagram (v0.1.12)

```
┌─────────────────┐
│   CLI Entry     │
│   (cli.py)      │
│ --init          │ ← Project initialization (v0.1.11)
│ --config        │ ← YAML configuration (v0.1.11)
│ --query         │ ← Advanced Query Engine (v0.1.12) 🆕
│ --merge-to-centralized │ ← Centralized Database (v0.1.12) 🆕
│ --batch-ingest  │ ← Parallel Batch Processing (v0.1.12) 🆕
│ --db-stats      │ ← Database Management (v0.1.12) 🆕
│ --s3-sync       │ ← Cloud Storage Integration (v0.1.12) 🆕
│ --network-model │ ← Multi-model support
│ --table-method  │ ← Table extraction (v0.1.6)
│ --vector-regenerate │ ← Vector regeneration (v0.1.10)
└────────┬────────┘
         │
    ┌────┴────────────────────────────────────────────┬─────────────────┬─────────────┐
    ▼                                                 ▼                 ▼             ▼
┌─────────────┐ ┌──────────────────┐ ┌─────────────────────┐ ┌─────────────────────┐ ┌──────────────┐
│ Single Doc  │ │  Batch Processor │ │   Query Engine      │ │ Centralized DB Mgr  │ │ S3 Storage   │
│ Processing  │ │   (v0.1.12) 🆕   │ │    (v0.1.12) 🆕     │ │    (v0.1.12) 🆕     │ │(v0.1.12) 🆕  │
└─────┬───────┘ └─────────┬────────┘ └─────────┬───────────┘ └─────────┬───────────┘ └──────┬───────┘
      │                   │                    │                       │                    │
      ▼                   ▼                    ▼                       ▼                    ▼
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                Processing Pipeline                                                 │
├──────────────┬─────────────┬─────────────┬─────────────┬─────────────┬─────────────┬─────────────┤
│ Text Model   │Network Model│Table Extract│Vector Gen   │Query Engine │Embedding    │Storage Sync │
│(nanonets-ocr)│(qwen2.5vl)  │(pdfplumber) │(Enhanced)   │(Advanced)   │Management   │(S3/MinIO)   │
└──────┬───────┴─────┬───────┴─────┬───────┴─────┬───────┴─────┬───────┴─────┬───────┴─────┬───────┘
       │             │             │             │             │             │             │
    ┌──┴──┬──────────┴──┬──────────┴──┬──────────┴──┬──────────┴──┬──────────┴──┬──────────┴──┐
    ▼     ▼             ▼             ▼             ▼             ▼             ▼             ▼
┌────────┐┌──────────┐┌──────────┐┌──────────┐┌──────────┐┌──────────┐┌──────────┐┌──────────┐
│  PDF   ││ Network  ││ Table    ││ Vector   ││Centralized││ Query    ││Embedding ││ Cloud    │
│Handler ││ Diagram  ││ Extract  ││Generator ││Database   ││ Filter & ││Provider  ││Storage   │
│        ││ Module   ││ Module   ││ Module   ││ Manager   ││ Ranker   ││& Cache   ││Sync      │
└────────┘└──────────┘└──────────┘└──────────┘└──────────┘└──────────┘└──────────┘└──────────┘
             │           │           │           │           │           │           │
         ┌───┴───┐   ┌───┴───┐   ┌───┴───┐   ┌───┴───┐   ┌───┴───┐   ┌───┴───┐   ┌───┴───┐
         ▼       ▼   ▼       ▼   ▼       ▼   ▼       ▼   ▼       ▼   ▼       ▼   ▼       ▼
    ┌─────────┐┌──────┐┌──────┐┌──────┐┌──────┐┌──────┐┌──────┐┌──────┐┌──────┐┌──────┐
    │Detector ││Chunk ││Table ││Dedup ││Merge ││Filter││OpenAI││Cache ││Upload││Backup│
    │Extractor││ Gen  ││Valid ││Logic ││Index ││Multi ││Ollama││& TTL ││S3    ││& Ver │
    └─────────┘└──────┘└──────┘└──────┘└──────┘└──────┘└──────┘└──────┘└──────┘└──────┘
```

### Enhanced Data Flow (v0.1.12)

#### Single Document Processing
```
PDF File → MD5 Checksum → Create output/<md5>/ folder →
→ PDF to Images → Model Selection (Text + Network) →
→ Detection Phase (Network & Tables) →
├── [Network Diagram] → Component Extraction → Mermaid Generation → Validation
├── [Table Detected] → Library/LLM Extraction → JSON Generation → Validation
└── [Text Only] → Standard Transcription
→ Individual Markdown Files → Merged Document with Footer Metrics →
→ Enhanced Vector Generation (v0.1.12) →
   ├── Multi-Provider Embedding (OpenAI/Ollama/HF)
   ├── Intelligent Caching with TTL
   ├── Content Filtering (remove artifacts)
   ├── Create document-vector.md
   ├── Generate chunks.jsonl (LanceDB-ready)
   └── Create metadata & schema files
→ Optional: Auto-merge to Centralized Database
→ Optional: S3/MinIO Cloud Sync
→ Update index.md with document entry
```

#### Batch Processing Pipeline (NEW v0.1.12)
```
Batch Directory/S3 → Document Discovery & Validation →
→ Parallel Worker Pool (1-16 workers) →
   ├── Worker 1: PDF Processing Pipeline
   ├── Worker 2: PDF Processing Pipeline  
   ├── Worker N: PDF Processing Pipeline
   └── Progress Coordinator (ETA, Throughput)
→ Checkpoint Management (Resume on Interruption) →
→ Results Aggregation →
→ Auto-merge to Centralized Database →
→ Cloud Storage Sync (if enabled) →
→ Comprehensive Batch Report
```

#### Centralized Database Architecture (NEW v0.1.12)
```
Per-Document Databases → Document Validation →
→ MD5 Deduplication Check →
→ Schema Migration (if needed) →
→ Unified LanceDB Creation →
   ├── Document Metadata Table
   ├── Chunks Vector Table  
   ├── Tables Data Table
   └── Ingestion Log Table
→ Index Optimization →
→ Health Validation →
→ Query Engine Ready
```

#### Advanced Query Flow (NEW v0.1.12)
```
Query Input → Query Parser & Validation →
→ Vector Similarity Search →
→ Multi-field Filtering →
   ├── Source File Filters
   ├── Document Type Filters
   ├── Confidence Score Filters
   ├── Date Range Filters
   └── Custom JSON Filters
→ Reranking Pipeline →
   ├── Semantic Reranking
   ├── Hybrid Score Reranking
   └── Temporal Reranking
→ Result Formatting →
   ├── JSON Output
   ├── Markdown Output
   └── CSV Output
→ Pagination & Limiting →
→ Response Delivery
```

### Output Directory Structure (v0.1.10)

```
output/                                    # Base directory (configurable)
├── index.md                              # Master index of all documents
├── <md5_checksum_1>/                     # Unique folder per document
│   ├── images/                           # Extracted images
│   ├── markdown/                         # Markdown files
│   │   ├── page_001.md
│   │   ├── page_002.md
│   │   ├── document.md                   # Merged with footer metrics
│   │   └── document-vector.md            # Vector-optimized (AUTO-CREATED v0.1.7)
│   ├── lancedb/                          # Vector DB files (AUTO-CREATED v0.1.7)
│   │   ├── chunks.jsonl                  # Pre-chunked content
│   │   ├── metadata.json                 # Document metadata
│   │   ├── schema.json                   # LanceDB schema
│   │   ├── embeddings_config.json        # Embedding config
│   │   └── index_config.json             # Index config
│   ├── tables/                           # Table files
│   │   ├── table_page_001.json          # Original tables
│   │   └── tables_flat.jsonl            # Flattened for vector DB (v0.1.7)
│   ├── .checkpoint/                      # Checkpoint data (if interrupted)
│   ├── .vector_cache/                    # Vector processing cache (v0.1.7)
│   └── summary.md
└── <md5_checksum_2>/                     # Another document
    └── ...
```

## v0.1.12 Advanced Components

### 1. Centralized Database Manager (`centralized_database/`)
- **Purpose**: Unified database management with deduplication and optimization
- **Components**:
  - `manager.py`: Core database operations, schema management, health monitoring
  - `merger.py`: Document merging logic with MD5-based deduplication
  - `validator.py`: Schema validation, integrity checking, data consistency
  - `optimizer.py`: Performance optimization, vacuum operations, index tuning
- **Responsibilities**:
  - Merge per-document databases into unified LanceDB
  - Prevent duplicate ingestion using MD5 checksums
  - Maintain document metadata and ingestion tracking
  - Optimize database performance and storage efficiency
  - Provide comprehensive database statistics and health metrics

### 2. Advanced Query Engine (`query_engine/`)
- **Purpose**: Sophisticated vector search with filtering and reranking
- **Components**:
  - `engine.py`: Core query execution with vector similarity search
  - `filters.py`: Multi-field filtering (source, type, confidence, dates)
  - `rankers.py`: Multiple reranking strategies (semantic, hybrid, temporal)
  - `exporters.py`: Output format generation (JSON, Markdown, CSV)
- **Responsibilities**:
  - Execute vector similarity queries with configurable distance metrics
  - Apply complex filtering logic with JSON-based filter expressions
  - Rerank results using multiple strategies for better relevance
  - Export results in multiple formats with customizable field selection
  - Handle pagination and result limiting for large datasets

### 3. Batch Processing System (`batch_processor/`)
- **Purpose**: Parallel processing of multiple PDFs with progress tracking
- **Components**:
  - `processor.py`: Main coordinator managing worker pool and job distribution
  - `worker.py`: Individual worker process for PDF processing
  - `progress.py`: Real-time progress tracking with ETA and throughput
  - `checkpoints.py`: Checkpoint management for resumable batch operations
- **Responsibilities**:
  - Distribute PDF processing across multiple parallel workers (1-16)
  - Track progress with real-time ETA calculation and throughput metrics
  - Manage checkpoints for resuming interrupted batch operations
  - Handle memory management and resource allocation
  - Aggregate results and auto-merge to centralized database

### 4. S3/MinIO Storage Backend (`storage_backends/`)
- **Purpose**: Cloud storage integration with bi-directional sync
- **Components**:
  - `s3_manager.py`: S3-compatible storage operations (AWS S3, MinIO, etc.)
  - `sync.py`: Bi-directional synchronization with conflict resolution
  - `backup.py`: Backup and restore operations with versioning
- **Responsibilities**:
  - Upload/download files to/from S3-compatible storage
  - Synchronize local and remote storage with versioning
  - Create and restore backups with retention policies
  - Manage credentials and endpoint configuration
  - Support distributed access for multiple workers

### 5. Enhanced Embedding Management (`embedding_management/`)
- **Purpose**: Multi-provider embedding generation with intelligent caching
- **Components**:
  - `providers/`: Provider-specific implementations (OpenAI, Ollama, HuggingFace)
  - `cache.py`: Intelligent caching with TTL and hit/miss tracking
  - `batch.py`: Batch processing optimization for embedding generation
- **Responsibilities**:
  - Support multiple embedding providers with automatic fallback
  - Implement intelligent caching with TTL to reduce API costs
  - Optimize embedding generation through batch processing
  - Handle rate limiting and backoff for API compliance
  - Provide embedding performance metrics and cost tracking

## v0.1.11 Infrastructure Components

### 1. Project Initializer (`project_initializer.py`)
- **Purpose**: Create complete containerized project structure
- **Responsibilities**:
  - Generate Docker files (Dockerfile, docker-compose.yml)
  - Create Helm charts for Kubernetes deployment
  - Generate configuration files (config.yml, .env)
  - Setup directory structure for containers
  - Provide MinIO integration setup

### 2. Configuration Manager (`config_manager.py`)
- **Purpose**: Handle YAML configuration and environment variables
- **Responsibilities**:
  - Load and parse YAML configuration files
  - Apply environment variable overrides
  - Merge CLI arguments with configuration
  - Validate configuration completeness
  - Provide configuration defaults

### 3. Query Interface (`query_interface.py`)
- **Purpose**: Foundation for vector database queries and centralized DB
- **Classes**:
  - `QueryInterface`: Query vector databases (foundation mode)
  - `CentralizedDatabaseManager`: Merge per-document databases
- **Responsibilities**:
  - Query local or remote LanceDB (foundation)
  - List and validate documents
  - Track ingestion for deduplication
  - Format query results (JSON/Markdown/CSV)

### 4. Docker Infrastructure
- **Dockerfile**: Production-ready container with all dependencies
- **docker-compose.yml**: Multi-service orchestration with MinIO
- **MinIO Integration**: S3-compatible storage for distributed deployments
- **Health Checks**: Ollama connectivity verification

### 5. Kubernetes Support
- **Helm Chart**: Complete deployment templates
- **Components**:
  - Deployment for NetIntel-OCR service
  - StatefulSet for MinIO storage
  - ConfigMaps for configuration
  - PersistentVolumeClaims for data
  - Ingress for external access
  - Autoscaling support (HPA)

## Core Components

### 1. CLI Interface (`cli.py`)
- **Purpose**: Command-line interface with multiple processing modes
- **Responsibilities**:
  - Parse command-line arguments
  - Validate user inputs
  - Select appropriate processor (hybrid/network/text-only)
  - Handle output directory naming with timestamps
  - Configure network detection parameters

### 2. Processing Engines

#### 2.1 Standard Processor (`processor.py`)
- **Purpose**: Basic PDF text extraction
- **Responsibilities**:
  - Validate PDF file existence
  - Check Ollama server connectivity
  - Coordinate PDF conversion and transcription
  - Handle error states and user feedback
  - Manage cleanup of temporary files

#### 2.2 Hybrid Processor (`hybrid_processor.py`)
- **Purpose**: Automatic network diagram detection with text extraction
- **DEFAULT MODE**: Processes all pages with intelligent detection
- **Responsibilities**:
  - Text-first processing (guarantees content capture)
  - Automatic network diagram detection per page
  - Falls back gracefully on timeout or detection failure
  - Combines diagram and text content in output
  - Progress tracking with detailed messages

#### 2.3 Network Processor (`network_processor.py`)
- **Purpose**: Dedicated network diagram extraction
- **Responsibilities**:
  - Focus exclusively on network diagram pages
  - Skip non-diagram content for efficiency
  - Generate Mermaid diagrams with icons
  - Component and connection extraction

#### 2.4 Optimized Processor (`optimized_processor.py`)
- **Purpose**: Fast extraction mode for performance
- **Responsibilities**:
  - 50-70% faster processing using simplified prompts
  - Reduced timeouts (15s detection, 20s extraction)
  - Automatic fallback to rule-based generation
  - Minimal token usage for faster responses

### 3. PDF Handler (`pdf.py`)
- **Purpose**: Convert PDF pages to images
- **Technology**: PyMuPDF (pymupdf)
- **Responsibilities**:
  - Open and parse PDF documents
  - Convert specified page ranges to PNG images
  - Handle page number validation
  - Progress tracking during conversion

### 4. Ollama Client (`ollama.py`)
- **Purpose**: Interface with Ollama API for AI transcription
- **Responsibilities**:
  - Check server availability
  - Encode images to base64
  - Send transcription requests to Ollama
  - Handle API responses and errors

### 5. Utilities (`utils.py`)
- **Purpose**: Common utility functions
- **Responsibilities**:
  - Create and manage output directory structure
  - Resize images to optimize for AI processing
  - Merge individual text files into a single document

### 6. Constants (`constants.py`)
- **Purpose**: Centralized configuration
- **Responsibilities**:
  - Store Ollama API endpoints
  - Define transcription prompts
  - Support environment variable configuration (OLLAMA_HOST)

### 7. Network Diagram Module (`network_diagram/`)
- **Purpose**: Specialized network diagram processing
- **Components**:
  - **Detector** (`detector.py`, `fast_detector.py`): Identifies network diagrams
  - **Extractor** (`extractor.py`, `fast_extractor.py`, `improved_extractor.py`): Extracts components and connections
  - **Generator** (`mermaid_generator.py`): Creates Mermaid.js diagrams
  - **Validators** (`validator.py`, `robust_validator.py`, `smart_validator.py`): Ensures valid Mermaid syntax
  - **Icons** (`icons.py`): Font Awesome icon mappings
  - **Prompts** (`prompts.py`, `improved_prompts.py`): LLM prompt templates

### 8. Support Utilities
- **Timeout Utils** (`timeout_utils.py`): Graceful timeout handling
- **Output Utils** (`output_utils.py`): Progress messages and debugging
- **PDF Utils** (`pdf_utils.py`): Page count checking and validation

### 9. Multi-Model Architecture (v0.1.4)
- **Purpose**: Optimize processing with task-specific models
- **Components**:
  - **Model Selection**: Automatic selection based on task type
  - **Text Model**: Default `nanonets-ocr-s` for fast OCR
  - **Network Model**: Default `qwen2.5vl` for diagram understanding
  - **Fallback Chains**: Automatic fallback to alternative models
- **Benefits**:
  - 30-50% faster text extraction with OCR-optimized models
  - Better diagram understanding with vision-language models
  - Resource efficiency by using appropriate model sizes
  - Flexibility to experiment with different combinations

### 10. Enhanced Utilities (v0.1.4)
- **MD5 Checksum** (`utils.py`): Document identification and folder naming
- **Index Management** (`utils.py`): Automatic index.md creation and updates
- **Metrics System** (`utils.py`): Comprehensive footer with processing details
- **Directory Management**: MD5-based unique folders per document

### 11. Table Extraction Module (v0.1.6)
- **Purpose**: Detect and extract tables from PDF documents
- **Components**:
  - **TableDetector** (`detector.py`): Identifies tables in PDF pages
  - **LibraryTableExtractor** (`library_extractor.py`): Uses pdfplumber for fast extraction
  - **LLMTableExtractor** (`llm_extractor.py`): Vision model-based extraction for complex tables
  - **TableValidator** (`validator.py`): Validates and auto-corrects table data
  - **TableJSONGenerator** (`json_generator.py`): Generates JSON and markdown output
- **Features**:
  - Automatic detection alongside network diagrams
  - Multiple extraction methods (hybrid/library/LLM)
  - Complex table support (merged cells, multi-row fields)
  - Type inference and validation
  - JSON and markdown output formats

### 12. Checkpoint/Resume System (v0.1.5)
- **Purpose**: Enable resumable processing for long documents
- **Components**:
  - **CheckpointManager** (`checkpoint.py`): Manages checkpoint files and directories
  - **ProcessingState** (`checkpoint.py`): Tracks processing progress
  - **Page-Level Tracking**: Individual page completion status
  - **Atomic Saves**: Prevents checkpoint corruption
- **Features**:
  - Automatic checkpoint creation during processing
  - Resume from exact page where interrupted
  - Smart skip of already processed pages
  - Processing metrics preservation
  - Automatic cleanup on completion
- **Storage**: Checkpoints stored in `output/<md5>/.checkpoint/`

## Current Capabilities

### Core Features
1. **Intelligent PDF Processing**
   - **Hybrid Mode (DEFAULT)**: Automatic network diagram detection
   - **Text-First Architecture**: Guarantees content capture before diagram processing
   - Full document or page range processing (max 100 pages per run)
   - Preserves formatting using Markdown syntax

2. **Network Diagram Detection & Extraction**
   - **Automatic Detection**: Identifies network diagrams with confidence scoring
   - **Component Recognition**: 9+ network device types (routers, switches, firewalls, servers, etc.)
   - **Connection Mapping**: 5 connection types (ethernet, wireless, VPN, data flow, redundant)
   - **Zone Detection**: Identifies DMZ, trusted/untrusted zones, VLANs, subnets
   - **Protocol Recognition**: Extracts protocol and port information

3. **Mermaid.js Generation**
   - **Automatic Conversion**: Network diagrams to Mermaid format
   - **Icon Support**: Font Awesome icons enabled by default
   - **Syntax Validation**: 4-phase validation and auto-correction
   - **Multiple Layouts**: TB, LR, BT, RL based on diagram structure
   - **Styling Classes**: Different colors for component types

4. **Performance Modes**
   - **Standard Mode**: 30-60 seconds per page with full detection
   - **Fast Extraction**: 50-70% faster using optimized prompts
   - **Text-Only Mode**: 15-30 seconds per page (skip detection)
   - **Network-Only Mode**: Process only diagram pages
   - **Multi-Diagram Support**: Handle multiple diagrams per page

5. **Flexible Model Support (Enhanced v0.1.4)**
   - **Multi-Model Support**: Different models for text and network processing
   - **Text Default**: nanonets-ocr-s:latest (optimized for OCR)
   - **Network Default**: qwen2.5vl:latest (optimized for diagrams)
   - **CLI Options**: `--model` for text, `--network-model` for diagrams
   - **Model Info Display**: Shows which model is used in progress messages
   - **Fallback Chains**: Automatic fallback to alternative models

6. **Robust Error Handling**
   - **Timeout Protection**: 60-second configurable timeout per operation
   - **Graceful Fallback**: Falls back to text on detection failure
   - **Syntax Auto-Fix**: Corrects common Mermaid syntax issues
   - **Progress Tracking**: Step-by-step progress with timing
   - **Retry Logic**: Automatic retry with timeout wrapper

7. **Output Management (Enhanced v0.1.4)**
   - **MD5-Based Structure**: Each document in `output/<md5_checksum>/`
   - **Index File**: Automatic `output/index.md` tracking all documents
   - **Enhanced Footer**: Comprehensive metrics in merged documents
   - **Dual Content**: Pages include both diagram and text
   - **Smart Naming**: Uses original PDF filename for merged output
   - **Checksum Verification**: MD5 checksums for document integrity
   - **Processing Metrics**: Date, time, models, errors, warnings
   - **Summary Report**: Processing statistics and results

8. **Checkpoint/Resume Support (NEW v0.1.5)**
   - **Automatic Checkpointing**: Pages saved after each successful processing
   - **Resume Capability**: Continue from exact interruption point with `--resume`
   - **Page-Level Granularity**: Track individual page completion
   - **Processing State**: Preserve statistics and settings across sessions
   - **Smart Skip**: Already processed pages are automatically skipped
   - **Resume Summary**: Clear display of what's completed and remaining
   - **Atomic Operations**: Checkpoint integrity with atomic file writes
   - **Automatic Cleanup**: Checkpoints removed after successful completion

9. **Vector Generator Module (NEW v0.1.7 - AUTO-ENABLED)**
   - **Automatic Generation**: Creates vector files by default (no flags needed)
   - **Content Filtering**: Removes all processing artifacts, keeps only source content
   - **JSON Flattening**: Flattens nested structures for vector database ingestion
   - **Markdown Optimization**: Creates clean markdown for consistent chunking
   - **Smart Chunking**: Semantic chunking respects document structure
   - **Minimal Metadata**: Only essential fields (source, pages, date) to maximize content
   - **LanceDB Ready**: Pre-chunked JSONL files for direct ingestion
   - **Multi-Format Support**: Schemas for LanceDB, Pinecone, Weaviate, Qdrant, Chroma

   **Sub-components:**
   - `ContentFilter`: Removes non-source artifacts (timestamps, model info, etc.)
   - `JSONFlattener`: Handles nested table/diagram structures
   - `MarkdownOptimizer`: Creates vector-optimized markdown
   - `MetadataEnricher`: Adds minimal essential metadata
   - `Chunker`: Creates optimized chunks (semantic/fixed/sentence strategies)
   - `SchemaGenerator`: Generates database-specific schemas
   - `VectorGenerator`: Orchestrates all components

### Command-Line Options

#### Basic Options
- `pdf_path`: Input PDF file (required)
- `--output, -o`: Base output directory (default: `output`, documents stored in `output/<md5>/`)
- `--model, -m`: Ollama model for text extraction (default: nanonets-ocr-s:latest)
- `--network-model`: Separate model for network diagrams (v0.1.4)
- `--keep-images, -k`: Preserve converted images
- `--width, -w`: Resize images to specified width
- `--start, -s`: Start page number
- `--end, -e`: End page number
- `--resume`: Resume from checkpoint if available (v0.1.5)
- `--debug, -d`: Enable debug output
- `--verbose, -v`: Verbose output (default is quiet)

#### Processing Modes
- `--text-only, -t`: Skip network detection for speed
- `--network-only`: Process only network diagram pages
- `--fast-extraction`: Use optimized fast extraction (50-70% faster)
- `--multi-diagram`: Force multi-diagram extraction mode

#### Network Diagram Options
- `--confidence, -c`: Detection confidence threshold (0.0-1.0, default: 0.7)
- `--no-icons`: Disable Font Awesome icons
- `--diagram-only`: Extract only diagram without text content
- `--timeout`: Timeout in seconds for LLM operations (default: 60)

#### Vector Database Options (v0.1.7)
- `--no-vector`: Disable vector generation (enabled by default)
- `--vector-format`: Target database format (default: lancedb)
- `--chunk-size`: Chunk size in tokens (default: 1000)
- `--chunk-overlap`: Token overlap between chunks (default: 100)
- `--chunk-strategy`: Chunking strategy (semantic/fixed/sentence, default: semantic)
- `--embedding-metadata`: Include extended metadata (reduces content space)

## Enhanced Document Footer (v0.1.4)

Every merged document now includes comprehensive processing metrics:

### Footer Components
1. **Document Information**
   - Source filename and file size
   - MD5 checksum for verification
   - Total pages and diagrams detected

2. **Processing Details**
   - Extraction date and time
   - Models used (text and network)
   - Total processing time
   - Processing mode

3. **Quality Report**
   - Error detection with page numbers
   - Warning/timeout tracking
   - Success metrics
   - Confidence thresholds

4. **Configuration Section**
   - Auto-detection status
   - Icon usage
   - Fast extraction mode
   - Timeout settings

### Index File Format
The `output/index.md` file tracks all processed documents:
```markdown
| Filename | Timestamp | MD5 Checksum | Folder | Processing Time |
|----------|-----------|--------------|--------|----------------|
| doc1.pdf | 2025-08-20 14:30:15 | `6c9289...` | [📁 6c9289...](./6c9289.../) | 2m 30s |
```

## Technical Stack

### Dependencies
- **Python**: 3.10+ required
- **pymupdf**: PDF processing (v1.26.1+)
- **Pillow**: Image manipulation (v11.2.1+)
- **requests**: HTTP client for Ollama API (v2.32.4+)
- **tqdm**: Progress bars (v4.67.1+)
- **opencv-python**: Computer vision for diagram detection (optional)
- **numpy**: Array operations for image processing

### External Requirements
- **Ollama**: AI model server (must be running)
- **Multimodal Model**: Vision-language model (e.g., qwen2.5vl, nanonets-ocr-s)

## Current Limitations

### Functional Limitations
1. **Page Limit**: Maximum 100 pages per processing run (configurable with --start/--end)
2. **Single Document Processing**: No batch processing of multiple PDFs
3. **Sequential Processing**: Pages processed one at a time (no parallel processing)

### Technical Limitations
1. **Memory Usage**: Loads entire images into memory
2. **Network Dependency**: Requires active Ollama server connection
3. **Mermaid Complexity**: Very complex diagrams may exceed Mermaid rendering limits

### API Limitations
1. **Synchronous Processing**: Blocking API calls to Ollama
2. **Model Dependency**: Quality depends on chosen multimodal model
3. **Timeout Constraints**: Long operations may timeout (configurable)

## Security Considerations

1. **Local Processing**: All data processed locally (when using local Ollama)
2. **No Authentication**: Ollama API accessed without authentication
3. **Temporary File Handling**: Images can be deleted after processing
4. **Environment Variables**: Sensitive configuration via env vars

## Future Enhancement Opportunities

1. **Parallel Processing**: Process multiple pages simultaneously
2. **Incremental Processing**: Resume from last successful page
3. **Multiple Output Formats**: Support JSON, HTML, DocX
4. **OCR Integration**: Fallback to traditional OCR if AI fails
5. **Web Interface**: Browser-based UI for easier use
6. **Cloud Storage Integration**: Direct input/output from cloud services
7. **Batch Processing**: Handle multiple PDFs in one run
8. **Quality Metrics**: Confidence scores for transcription accuracy
9. **Custom Prompts**: User-defined transcription instructions
10. **Webhook Support**: Notify external systems on completion

## Performance Characteristics

### Processing Speed
- **Standard Mode**: 30-60 seconds per page with network detection
- **Fast Extraction Mode**: 10-20 seconds per diagram (50-70% improvement)
- **Text-Only Mode**: 15-30 seconds per page
- **Factors Affecting Speed**:
  - Ollama server performance
  - Model size and complexity
  - Image resolution (affected by --width parameter)
  - Network latency (for remote servers)
  - Diagram complexity and component count

### Resource Usage
- **CPU**: Moderate (image processing, OpenCV operations)
- **Memory**: Proportional to image size and diagram complexity
- **Disk**: Temporary storage for images (cleanup available)
- **Network**: Continuous during transcription

### Optimization Strategies
1. **Fast Extraction** (`--fast-extraction`): Simplified prompts for 50-70% speed improvement
2. **Text-Only** (`--text-only`): Skip detection when documents have no diagrams
3. **Timeout Adjustment** (`--timeout`): Reduce for faster fallback to text
4. **Model Selection**: Use lighter models for simple diagrams
5. **Image Resizing** (`--width`): Reduce resolution for faster processing

## Error Handling

### Implemented Error Handling
- **PDF Validation**: File existence and page count checking
- **Ollama Connectivity**: Server availability verification
- **Page Range Validation**: Automatic adjustment and 100-page limit enforcement
- **Timeout Management**: Configurable timeouts with graceful fallback
- **Syntax Validation**: Automatic Mermaid syntax correction (4-phase validation)
- **Detection Failures**: Falls back to text-only transcription
- **Component Extraction Errors**: Uses rule-based generation as fallback

### Error Recovery Mechanisms
1. **Text-First Guarantee**: Always captures text before attempting diagram extraction
2. **Timeout Fallback**: Automatically falls back to simpler processing on timeout
3. **Syntax Auto-Fix**: Corrects common Mermaid issues (parentheses, comments, etc.)
4. **Partial Success**: Continues processing even if individual pages fail
5. **Progress Tracking**: Clear indication of which step failed

### Areas for Future Improvement
- Checkpoint/resume capability for long documents
- Parallel processing with error isolation
- More detailed error logging and diagnostics
- Automatic retry with exponential backoff

## Version History

### v0.1.10 (2025-08-20)
**New Features:**
- **Vector Regeneration**: Added `--vector-regenerate` flag to regenerate vectors from existing markdown
- **Smart ToC Handling**: Table of Contents detected and extracted with PDFPlumber only, skipping LLM
- **New Module**: `vector_regenerator.py` for regenerating vectors without reprocessing

**Bug Fixes:**
- **Vector Generation**: Fixed chunk processing errors and metadata handling
- **Table Detection**: Fixed function signature and improved ToC detection
- **Error Handling**: Better error recovery in vector generation pipeline

### v0.1.9 (2025-08-20)
**Bug Fixes:**
- **Function Signature Fix**: Fixed `transcribe_image()` function to accept optional custom prompt parameter
- **Table Detection**: Resolved "takes 2 positional arguments but 3 were given" error
- **Backward Compatibility**: Maintains compatibility with existing code while enabling custom prompts
- **Affected Module**: Updated `ollama.py` to support flexible prompt parameter

### v0.1.8 (2025-08-20)
**Bug Fixes:**
- **Table Detection Fix**: Corrected `retry_with_timeout` function usage in table extraction modules
- **Timeout Handling**: Fixed timeout error "missing 1 required positional argument: 'func'"
- **Improved Reliability**: Enhanced error handling in table detection and extraction
- **Affected Modules**: Fixed `detector.py` and `llm_extractor.py` in table_extraction module

### v0.1.6 (2025-08-20)
**Table Extraction Support:**
- **Table Detection**: Automatic table identification in PDF pages
- **Library Extraction**: Fast extraction using pdfplumber
- **LLM Enhancement**: Vision model support for complex tables
- **JSON Output**: Structured table data in JSON format
- **Validation System**: Type inference and data validation
- **CLI Integration**: New options for table extraction control
- **Checkpoint Support**: Table state tracked in checkpoints

### v0.1.5 (2025-08-20)
**Checkpoint/Resume Support:**
- **Checkpoint System**: Automatic save after each page
- **Resume Capability**: Continue from interruption with `--resume`
- **Page Tracking**: Individual page completion status
- **State Preservation**: Settings and statistics maintained
- **Atomic Operations**: Prevent checkpoint corruption
- **Smart Recovery**: Skip already processed pages
- **Progress Display**: Clear resume summary

### v0.1.4 (2025-08-20)
**Major Enhancements:**
- **Multi-Model Support**: Separate models for text and network processing
- **MD5-Based Output**: Unique folders using document checksums
- **Index File**: Automatic tracking of all processed documents
- **Enhanced Footer**: Comprehensive processing metrics
- **Model Display**: Shows which model is used in progress messages

### v0.1.3 (2025-08-19)
**Architecture Updates:**
- Multiple processing engines (Standard, Hybrid, Network, Optimized)
- Text-first processing architecture
- Modular network diagram system
- Enhanced error recovery

### v0.1.2 (2025-08-19)
**Performance & Validation:**
- Fast extraction mode (50-70% faster)
- Robust Mermaid validation
- Automatic syntax fixing
- Timeout management

### v0.1.1 (2025-08-19)
**Initial Features:**
- Network diagram detection
- Mermaid.js generation
- Component recognition
- Basic error handling

### v0.1.0 (2025-08-19)
**Initial Release:**
- PDF to text conversion
- Ollama integration
- Basic CLI interface
- Network diagram support

## v0.1.11 Architecture Enhancements

### Containerization Architecture
```
┌─────────────────────────────────────────────────┐
│              Host System                        │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌──────────────────────────────────────────┐  │
│  │         Docker Compose Stack              │  │
│  ├──────────────────────────────────────────┤  │
│  │                                           │  │
│  │  ┌─────────────┐      ┌──────────────┐  │  │
│  │  │NetIntel-OCR │      │    MinIO     │  │  │
│  │  │  Container  │◄────►│   Storage    │  │  │
│  │  └─────────────┘      └──────────────┘  │  │
│  │         ▲                     ▲          │  │
│  │         │                     │          │  │
│  │  ┌──────┴────────┐    ┌──────┴──────┐  │  │
│  │  │ /data/input   │    │ /data/output│  │  │
│  │  │ /config       │    │ MinIO Data  │  │  │
│  │  └───────────────┘    └─────────────┘  │  │
│  └──────────────────────────────────────────┘  │
│                      ▲                          │
│                      │                          │
│              External Ollama Server             │
│              (Not containerized)                │
└─────────────────────────────────────────────────┘
```

### Configuration Flow (v0.1.11)
```
CLI Arguments
     │
     ▼
config.yml ──► ConfigManager ◄── Environment Variables
                    │
                    ▼
            Merged Configuration
                    │
     ┌──────────────┼──────────────┐
     ▼              ▼              ▼
Processing     Model Config    Storage Config
Settings       (Multi-model)    (S3/MinIO)
```

### Project Structure Created by --init
```
netintel-ocr/
├── docker/
│   ├── Dockerfile                 # Production container
│   ├── docker-compose.yml         # Multi-service stack
│   └── requirements.txt           # Python dependencies
├── helm/
│   ├── Chart.yaml                 # Helm chart metadata
│   ├── values.yaml                # Deployment configuration
│   └── templates/
│       ├── deployment.yaml        # K8s deployment
│       ├── service.yaml           # K8s service
│       ├── configmap.yaml         # Configuration
│       ├── pvc.yaml               # Storage claims
│       └── ingress.yaml           # External access
├── config/
│   └── config.yml                 # YAML configuration
├── input/                         # PDF input directory
├── output/                        # Processing output
└── .env                           # Environment variables
```

### Deployment Options
1. **Local Development**: Direct Python execution
2. **Docker Compose**: Containerized with MinIO
3. **Kubernetes**: Helm-based deployment with scaling
4. **Cloud**: S3/MinIO backend for distributed access

### Key v0.1.11 Features
- **Infrastructure as Code**: Complete deployment automation
- **Configuration Management**: YAML + environment variables
- **External Ollama**: Efficient use of existing infrastructure
- **MinIO Integration**: S3-compatible distributed storage
- **Query Foundation**: Preparation for v0.1.12 centralized DB
- **Production Ready**: Health checks, non-root user, resource limits