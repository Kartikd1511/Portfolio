# iMoot AI Backend (FastAPI)
**AI-Powered Moot Court Simulation Platform**
---
## Table of Contents
1. [Project Overview](#project-overview)
2. [Key Achievements](#key-achievements)
3. [Major Functionalities](#major-functionalities)
4. [Technology Stack](#technology-stack)
5. [Project Structure](#project-structure)
6. [Deployment](#deployment)
7. [Key Design Patterns](#key-design-patterns)
8. [Security Considerations](#security-considerations)
9. [Performance Characteristics](#performance-characteristics)
10. [Environment Variables](#environment-variables)
11. [Dependencies](#dependencies)
12. [Version & Maintainer](#version--maintainer)

---

## Project Overview
**iMoot AI Backend** is a FastAPI-based service designed to simulate AI-driven moot court sessions. It leverages **AWS Bedrock (Nova Pro)** for natural language processing to create realistic courtroom interactions between judges, attorneys, and counsels. The platform is tailored for legal education, offering immersive, structured, and evaluative moot court experiences.

---

## Key Achievements
- **Real-time AI Courtroom Simulation**: Multi-agent system with distinct AI personas (Judge & Attorney).
- **Streaming Response Architecture**: Word-by-word streaming (30ms delay per word) for natural conversation flow.
- **Phase-based Session Management**: 4-phase structured moot court proceedings with timer-based progression.
- **Intelligent Context Management**: MongoDB-backed conversation history with a 10-message context window.
- **Advanced Report Generation**: AI-powered performance evaluation with detailed grading across multiple dimensions.
- **Production-Ready Deployment**: Running on **AWS EC2** with **nginx** reverse proxy and **PM2** process management.

---

## Major Functionalities

### 1. **AI-Powered Courtroom Roles**
- **Judge AI**: Presides over sessions, asks follow-up questions (50-150 words), and provides constructive criticism.
- **Attorney AI**: Acts as opposing counsel, counters arguments with legal insights.
- **Context-Aware Responses**: Maintains conversation history and adapts based on counsel role (Appellant/Respondent).

### 2. **Four-Phase Session Management**
- **Phase 1**: Initial arguments with the judge.
- **Phase 2**: Role transition (Judge ↔ Attorney switching).
- **Phase 3**: Rebuttal session with conditional judge intervention.
- **Phase 4**: Final verdict generation.
- **Timer System**: 300 seconds per phase with pause/resume capabilities.

### 3. **Memorial & Case Management**
- Memorial submission and storage.
- Case details and judge personal qualities integration.
- Context injection into AI prompts for personalized responses.

### 4. **Advanced Evaluation System**
- **Memorial Grade**: Evaluation of pre-submitted legal memorandum (0-100).
- **Arguments Grade**: Assessment of logical coherence and legal precedents (0-100).
- **Answers to Judge Grade**: Confidence and accuracy of responses (0-100).
- **Rebuttal Session Grade**: Effectiveness of counterarguments (0-100).
- **Overall Grade**: Weighted average with detailed strengths/weaknesses/improvements.

### 5. **Streaming API**
- **Server-Sent Events (SSE)** for real-time text streaming.
- Async generator-based response handling.
- Custom `custom_message=empty` protocol for conditional responses.

### 6. **AI User Response Generation**
- Auto-generated counsel responses based on conversation context.
- 50-150 word constraint enforcement.
- Legal provision integration (e.g., Constitution of India references).

---

## Technology Stack

### Core Framework
| Category          | Tech                          |
|-------------------|-------------------------------|
| Framework         | FastAPI 0.115.5               |
| Runtime           | Python 3.11.9                   |
| ASGI Server       | Uvicorn 0.32.0                |

### AI/ML & LangChain Ecosystem
| Category          | Tech                          |
|-------------------|-------------------------------|
| LLM Orchestration | LangChain 0.3.7               |
| AWS Integration   | LangChain-AWS (ChatBedrockConverse) |
| Primary LLM       | AWS Bedrock Nova Pro v1       |
| Chat History      | LangChain-MongoDB 0.2.0       |

### Database
| Category          | Tech                          |
|-------------------|-------------------------------|
| Database          | MongoDB                       |
| Database Name     | chatbot_db                    |
| Collection        | chat_histories                |
| Driver            | PyMongo 4.10.1                |

### Configuration & Environment
| Category          | Tech                          |
|-------------------|-------------------------------|
| Settings          | Pydantic Settings 2.6.1        |
| Environment       | Python-dotenv 1.0.1           |

### Document Processing (Auxiliary)
| Category          | Tech                          |
|-------------------|-------------------------------|
| PDF Extraction    | PyPDF/PyPDF2/PDFPlumber       |
| Document Parsing  | Unstructured-ingest           |
| Web Scraping      | Beautiful Soup 4.12.3         |
| Table Extraction  | Tabula-py/Camelot-py          |

---

## Project Structure
- **System Constants**:
  - Total Phases: 4
  - Phase Duration: 300 seconds (5 minutes)
  - Streaming Delay: 0.03 seconds per word
  - Context Window: Last 10 messages
  - Response Length: 50-150 words
  - LLM Temperature: 0.1
  - Max Tokens: 4000

---

## Deployment
### Production Environment
| Category          | Configuration                 |
|-------------------|-------------------------------|
| Platform          | AWS EC2 t3.medium             |
| OS                | Ubuntu                        |
| Web Server        | Nginx (reverse proxy)         |
| Process Manager   | PM2                           |
| Workers           | 2 Uvicorn workers             |
| Memory Limit      | 1GB per process               |

---

## Key Design Patterns
1. **Async Generator Pattern**: Streaming responses using Python async generators.
2. **Message History Pattern**: LangChain MongoDB integration for context persistence.
3. **Template Pattern**: Prompt templates with variable substitution.
4. **Dependency Injection**: FastAPI's dependency injection for settings.
5. **Router Pattern**: Modular route organization.
6. **Service Layer Pattern**: Business logic separation in `services/`.

---

## Security Considerations
- Environment-based configuration (no hardcoded credentials).
- AWS credentials via environment variables.
- Nginx reverse proxy for SSL termination capability.
- Input validation using Pydantic models.
- CORS configuration for API access control.

---

## Performance Characteristics
- **Streaming Latency**: ~30ms per word.
- **Context Window**: 10 messages (optimized for token limits).
- **Concurrent Sessions**: Supports multiple simultaneous moot courts.
- **Session Isolation**: Separate MongoDB collections per session.
- **Memory Footprint**: ~1GB per worker.

---

## Environment Variables
- PYTHONPATH=/path/to/imoot-ai-backend
- AWS_ACCESS_KEY_ID=<configured>
- AWS_SECRET_ACCESS_KEY=<configured>
- MONGO_URI=<configured>
- JWT_SECRET_KEY=<configured>

---

## Dependencies

**Core**
- fastapi==0.115.5
- uvicorn==0.32.0
- pymongo==4.10.1
- pydantic==2.6.1
- python-dotenv==1.0.1

**AI/ML**
- langchain==0.3.7
- langchain-aws==0.1.0
- langchain-mongodb==0.2.0

**Document Processing**
- pypdf==4.2.0
- pdfplumber==0.11.0
- beautifulsoup4==4.12.3
- tabula-py==2.9.2
- camelot-py==0.11.0

**Utilities**
- opencv-python==4.10.0

---

## Version & Maintainer
- **Last Updated**: January 2026
- **Version**: 1.0.0
- **Maintained By**: iMoot AI Development Team
