# Applied AI / GenAI Backend Engineer

Applied AI Engineer specializing in **AI-powered platforms**, **enterprise automation**, and **real-time systems**. Proven track record of building production-grade backends that process millions of transactions, serve thousands of users, and integrate cutting-edge AI technologies.

---

## 🎯 Core Expertise

**Backend Engineering**
- FastAPI, Python 3.11-3.12
- MongoDB, AWS Services (S3, EC2, Bedrock, Textract, SageMaker)
- RESTful APIs, WebSocket real-time systems, Server-Side Events
- JWT authentication, OAuth, OTP verification

**AI/ML Integration**
- AWS Bedrock (Nova Pro, Nova Lite, Titan Embeddings)
- LangChain, LlamaIndex
- RAG (Retrieval-Augmented Generation)
- Document processing with OCR (AWS Textract)
- Predictive analytics with Facebook Prophet

**System Architecture**
- Microservices design
- Multi-tenant architecture
- Real-time data processing
- Background job scheduling
- Production deployment (PM2, Nginx, EC2)

**Integration & Automation**
- POS system integration (Verifone, Gilbarco)
- Web scraping (Selenium, BeautifulSoup)
- Firebase Cloud Messaging (FCM)
- Google Maps API, geolocation services
- Email automation (SMTP)

---

## 🚀 Featured Projects

### [AdvocateGPT AI](AdvocateGPTAI.md) - Legal AI Assistant
**AI-powered legal consultation platform for the Indian legal system**

**Tech Stack**: FastAPI, AWS Bedrock (Nova Pro/Lite), MongoDB, LangChain, AWS Textract, S3

**Key Features**:
- Intelligent legal chat with domain-specific guardrails (IPC, BNS, CrPC, CPC, Constitution)
- RAG system for case law retrieval and document analysis
- Automated case summarization with structured JSON output
- Multi-document comparison engine with MapReduce summarization
- OCR-powered scanned document processing
- Streaming responses (SSE) for natural conversation flow

**Technical Highlights**:
- Context-aware conversations with 6-message history
- Hybrid text extraction (PyMuPDF + AWS Textract OCR)
- Session-based knowledge base management
- Production deployment on EC2 with 2 Uvicorn workers

---

### [iMoot AI](iMoot%20AI.md) - Moot Court Simulation
**AI-driven courtroom simulation for legal education**

**Tech Stack**: FastAPI, AWS Bedrock Nova Pro, MongoDB, LangChain

**Key Features**:
- Multi-agent AI system (Judge AI + Attorney AI)
- Four-phase structured moot court proceedings
- Timer-based session management (300s per phase)
- Advanced evaluation system with multi-dimensional grading
- Real-time streaming responses (30ms per word)
- Context-aware AI responses with 10-message window

**Technical Highlights**:
- Memorial and case management integration
- Auto-generated counsel responses (50-150 words)
- Performance evaluation across memorial, arguments, answers, and rebuttals
- Production-ready on EC2 with Nginx reverse proxy

---

### [Petrolynks](Petrolynks.md) - Gas Station Automation
**Enterprise platform for gas station and retail store management**

**Tech Stack**: FastAPI, MongoDB Atlas, Selenium, AWS S3, Facebook Prophet

**Key Features**:
- Real-time POS integration (Verifone/Gilbarco XML parsing)
- Automated fuel price synchronization (backend → dispenser < 30s)
- Advanced tank monitoring (Franklin Fueling, Veeder Root, OPW)
- Live transaction processing with 30-second polling
- Inventory management with low-stock alerts
- Predictive analytics for demand forecasting

**Technical Highlights**:
- Multi-store management with MAC-based auto-detection
- Web scraping for ATG system data (50-second refresh)
- Two-way POS communication with cookie authentication
- Automated compliance reporting (FMS, SIR, CSLD)
- Email digest system for alarms and notifications
- Purchase reconciliation with S3-based invoice storage

**Performance**:
- Processes thousands of transactions per day
- Supports unlimited store locations
- Real-time tank status monitoring across multiple sites

---

### [Shine On Car](Shine%20On%20Car.md) - Car Wash Booking Platform
**Real-time car wash booking and management system**

**Tech Stack**: FastAPI, MongoDB Atlas, Firebase Admin, AWS S3, Google Maps API

**Key Features**:
- Multi-role authentication (Admin JWT, User/Washer OTP)
- Real-time WebSocket updates for bookings
- Intelligent washer matching with Haversine distance calculation
- Automated booking status updates (hourly cron job)
- Smart reminder system (15-minute scheduler)
- Push notifications via Firebase Cloud Messaging

**Technical Highlights**:
- Geolocation-based washer discovery (1-500 km radius)
- Google Distance Matrix API integration
- AWS S3 document verification (Aadhar, PAN, DL)
- WebSocket connection management with heartbeat
- Production deployment on EC2 t3.medium with 4 workers

**Performance Optimizations**:
- MongoDB connection pooling (50 max, 10 min)
- Batch queries with `$in` operators
- Pre-computed distances in booking documents
- Stale connection cleanup for WebSocket

---

## 💼 Technical Skills

**Languages**: Python

**Frameworks**: FastAPI, LangChain, LlamaIndex

**Databases**: MongoDB Atlas, MongoDB (self-hosted)

**Cloud & DevOps**:
- AWS (EC2, S3, Bedrock, Textract, SageMaker, Lambda)
- Process Management: PM2, Uvicorn
- Web Servers: Nginx (reverse proxy)
- CI/CD: Git-based deployment workflows

**AI/ML**:
- Large Language Models (AWS Bedrock Nova Pro/Lite)
- RAG (Retrieval-Augmented Generation)
- Document Intelligence (AWS Textract OCR)
- Predictive Analytics (Facebook Prophet)
- Embedding Models (Titan Embed v2)

**Integration & APIs**:
- Firebase Cloud Messaging (FCM)
- Google Maps API (Geocoding, Distance Matrix)
- POS Systems (Verifone, Gilbarco)
- ATG Systems (Franklin Fueling, Veeder Root, OPW)
- Payment Gateways

**Real-Time Systems**:
- WebSocket (FastAPI native)
- Server-Sent Events (SSE)
- Background job scheduling (APScheduler)
- Async/await patterns

**Security**:
- JWT authentication
- OTP verification (phone-based)
- bcrypt password hashing
- Role-based access control (RBAC)
- AWS IAM integration

---

## 📊 Performance Metrics

- **Production Systems**: 4 live platforms serving active users
- **Transaction Processing**: Millions of retail transactions processed
- **Real-Time Connections**: WebSocket systems handling concurrent users
- **AI Inference**: Thousands of AI-powered legal consultations
- **Document Processing**: OCR and analysis of hundreds of legal documents
- **Multi-Tenant Systems**: Supporting unlimited store/location scalability

---

## 🏗️ Architecture Patterns

**Design Principles**:
- Microservices architecture
- Event-driven design
- Async-first approach
- Repository pattern for data access
- Service layer for business logic
- Dependency injection (FastAPI)

**Scalability**:
- Horizontal scaling with multiple workers
- Connection pooling for databases
- Batch processing for heavy operations
- Background job isolation
- Graceful degradation and fault tolerance

**Observability**:
- Structured logging
- Error tracking and alerting
- Performance monitoring
- S3-based log archival

---

## 📫 Contact

Available for backend engineering roles focusing on AI integration, enterprise automation, and high-performance systems.

---

## 📂 Project Documentation

Each project includes detailed documentation covering:
- System architecture and technology stack
- Core features and functionalities
- Performance optimizations
- Deployment configurations
- Environment setup and dependencies

Click on any project title above to view complete documentation.

---

*Last Updated: January 2026*
