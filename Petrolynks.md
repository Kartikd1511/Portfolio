# Petrolynks – Gas Station & Retail Automation

Backend platform for real-time gas station and convenience store management with POS integration, automated compliance reporting, tank monitoring, inventory management, and predictive analytics.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Core Features](#core-features)
4. [Performance Optimizations](#performance-optimizations)
5. [Technical Achievements](#technical-achievements)
6. [System Metrics](#system-metrics)
7. [Environment Variables](#environment-variables)
8. [Dependencies](#dependencies)
9. [Major Functionalities](#major-functionalities)
10. [Startup Sequence](#startup-sequence)
11. [Version & Maintainer](#version--maintainer)

---

## Project Overview

Petrolynks is a comprehensive gas station and convenience store management platform that bridges the gap between legacy POS systems and modern cloud-based analytics. The system integrates directly with Verifone and Gilbarco POS hardware, automating sales tracking, tax compliance, fuel price synchronization, tank monitoring, and inventory management across multiple store locations.

**Core Data Flow:**
```
Verifone/Gilbarco POS (XML) → Python ETL Scripts → MongoDB (JSON) → Analytics & Reporting
↓
ATG Systems (Web Scraping)
```

---

## Architecture

### Technology Stack

| Category | Tech |
|---|---|
| Framework | FastAPI 0.115.6 |
| Runtime | Python 3.12 |
| Database | MongoDB Atlas (Cloud) |
| POS Systems | Verifone, Gilbarco Passport |
| ATG Systems | Franklin Fueling, Veeder Root, OPW Integra |
| Cloud Storage | AWS S3 |
| Process Manager | PM2 |
| Server | Uvicorn |
| Storage | AWS S3 (Logs, Backups) |
| Web Automation | Selenium + ChromeDriver |
| Email Service | SMTP (Gmail) |
| Machine Learning | AWS SageMaker Canvas, Facebook Prophet |

### Key Technologies

- **FastAPI** - REST API endpoints for web dashboard
- **MongoDB** - Centralized multi-store database
- **Selenium** - ATG system web scraping
- **Boto3** - AWS S3 integration
- **PyMongo** - MongoDB driver
- **XMLtoDict** - POS XML parsing
- **Schedule** - Background job scheduling
- **Requests** - POS HTTP communication

---

## Core Features

### 1. Multi-Store Management System

- **MAC-Based Store Detection**: Automatic store identification via physical network adapter MAC addresses
- **Centralized Configuration**: MongoDB-based store profiles with POS credentials, ATG URLs, and notification settings
- **Multi-Tenant Architecture**: Single codebase supports unlimited store locations
- **Store Attributes**:
  - `storeid`, `corpid`, `store_name`
  - `macAddress` (supports multiple formats: dash/colon, upper/lower)
  - `tankStatusUrl`, `site_id`, `ownerGmail`
  - POS credentials: `pos_username`, `pos_password`

### 2. Real-Time POS Integration

**Verifone/Gilbarco Communication Flow:**

```python
# Fetch XML datasets from POS
Commands: [
  "vMaintenance&dataset=Item",      # Product catalog
  "vposcfg",                         # POS configuration
  "vtaxratecfg",                     # Tax rates
  "vfuelprices",                     # Current fuel prices
  "vfuelcfg",                        # Fuel configuration
  "vposjournal"                      # Transaction logs
]
```

**Features:**
- Cookie-based authentication with POS systems
- Real-time XML journal parsing for live sales
- Two-way communication (fetch + post)
- Automatic retry logic with connection pooling
- Support for both fuel and grocery transactions

### 3. Fuel Price Synchronization

**Backend-to-Dispenser Price Updates:**

Workflow:

1. Web dashboard triggers price change
2. Document inserted into database
3. update_fuel_price.py detects change (polling)
4. Fetches session cookies from POS
5. Posts updated vfuelprices XML to dispenser
6. Two-stage update: prices + fuel initialization
7. Deletes trigger document after success

**Response Time:** < 30 seconds from web to dispenser

### 4. Advanced Tank Monitoring (PetroView)

**Live Tank Status:**

- Selenium-based scraping from Franklin/Veeder Root/OPW web interfaces
- Real-time metrics per tank:
    - Volume (gross/net), Level, Ullage
    - Temperature, Water contamination
    - Capacity percentage, Alarms
- Stored in database with timestamp
- Refresh interval: 50 seconds

**ATG Alarm Centralization:**

- Unified alarm dashboard across multiple ATG vendors
- Email digest system (configurable: hourly/daily)
- Immediate alerts for critical alarms
- Alarm attributes: severity, timestamp, tank/manifold ID

**Daily Inventory Reports:**

- Automated daily snapshots at configurable times
- Variance detection (delivery vs. sales)
- FMS compliance report fetching
- Delivery history tracking

**Flow Rate & Stopped Nozzles:**

- Nozzle-wise performance monitoring
- Historical sales-based priority alerts
- Automatic detection of non-operational nozzles
- Email notifications to store owners

**SIR Compliance:**

- Tank leak detection reports
- Variance test results
- CSLD diagnostics
- Sensor test audit logs

### 5. Real-Time Sales Tracking

**Live Transaction Processing:**

Workflow:

1. Poll vposjournal XML every 30 seconds
2. Parse transaction XML (RetailTransaction)
3. Extract line items (Sale/MerchandiseCodeLine/FuelSale)
4. Calculate tax, discounts, promotions

**Transaction Data Captured:**
- Timestamp, sequence number, drawer ID
- Line items: PLU, description, quantity, price
- Tax breakdown (tax ID, rate, amount)
- Payment methods (cash, credit, EBT)
- Network data (Visa/MC/Amex/Discover)
- Fuel dispensed (grade, volume, PPG, pump #)
- Promotions, discounts, manual price overrides

### 6. Inventory Management System

**Real-Time Stock Tracking:**
- Automatic inventory updates on each sale
- Low stock alerts via email
- Product attributes:
    - currentStock, minStock, maxStock
    - itemID (ObjectId reference to priceBook)
    - Purchase history, last updated timestamp

**Vendor Management:**
- Vendor profiles with contact details
- Purchase order tracking
- Invoice reconciliation

**Purchase Reconciliation:**
- S3-based purchase data storage
- SKU-level inventory reconciliation
- Automated email alerts for discrepancies

### 7. Automated Reporting

**Report Types:**

| Report | Contents |
| --- | --- |
| Summary | Daily sales, payment modes, tax, discounts, totalizers |
| Department | Sales by department with tax breakdown |
| Tax | Tax breakdown |
| Fuel Dashboard | Cash vs credit, tank-wise, tier product stats |
| Network Product | Card network-wise sales (Visa/MC/Amex/Disc) |
| Tank-wise Sales | Tank-specific sales and usage |
| PLU Report | Product-level performance |

**Daily S3 Backups:**
- Automated daily JSON export to S3
- Folder structure: SalesDataJSON/{store_name}/{date}/
- Incremental backups with deduplication
- Retention: 90 days (configurable)

### 8. Label Printing & Barcode Generation

**Zebra ZPL Integration:**
**Features:**
- Auto-generated barcodes for custom products
- ZPL code generation for price labels
- Direct printing via win32print (Windows)
- Label format: Store name, product name, barcode, price
- Preview functionality before printing

### 9. Smart Notifications & Alerts

**Email Automation:**

| Alert Type | Trigger | Recipients |
| --- | --- | --- |
| Low Stock | currentStock < minStock | ownerGmail |
| ATG Alarms | Critical alarm detected | ownerGmail, admin |
| Stopped Nozzles | No sales in 24 hours | ownerGmail |
| Day Close Report | End of business day | ownerGmail |
| System Status	| Script failure/restart | development |
| Price Update | Fuel price change success | ownerGmail |

**SMTP Configuration:**

- Server: smtp.gmail.com:587
- Authentication: App passwords
- HTML email templates with inline CSS
- Retry logic with exponential backoff

### 10. Predictive Analytics & ML

**AWS SageMaker Canvas Integration:**
- Domain creation from filtered sales data
- Foundational predictive models
- Sales forecasting reports
- Demand prediction by product category

**Facebook Prophet Models:**
- Time series forecasting
- Seasonal trend analysis
- Holiday effects modeling

### 11. Configuration Management

**Store Setup (POS Configuration):**

**XML Datasets Synced:**
- vposcfg → Department configuration (discounts, promotions)
- vtaxratecfg → Tax rates and rules
- vMaintenance&dataset=Item → Product catalog (pricebook)
- vfuelcfg → Fuel grade configuration
- vMaintenance&dataset=MixMatch → Mix & match promotions
- vMaintenance&dataset=Combo → Combo pricing rules

**Automatic Sync Schedule:** Every 4 hours (configurable)

---

## Performance Optimizations

**Database Optimization**

**Connection Pool:**
- maxPoolSize: 60
- minPoolSize: 10
- connectTimeout: 60 seconds
- socketTimeout: 60 seconds

**Query Optimization:**
- ObjectId indexing on storeid, corpid
- Compound indexes on date + storeid
- Projection-based field filtering
- Batch inserts for transaction data

Background Job Efficiency

**Scheduled Tasks:**
- Cron jobs via schedule library
- Daemon threads for continuous monitoring
- Error isolation (try/except per job)
- Graceful shutdown handlers (SIGINT, SIGTERM)

**Polling Intervals:**
- Live sales: 30 seconds
- Tank status: 50 seconds
- Fuel price updates: 5 seconds (when active)
- ATG alarms: 60 seconds
- Daily inventory: Once per day (configurable time)

Logging Infrastructure

**Custom S3LogHandler:**
Features:
- Real-time log streaming to S3
- Buffer-based uploads (10KB or 60 seconds)
- Emergency uploads on critical errors
- Folder structure by activity type:
  - logs/activity/{store}/{date}/
  - logs/flowRate/{store}/{date}/
  - logs/emailTrigger/{store}/{date}/
  - logs/SalesDataJSON/{store}/{date}/

**Log Retention:** 90 days

Web Scraping Optimization

**Selenium Configuration:**
- Headless Chrome with GPU disabled
- No sandbox mode for Linux compatibility
- WebDriver Manager for automatic ChromeDriver updates
- Explicit waits (max 30s)
- BeautifulSoup for HTML parsing

---

## Technical Achievements

### 1. Legacy System Integration

- Seamless XML-to-JSON transformation pipeline
- Cookie-based authentication with 20+ year old POS systems
- Support for multiple POS vendors (Verifone, Gilbarco)
- Backward compatibility with various POS firmware versions

### 2. Real-Time Operations

- < 30 second latency for live sales updates
- Instant fuel price synchronization to dispensers
- Live tank monitoring with 50-second refresh
- Real-time inventory depletion tracking

### 3. Multi-Vendor ATG Support

- Unified interface for Franklin, Veeder Root, OPW
- Automated web scraping with retry logic
- Alarm normalization across different formats
- Compliance report automation (SIR, CSLD)

### 4. Scalability & Reliability

- Multi-store architecture with single codebase
- Automatic failover and reconnection logic
- Graceful error handling with detailed logging
- S3-based log aggregation for debugging

### 5. Operational Efficiency

- 90% reduction in manual tax reporting time
- Automated vendor compliance submissions
- Real-time low stock alerts prevent stockouts
- Predictive analytics enable proactive inventory management

### 6. Developer Experience

- Clean separation of concerns (Fuel/Retail/PetroView)
- Reusable MAC detection and store lookup functions
- Comprehensive error logging with context
- Environment-based configuration management

---

## System Metrics

API Response Times

- **POS Data Fetch:** < 2 seconds (XML parsing)
- **Tank Status Update:** < 60 seconds (web scraping)
- **Fuel Price Push:** < 30 seconds (end-to-end)
- **Sales Transaction Processing:** < 100ms (per transaction)
- **Report Generation:** < 5 seconds (daily summary)

Concurrent Operations

- **POS Polling:** 30-second intervals per store
- **MongoDB Connections:** 60 max pool size
- **Selenium Instances:** 1 per ATG vendor per store
- **S3 Uploads:** Asynchronous with 60-second batching
- **Email Alerts:** Throttled (1 per alert type per hour)

Data Processing

- **Transactions Per Day:** 5,000+ per store
- **XML Parsing:** 50+ datasets per store per day
- **Tank Readings:** 1,728 per tank per day (50s interval)
- **Log Volume:** ~500MB per store per month

---

## Environment Variables

```ini
# MongoDB Atlas
MONGODB_URI=mongodb+srv://petrolynks:<password>@petro.seraijq.mongodb.net/systems

# AWS S3
S3_BUCKET_NAME=petrolynks110
AWS_ACCESS_KEY_ID=<configured>
AWS_SECRET_ACCESS_KEY=<configured>
S3_REGION=us-east-1

# SMTP Email
EMAIL_ADDRESS=development@betasys.ai
EMAIL_PASSWORD=<app_password>
SMTP_SERVER=smtp.gmail.com
SMTP_PORT=587

# Store Identification (Auto-detected via MAC)
# No manual configuration required

# POS Credentials (Stored in MongoDB per store)
# Fetched dynamically via get_store_info()
```

---

## Dependencies

Core
```ini
fastapi==0.115.6
uvicorn[standard]==0.34.0
pymongo==4.10.1
requests==2.31.0
xmltodict==0.13.0
```

Web Automation
```ini
selenium==4.15.2
webdriver-manager==4.0.1
beautifulsoup4==4.12.2
```

Cloud Services
```ini
boto3==1.35.97
botocore==1.35.97
```

Utilities
```ini
schedule==1.2.0
pytz==2024.1
python-dateutil==2.8.2
dnspython==2.7.0
```

Windows Support (Label Printing)
```ini
pywin32==306  # For Zebra printer integration
```

Machine Learning (Optional)
```ini
prophet==1.1.5
sagemaker==2.200.0
```

---

## Major Functionalities

### 1. Live Sales Monitoring
- Real-time transaction tracking from POS journal files
- Line-item level detail capture (PLU, price, tax, discounts)
- Payment method breakdown (cash, credit cards, EBT)
- Network product sales (Visa, Mastercard, Amex, Discover)

### 2. Fuel Price Management
- Web-based price change interface
- Direct backend-to-dispenser synchronization
- Two-stage update process (prices + initialization)
- Automatic rollback on failure

### 3. Tank Monitoring Dashboard
- Real-time tank levels across all stores
- Temperature and water contamination alerts
- Ullage calculation for delivery planning
- Historical trend analysis

### 4. ATG Alarm System
- Centralized alarm dashboard for all stores
- Email digest with configurable batching
- Critical alarm immediate notifications
- Alarm history and audit trail

### 5. Compliance Reporting
- SIR (Statistical Inventory Reconciliation) reports
- CSLD (Continuous Statistical Leak Detection) diagnostics
- Daily inventory variance reports
- Delivery history tracking

### 6. Inventory Management
- Real-time stock updates on each sale
- Low stock email alerts
- Vendor management and purchase tracking
- SKU-level reconciliation

### 7. Automated Backups
- Daily JSON export to S3
- Incremental backups with deduplication
- 90-day retention policy
- Disaster recovery ready

### 8. Pricebook Synchronization
- Automatic product catalog sync from POS
- Custom product addition with barcode generation
- Zebra ZPL label printing integration
- Department and tax configuration management

### 9. Multi-Store Management
- MAC-based automatic store detection
- Centralized configuration in MongoDB
- Per-store POS credentials and ATG URLs
- Store owner email notification preferences

### 10. Predictive Analytics
- Sales forecasting with Facebook Prophet
- AWS SageMaker Canvas integration
- Demand prediction by product category
- Seasonal trend analysis

---

## Startup Sequence

### 1. Store Identification

1. Detect physical MAC addresses (getmac)
2. Query MongoDB for matching store document
3. Cache working MAC for future lookups
4. Extract: storeid, corpid, store_name, tankStatusUrl, ownerGmail

### 2. Database Connection

1. Initialize MongoDB Atlas client
2. Set connection pool parameters (maxPoolSize: 60)
3. Connect to 'systems' database
4. Initialize collection references

### 3. Background Services

1. Start sales monitoring script (sales.py)
2. Start tank status scraper (tank_status.py)
3. Start ATG alarm monitor (ATGAlarms.py)
4. Start daily inventory script (dailyInventory.py)
5. Start flow rate monitor (FRSN.py)
6. Start fuel price updater (update_fuel_price.py)
7. Start setup sync (setup.py) - runs every 4 hours

### 4. API Server

1. FastAPI app initialization (purchase.py)
2. CORS middleware configuration
3. Router registration
4. Uvicorn server start on port 8000

### 5. Log Handler Setup

1. Initialize S3LogHandler per script
2. Setup exit handlers (SIGINT, SIGTERM)
3. Configure log buffering and upload intervals

---

## Version & Maintainer

**Last Updated:** January 2026
**Version:** 2.1.0
**Maintained By:** Petrolynks Development Team

**Supported POS Systems:**
- Verifone Ruby CI / Topaz / Sapphire
- Gilbarco Passport

**Supported ATG Systems:**
- Franklin Fueling Systems TS-550 / EVO
- Veeder-Root TLS-350 / TLS-450
- OPW Integra 100

**Supported Store Types:**
- Fuel-only stations
- Convenience stores (grocery)
- Hybrid fuel + grocery locations

---
