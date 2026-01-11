# AdvocateGPT AI Backend (FastAPI)

Backend platform for intelligent legal consultation, case document analysis, legal research assistant powered by AWS Bedrock AI models, designed specifically for advocates, judges, and law students in the Indian legal system.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Core Features](#core-features)
4. [Performance Optimizations](#performance-optimizations)
5. [Deployment Configuration](#deployment-configuration)
6. [Technical Achievements](#technical-achievements)
7. [System Metrics](#system-metrics)
8. [Environment Variables](#environment-variables)
9. [Dependencies](#dependencies)
10. [Major Functionalities](#major-functionalities)
11. [API Base URLs](#api-base-urls)
12. [Startup Sequence](#startup-sequence)
13. [Deployment](#deployment)
14. [Version & Maintainer](#version--maintainer)

---

## Project Overview

AI Legal Assistant is a specialized backend platform built with FastAPI that provides intelligent legal consultation services powered by AWS Bedrock. The system features real-time conversational AI, document processing with OCR, RAG (Retrieval-Augmented Generation) capabilities, and automated case summarization specifically tailored for the Indian legal system.

---

## Architecture

### Technology Stack

| Category | Tech |
|---|---|
| Framework | FastAPI 0.122.0 |
| Runtime | Python 3.12 |
| Database | MongoDB Atlas (Cloud) |
| AI/LLM | AWS Bedrock (Nova Pro, Nova Lite) |
| Embeddings | AWS Bedrock Titan Embed v2 |
| Document Processing | AWS Textract (OCR) |
| Cloud Storage | AWS S3 |
| RAG | AWS Bedrock Knowledge Base |
| Instance Type | EC2 t3.medium (Ubuntu) |
| Process Manager | PM2 |
| Server | Uvicorn (2 workers) |

### Key Technologies

- **FastAPI** - Modern async web framework
- **LangChain** - LLM orchestration and chains
- **AWS Bedrock** - Foundation models (Nova Pro, Nova Lite, Titan)
- **MongoDB** - Document storage and chat history
- **AWS Textract** - OCR for scanned PDFs
- **PyMuPDF (fitz)** - PDF text extraction
- **LlamaIndex** - Document indexing and retrieval

---

## Core Features

### 1. Intelligent Legal Chat System

**Domain-Specific AI Assistant**
- Specialized for Indian legal system (IPC, BNS, CrPC, BNSS, CPC, BSA)
- Constitutional law and fundamental rights expertise
- Supreme Court and High Court judgment knowledge
- Legal procedures and documentation guidance

**Guardrail System**
- Pre-generation prompt engineering with domain restrictions
- Post-generation response validation
- Automatic rejection of non-legal queries
- Ensures responses stay within legal domain

**Session Management**
- MongoDB-backed chat history persistence
- Context-aware conversations (last 6 messages)
- Session continuity across multiple requests
- Auto-generated session IDs for new conversations

### 2. Document Processing & Analysis

**Multi-Format Support**
- PDF extraction (text-based and scanned)
- DOCX document processing
- AWS Textract OCR for scanned documents
- Vision API integration for image-based text

**Smart Text Extraction**
- Hybrid approach: Direct text + OCR fallback
- Direct text extraction using PyMuPDF
- Automatic OCR when text extraction fails
- Base64 image encoding for Vision API
- Page-wise processing for large documents

**Chunking Strategy**
- chunk_size = 4000 characters
- chunk_overlap = 200 characters
- max_chunks_used = 8 (first 8 chunks)

### 3. RAG (Retrieval-Augmented Generation)

**AWS Bedrock Knowledge Base Integration**
- S3-backed document storage
- Automatic knowledge base synchronization
- Session-namespaced document organization
- Embedding-based similarity search

**Document Upload Flow**
```ini
Upload → S3 Storage → KB Sync → Ingestion Job → Ready for Queries
```

**Metadata Tracking**
- MongoDB stores file metadata, S3 URIs, and ingestion status
- Session-based file organization
- Ingestion job status monitoring

### 4. Advanced Case Summarization

**Automated Case Analysis**
- Extracts case metadata (name, number, court, judges)
- Generates structured JSON summaries
- Tags cases with relevant legal topics
- Provides optimized summaries with key points

**Case Information Extracted**
```JSON
{
  "case_name": "Plaintiff vs Defendant",
  "case_number": "Case reference",
  "case_year": 2024,
  "court": "Court name",
  "date": "YYYY-MM-DD",
  "judges": ["Judge names"],
  "judgment": "Brief decision",
  "case_tags": ["civil procedure", "appeal"],
  "summary": "Concise 3-4 line summary",
  "optimized_summary": "Detailed summary with key points"
}
```

**Processing Pipeline**

1. Fetch PDF from S3 via MongoDB metadata
2. Extract text using AWS Textract
3. Truncate to first 15,000 characters
4. Generate structured summary via Bedrock Nova
5. Parse JSON response with fallback mechanisms
   
### 5. Streaming Responses

**Server-Sent Events (SSE) Architecture**
- Word-by-word streaming for natural feel
- 30ms delay per word for smooth reading
- Session ID transmitted at stream completion
- [DONE] marker for stream termination

**Streaming Format**
```
data: word \n\n
data: [SESSION_ID]session-id-here\n\n
data: [DONE]\n\n
```

**Implementation Strategy**
*Non-streaming API call → Python-side streaming*
1. Get full response from Bedrock Converse API
2. Split response into words
3. Stream words with controlled delay
4. Save to chat history after completion

### 6. File Comparison & Multi-Document Analysis

**Document Comparison Engine**
- Upload multiple files simultaneously
- Compare selected cloud-stored documents
- Generate unified summaries across documents
- Background task processing for heavy operations

**MapReduce Summary Generation**
- **Map Phase**: Per-page summaries
- **Reduce Phase**: Merged comprehensive summary
- **Vector Indexing**: LlamaIndex with embeddings
- **Query Engine**: Retry-based retrieval with relevancy evaluation

### 7. AWS Bedrock Converse API Integration

**Model Configuration**
```
Model: amazon.nova-pro-v1:0 (main responses)
Guardrail Model: amazon.nova-lite-v1:0 (fast validation)
Temperature: 0.5 (balanced creativity/accuracy)
Max Tokens: 4096
Top P: 0.9
```

**Message Format**
```JSON
[
  {"role": "user", "content": [{"text": "query"}]},
  {"role": "assistant", "content": [{"text": "response"}]}
]
```

**Performance Optimizations**

Response Generation

**Guardrail Optimization**
*Use Nova Lite for fast guardrail checks*
```
Temperature: 0.1 (deterministic)
Max Tokens: 50 (YES/NO answers only)
Snippet: 300 chars max for faster processing
```

**Context Management**
- Last 6 messages for conversation history
- First 8 chunks for file context (max ~32K chars)
- Truncate documents to 15K chars for summarization

Database Optimization

**MongoDB Connection Pool**
- Default connection pooling
- Async operations for non-blocking I/O
- Upsert operations for session management

Concurrent Processing

**Background Tasks**
- FastAPI BackgroundTasks for async summary generation
- ThreadPoolExecutor for parallel file processing (20 workers)
- Concurrent file comparison and indexing

Error Handling

**Graceful Degradation**
- Fail-open guardrails (allow on check failure)
- Fallback JSON parsing (regex extraction)
- Timeout handling with retries
- Detailed error logging

---
**Deployment Configuration**

PM2 Ecosystem
```JSON
{
  "name": "ai-backend",
  "script": "venv/bin/python",
  "args": "-m uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 2",
  "cwd": "/home/ubuntu/ai-backend-production",
  "instances": 1,
  "exec_mode": "fork",
  "autorestart": true,
  "max_memory_restart": "1G"
}
```

**Server Specifications**
- **Host**: 0.0.0.0
- **Port**: 8000
- **Workers**: 2
- **Memory Limit**: 1GB per instance
- **Auto-restart**: Enabled

---

## Technical Achievements
**1. Intelligent Domain Restriction**
- Multi-layer guardrail system prevents off-topic responses
- Specialized legal AI with comprehensive Indian law knowledge
- Professional response quality for legal practitioners

**2. Hybrid Document Processing**
- Seamless fallback from text extraction to OCR
- Multi-format support (PDF, DOCX, scanned documents)
- Page-wise processing for granular control

**3. Advanced RAG Architecture**
- AWS Bedrock Knowledge Base for scalable retrieval
- Session-namespaced document organization
- Automatic synchronization and ingestion tracking

**4. Structured Data Extraction**
- AI-powered case metadata extraction
- JSON-structured output for easy integration
- Robust parsing with multiple fallback strategies

**5. Real-Time Streaming**
- Natural word-by-word streaming experience
- Non-blocking async architecture
- Efficient resource utilization

**6. Multi-Document Intelligence**
- Comparative analysis across multiple documents
- Unified summary generation with MapReduce
- Vector-based retrieval for accurate information

---

## System Metrics

**API Response Times**
- Chat (simple): < 3-5 seconds
- Chat (with file context): < 8-10 seconds
- Document upload: < 2-5 seconds (excluding KB sync)
- Case summarization: < 15-20 seconds

**Processing Capabilities**
- Concurrent file uploads: 20 workers
- Chat history context: 6 messages
- Document chunk size: 4000 characters
- Max file context: ~32,000 characters

**Model Performance**
- Nova Pro: High-quality legal responses
- Nova Lite: Fast guardrail checks (< 1 second)
- Titan Embeddings: Accurate semantic search

---

## Environment Variables

```ini
# AWS Configuration
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=<configured>
AWS_SECRET_ACCESS_KEY=<configured>

# AWS Bedrock Models
AWS_BEDROCK_MODEL_ID=amazon.nova-pro-v1:0
AWS_EMBEDDING_MODEL_ID=amazon.titan-embed-text-v2:0

# Bedrock Knowledge Base
BEDROCK_KB_ID=<configured>
BEDROCK_DATA_SOURCE_ID=<configured>
BEDROCK_S3_BUCKET=<configured>

# MongoDB
MONGO_DB_URI=mongodb+srv://...
MONGO_DB_NAME=advGPT
NODE_MONGODB_URI=<configured>
NODE_MONGODB_NAME=advGPT

# Server Configuration
HOST=localhost
PORT=5813

#Optional APIs
GOOGLE_API_KEY=<configured>
GOOGLE_CSE_ID=<configured>
VISION_API_KEY=<configured>
ZENROWS_API_KEY=<configured>
```
---

## Dependencies

**Core Framework**
- fastapi==0.122.0
- uvicorn==0.34.0
- pydantic==2.10.5

**AWS Services**
- boto3==1.41.3
- botocore==1.41.3

**LangChain Ecosystem**
- langchain==0.3.27
- langchain-aws==0.2.35
- langchain-community==0.3.31
- langchain-core==0.3.80
- langchain-mongodb==0.7.2
- langchain-openai==0.3.35
- langchain-text-splitters==0.3.11

**LlamaIndex**
- llama-index-core==0.12.52.post1
- llama-index-embeddings-openai==0.3.1
- llama-index-llms-openai==0.3.44

**Document Processing**
- PyMuPDF (fitz)==6.0.2
- PyPDF2
- python-docx
- pdf2image
- Pillow

**Database**
- pymongo==4.10.1
- dnspython==2.7.0

**Utilities**
- aiofiles==25.1.0
- python-multipart==0.0.20
- requests
- beautifulsoup4==4.14.2

---

## Major Functionalities

**1. Legal Conversational AI**
- Context-aware chat with memory
- Specialized for Indian legal system
- Streaming responses with SSE
- Session-based conversation continuity
  
**2. Document Upload & Processing**
- Multi-format support (PDF, DOCX)
- OCR for scanned documents
- S3 storage with metadata tracking
- Bedrock Knowledge Base integration
  
**3. RAG-Powered Q&A**
- Document-context aware responses
- Retrieval-Augmented Generation
- Session-namespaced knowledge bases
- Citation and source tracking
  
**4. Automated Case Summarization**
- Extract case metadata automatically
- Generate structured JSON summaries
- Legal topic tagging
- Judgment and key points extraction
  
**5. Multi-Document Comparison**
- Upload and compare multiple files
- Cloud-stored document selection
- MapReduce-based summary generation
- Background task processing
  
**6. Intelligent Guardrails**
- Domain-specific response validation
- Automatic rejection of off-topic queries
- Post-generation quality checks
- Professional legal tone enforcement
  
**7. Session Management**
- MongoDB-backed chat history
- Auto-generated session IDs
- Conversation context preservation
- Multi-session support
  
**8. File Context Integration**
- Attach documents to conversations
- Chunked context injection
- Relevant passage retrieval
- Source attribution

---

## API Base URLs

```ini
Base URL: http://0.0.0.0:8000
Chat API: /v1/api/chat-without-stream (POST)
Case Summarization: /v1/api/case-summarize (POST)
File Upload: /v1/api/upload-file (POST)
Multi-File Upload: /v1/api/upload-multiple-files (POST)
File Comparison: /v1/api/compare-files (POST)
Get Summary: /v1/api/generate-summary (GET)
Health Check: / (GET)
```

---

## Startup Sequence

**1. Environment Loading**
- Load .env configuration
- Validate AWS credentials
- Check required environment variables

**2. AWS Service Initialization**
- Initialize Boto3 session
- Create Bedrock Runtime client
- Initialize Bedrock Agent Runtime
- Setup S3 client
- Configure Textract client

**3. MongoDB Connection**
- Connect to MongoDB Atlas
- Initialize collections (assistants, uploaded_files, query_engines, chat_histories)
- Setup connection pool

**4. LangChain Setup**
- Initialize ChatBedrock LLM (Nova Pro)
- Setup BedrockEmbeddings (Titan Embed v2)
- Configure message history backends

**5. FastAPI Application**
- Initialize FastAPI app
- Configure CORS middleware
- Register route blueprints (/chat, /upload)
- Mount static file handling (if any)

**6. Server Start**
- Uvicorn server starts on port 8000
- 2 worker processes spawn
- Health check endpoint activated
- Ready to accept requests

**7. Background Services**
- Knowledge Base sync monitoring
- Session cleanup (if configured)

---

## Deployment

The backend is deployed on an AWS EC2 instance.


Infrastructure Details

| Component |	Configuration |
| --- | --- |
| Cloud Provider | AWS |
| Instance Type |	EC2 t3.medium |
| OS	| Ubuntu |
| vCPUs |	2 |
| Memory |	4 GB |
| Deployment Method |	PM2 + Uvicorn |
| Network Access |	Public IPv4 + Security Group Rules |
| File Storage |	AWS S3 (documents, PDFs) |
| AI Services |	AWS Bedrock (Nova Pro, Titan Embeddings) |
| OCR Service |	AWS Textract |
| Database |	MongoDB Atlas (Cloud) |


Operational Notes
- **PM2** manages process lifecycle, automatic restart, and memory thresholds
- **Uvicorn** (async workers) handles HTTP requests and streaming responses concurrently
- **Security Groups** restrict inbound ports to 22 (SSH), 80 (HTTP), 443 (HTTPS), 8000 (API)
- **AWS S3** acts as object storage for uploaded legal documents and PDFs
- **AWS Bedrock Knowledge Base** provides RAG capabilities with automatic synchronization
- **MongoDB Atlas** provides cloud-hosted database with automatic backups
- **AWS Textract** processes scanned PDFs for OCR


Deployment Commands
```ini
# Start the application
pm2 start ecosystem.config.js

# View logs
pm2 logs ai-backend

# Restart application
pm2 restart ai-backend

# Stop application
pm2 stop ai-backend

# Monitor resources
pm2 monit
```

---

## Version & Maintainer

**Last Updated**: January 2026

**Version**: 1.0.0

**Maintained By**: AdvocateGPT AI Legal Assistant Development Team

---

## Notes
- This system is **specifically designed for the Indian legal system** and may require adaptation for other jurisdictions
- **AWS Bedrock Nova Pro** is the primary LLM, providing high-quality legal responses
- **Guardrail system** ensures all responses stay within legal domain expertise
- **Session-based architecture** allows for natural, context-aware conversations
- **Hybrid document processing** ensures both text-based and scanned PDFs are handled effectively
- **RAG architecture** provides accurate, source-attributed responses from uploaded documents

---

**Powered by AWS Bedrock | Built with FastAPI | Optimized for Legal Professionals**
