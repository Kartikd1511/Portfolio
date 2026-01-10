# Shine on Car Backend (FastAPI)

Backend platform for real-time car wash booking, intelligent washer matching, geolocation, document verification, and automated scheduling.

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

Shineoncar is a comprehensive car wash booking platform backend built with FastAPI that connects users with professional car washers. The system features real-time booking management, geolocation-based washer discovery, push notifications, and automated scheduling capabilities.

---

## Architecture

### Technology Stack

| Category | Tech |
|---|---|
| Framework | FastAPI 0.115.6 |
| Runtime | Python 3.12 |
| Database | MongoDB Atlas (Cloud) |
| Authentication | JWT (PyJWT) |
| Realtime | WebSocket |
| CLoud | AWS EC2 |
| Instance Type | t3.medium (Ubuntu) |
| Process Manager | PM2 |
| Server | Uvicorn + uvloop (4 workers) |
| Storage | AWS S3 |
| Push Notifications | FCM |
| Geolocation | Google Maps API |

### Key Technologies

- FastAPI
- MongoDB
- Firebase Admin
- Boto3
- PyMongo
- Passlib (bcrypt)
- Pydantic

---

## Core Features

### 1. Multi-Role Authentication System

- Admin Panel: JWT-based authentication with role-based access control
- User Authentication: OTP-based phone number verification
- Washer Authentication: OTP verification with device ID tracking
- Security: bcrypt password hashing, token expiration (720 minutes)

### 2. Real-Time Updates with WebSocket

**Endpoints**
```
/user/ws/pendingcarwashes/{user_id}
/washer/ws/bookinghistory/{washer_id}
```

**Features**

- Connection management
- Automatic reconnection
- Ping-pong heartbeat
- Stale connection cleanup
- Per-user connection limits

### 3. Intelligent Washer Matching

- Haversine Distance Calculation
- Google Distance Matrix API
- Service Radius Filtering (1–500 km)
- Multi-Washer Booking
- Availability Management

### 4. Automated Background Jobs

**Cron Job (Hourly)**

```python
# Runs every 1 hour
- Updates expired bookings
- Marks completed car washes
- Maintains booking status consistency
```

**Reminder Scheduler (Every 15 minutes)**

```python
# Smart notification system
- Sends 1-hour advance reminders to users
- Notifies washers of upcoming jobs
- Handles time slot parsing (12-hour format)
- Graceful error handling
```

### 5. Push Notification System

- Firebase Cloud Messaging
- Booking approvals/rejections
- Wash status updates
- Reminders
- Account approval notifications
- Platform Support: Android (high priority), iOS (APNS)
- Custom payloads: title, body, data, images

### 6. Geolocation Services

**Address Management**
- Multiple saved addresses per user
- Default address selection
- Google Geocoding API integration
- Lat/Long coordinate storage

**Location-Based Features**
- Nearby car wash centers discovery
- Real-time washer location tracking
- Distance calculation and sorting
- Area-based washer registration

### 7. Document Management & Verification

- AWS S3 Integration
- Aadhar, PAN, Driving License
- Front and back image uploads
- Admin approval workflow
- Profile images

---

## Performance Optimizations

### Database Optimization

```
maxPoolSize: 50
minPoolSize: 10
connectTimeout: 5 seconds
socketTimeout: 10 seconds
```

### Query Optimization

- Batch queries using `$in`
- ObjectId indexing
- Projection-based field filtering
- Pre-computed distances stored in booking documents

### WebSocket Optimization

- Stale connection cleanup
- Heartbeat mechanism
- Selective broadcasting
- Graceful recovery

### Background Job Efficiency

- Daemon thread for cron job
- asyncio for reminder scheduler
- Timeout handling (10s)
- Error isolation (fault tolerance)

---

## Deployment Configuration

### PM2 Ecosystem

```json
{
  "name": "Shineoncar-backend",
  "script": "uvicorn",
  "args": "app.main:app --host 0.0.0.0 --port 8002 --workers 4 --loop uvloop",
  "interpreter": "python3.12",
  "autorestart": true,
  "max_memory_restart": "1G"
}
```

### Server Specifications

- Host: 0.0.0.0
- Port: 8002
- Workers: 4
- Event Loop: uvloop
- Memory Limit: 1GB per instance

---

## Technical Achievements

1. Scalability
2. Real-Time Capabilities
3. Geographic Intelligence
4. Reliability
5. Security
6. Developer Experience

---

## System Metrics

**API Response Times**
- Authentication: < 100ms
- Washer Search: < 500ms
- Booking Creation: < 200ms
- WebSocket Connection: < 50ms

**Concurrent Operations**
- 4 Uvicorn workers
- 50 max database connections
- Unlimited WebSocket connections

---

## Environment Variables

```
PYTHONPATH=/home/ubuntu/Shineoncar-backend/app
AWS_ACCESS_KEY_ID=<configured>
AWS_SECRET_ACCESS_KEY=<configured>
AWS_BUCKET_NAME=Shineoncar
GOOGLE_PLACES_API_KEY=<configured>
JWT_SECRET_KEY=<configured>
```

---

## Dependencies

**Core**
- fastapi==0.115.6
- uvicorn==0.34.0
- pymongo==4.10.1
- pydantic==2.10.5

**Authentication & Security**
- python-jose==3.3.0
- passlib==1.7.4
- bcrypt (via passlib)

**Cloud Services**
- firebase-admin==6.5.0
- boto3==1.35.97
- google-auth==2.29.0

**Utilities**
- python-multipart==0.0.20
- dnspython==2.7.0
- email-validator==2.2.0

---

## Major Functionalities

1. Multi-Washer Booking System
2. Smart Scheduling
3. Real-Time Tracking via WebSocket
4. Comprehensive Admin Panel
5. Flexible Service Configuration
6. Intelligent Matching
7. Document Verification
8. Push Notification Engine
9. Address Management
10. Analytics (Weekly Statistics)

---

## API Base URLs

```
Base URL: http://0.0.0.0:8002
Admin APIs: /v1/api/admin
User APIs: /v1/api/user
Washer APIs: /v1/api/washer
Notifications: /v1/api/notifications
Static Files: /static
```

---

## Startup Sequence

1. FastAPI app initialization
2. CORS middleware configuration
3. Static file mounting
4. Router registration
5. Database connection pool creation
6. Background cron job thread start
7. Reminder scheduler async task start
8. Uvicorn server start on port 8002

---

## Deployment

The backend is deployed on an AWS EC2 instance.

### Infrastructure Details

| Component | Configuration |
|---|---|
| Cloud Provider | AWS |
| Instance Type | t3.medium |
| OS | Ubuntu |
| Deployment Method | PM2 + Uvicorn |
| Network Access | Public IPv4 + Security Group Rules |
| File Storage | AWS S3 (documents, profile images) |
| Notifications | Firebase Cloud Messaging (FCM) |

### Operational Notes

- PM2 manages process lifecycle, automatic restart, and memory thresholds.
- Uvicorn (async workers) handles HTTP requests and WebSockets concurrently.
- Security Groups restrict inbound ports to 80/443/8002 (project-specific).
- AWS S3 acts as object storage for documents and images.
- Cron jobs and schedulers run in-process for reminders and booking updates.

---

**Last Updated:** January 2026  
**Version:** 1.0.0  
**Maintained By:** Shineoncar Development Team
