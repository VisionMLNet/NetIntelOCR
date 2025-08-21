# NetIntel-OCR Architecture Overview

## Overview

NetIntel-OCR is a specialized Python-based tool that intelligently processes PDF documents, automatically detecting and converting network diagrams into Mermaid.js format while preserving text content. It uses multimodal AI models via Ollama to process PDFs by converting pages to images and then using vision-language models to transcribe content and identify network-specific diagrams for enhanced extraction. 

The system delivers the complete centralized database architecture with advanced query engine, parallel batch processing, S3/MinIO cloud storage integration, and comprehensive embedding management system.

## System Architecture

### Component Diagram

```mermaid
graph LR
    %% CLI Entry Point
    CLI[CLI Entry<br/>cli.py]
    CLI --> |--init| Init[Project Initialization]
    CLI --> |--config| Config[YAML Configuration]
    CLI --> |--query| Query[Advanced Query Engine]
    CLI --> |--merge-to-centralized| Merge[Centralized Database]
    CLI --> |--batch-ingest| Batch[Parallel Batch Processing]
    CLI --> |--db-stats| Stats[Database Management]
    CLI --> |--s3-sync| S3Sync[Cloud Storage Integration]
    CLI --> |--network-model| MultiModel[Multi-model Support]
    CLI --> |--table-method| TableMethod[Table Extraction]
    CLI --> |--vector-regenerate| VectorRegen[Vector Regeneration]

    %% Main Processing Components
    CLI --> SingleDoc[Single Doc<br/>Processing]
    CLI --> BatchProc[Batch Processor]
    CLI --> QueryEngine[Query Engine]
    CLI --> CentralDB[Centralized DB Mgr]
    CLI --> S3Storage[S3 Storage]

    %% Processing Pipeline
    subgraph Pipeline[Processing Pipeline]
        TextModel[Text Model<br/>nanonets-ocr]
        NetworkModel[Network Model<br/>qwen2.5vl]
        TableExtract[Table Extract<br/>pdfplumber]
        VectorGen[Vector Gen<br/>Enhanced]
        QueryEng[Query Engine<br/>Advanced]
        EmbedMgmt[Embedding<br/>Management]
        StorageSync[Storage Sync<br/>S3/MinIO]
    end

    SingleDoc --> Pipeline
    BatchProc --> Pipeline
    QueryEngine --> Pipeline
    CentralDB --> Pipeline
    S3Storage --> Pipeline

    %% Core Modules
    subgraph CoreModules[Core Modules]
        PDFHandler[PDF Handler]
        NetworkDiagram[Network Diagram<br/>Module]
        TableModule[Table Extract<br/>Module]
        VectorModule[Vector Generator<br/>Module]
        DBManager[Centralized<br/>Database Manager]
        QueryFilter[Query Filter<br/>& Ranker]
        EmbedProvider[Embedding Provider<br/>& Cache]
        CloudSync[Cloud Storage<br/>Sync]
    end

    TextModel --> PDFHandler
    NetworkModel --> NetworkDiagram
    TableExtract --> TableModule
    VectorGen --> VectorModule
    QueryEng --> DBManager
    QueryEng --> QueryFilter
    EmbedMgmt --> EmbedProvider
    StorageSync --> CloudSync

    %% Sub-components
    subgraph SubComponents[Sub-components]
        Detector[Detector<br/>Extractor]
        ChunkGen[Chunk<br/>Gen]
        TableValid[Table<br/>Valid]
        DedupLogic[Dedup<br/>Logic]
        MergeIndex[Merge<br/>Index]
        FilterMulti[Filter<br/>Multi]
        OpenAI[OpenAI<br/>Ollama]
        CacheTTL[Cache<br/>& TTL]
        UploadS3[Upload<br/>S3]
        BackupVer[Backup<br/>& Ver]
    end

    NetworkDiagram --> Detector
    VectorModule --> ChunkGen
    TableModule --> TableValid
    DBManager --> DedupLogic
    DBManager --> MergeIndex
    QueryFilter --> FilterMulti
    EmbedProvider --> OpenAI
    EmbedProvider --> CacheTTL
    CloudSync --> UploadS3
    CloudSync --> BackupVer

    %% Styling
    classDef cli fill:#f9f,stroke:#333,stroke-width:2px
    classDef new fill:#9f9,stroke:#333,stroke-width:2px
    classDef config fill:#99f,stroke:#333,stroke-width:2px
    classDef pipeline fill:#ff9,stroke:#333,stroke-width:2px
    classDef module fill:#f99,stroke:#333,stroke-width:2px
    classDef subcomp fill:#9ff,stroke:#333,stroke-width:2px

    class CLI cli
    class Query,Merge,Batch,Stats,S3Sync,BatchProc,QueryEngine,CentralDB,S3Storage new
    class Init,Config config
    class TextModel,NetworkModel,TableExtract,VectorGen,QueryEng,EmbedMgmt,StorageSync pipeline
    class PDFHandler,NetworkDiagram,TableModule,VectorModule,DBManager,QueryFilter,EmbedProvider,CloudSync module
    class Detector,ChunkGen,TableValid,DedupLogic,MergeIndex,FilterMulti,OpenAI,CacheTTL,UploadS3,BackupVer subcomp
```
