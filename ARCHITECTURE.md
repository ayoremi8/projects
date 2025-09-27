# TickerIQ System Architecture Overview

## 1. System Overview
TickerIQ is a modular financial analytics platform designed for scalable, multi-agent research and analysis. The system integrates real-time data sources, advanced AI agents, and a modern web interface to deliver actionable insights for financial analysts and investors.

```mermaid
graph TB
    subgraph "Frontend Layer"
        UI[React Web Interface]
        WS[WebSocket Client]
    end
    
    subgraph "API Gateway"
        LB[Load Balancer]
        API[REST API Endpoints]
    end
    
    subgraph "Backend Services"
        BE[Flask Backend]
        ORCH[Agent Orchestrator]
    end
    
    subgraph "Multi-Agent System"
        RA[Research Agent]
        DA[Duplication Audit Agent]
        SA[Streaming Agent]
        ORA[Object Rendering Agent]
    end
    
    subgraph "Data Layer"
        DB[(PostgreSQL)]
        CACHE[(Redis Cache)]
        GCS[(Cloud Storage)]
    end
    
    subgraph "External APIs"
        AV[Alpha Vantage]
        SP[S&P Global]
        CSV[CSV Upload Service]
    end
    
    UI --> LB
    WS --> LB
    LB --> API
    API --> BE
    BE --> ORCH
    ORCH --> RA
    ORCH --> DA
    ORCH --> SA
    ORCH --> ORA
    BE --> DB
    BE --> CACHE
    BE --> GCS
    BE --> AV
    BE --> SP
    BE --> CSV
```

## 2. Core Services Architecture

```mermaid
graph LR
    subgraph "Frontend Services"
        FE[React Frontend]
        COMP[Components]
        STATE[State Management]
    end
    
    subgraph "Backend Services"
        FLASK[Flask Application]
        MOD[Modular Backend]
        AUTH[Authentication Service]
        DATA[Data Processing Service]
    end
    
    subgraph "Agent Services"
        AGENT_MGR[Agent Manager]
        TASK_Q[Task Queue]
        RESULT_PROC[Result Processor]
    end
    
    FE --> COMP
    FE --> STATE
    COMP --> FLASK
    STATE --> FLASK
    FLASK --> MOD
    MOD --> AUTH
    MOD --> DATA
    MOD --> AGENT_MGR
    AGENT_MGR --> TASK_Q
    TASK_Q --> RESULT_PROC
```

### Service Details:
- **Frontend (React/JS):** User interface for interacting with AI agents, visualizing data, and managing watchlists.
- **Backend (Python/Flask Modular):** Handles API requests, orchestrates multi-agent workflows, manages data pipelines, and serves as the integration point for external services.
- **Data Services:** Ingests and processes financial data from sources like Alpha Vantage, S&P Global, and custom CSV uploads.
- **AI Analysis Agents:** Modular agents for research, duplication audit, streaming analysis, and object rendering. Each agent operates independently and communicates via backend APIs.
- **Database:** Stores user data, watchlists, analysis results, and system logs (PostgreSQL recommended).

## 3. API Architecture

```mermaid
sequenceDiagram
    participant UI as Frontend UI
    participant API as API Gateway
    participant BE as Backend
    participant AGENT as AI Agents
    participant DB as Database
    participant EXT as External APIs

    UI->>API: POST /api/analysis/start
    API->>BE: Process Analysis Request
    BE->>DB: Store Analysis Job
    BE->>AGENT: Trigger Research Agent
    AGENT->>EXT: Fetch Market Data
    EXT-->>AGENT: Return Data
    AGENT->>AGENT: Process & Analyze
    AGENT->>BE: Send Results
    BE->>DB: Store Results
    BE-->>UI: WebSocket Update
    UI->>API: GET /api/analysis/results
    API->>BE: Fetch Results
    BE->>DB: Query Results
    DB-->>BE: Return Data
    BE-->>API: Results Response
    API-->>UI: Analysis Results
```

### API Endpoints:
- **RESTful APIs:**
  - `/api/analysis`: Trigger and retrieve AI analysis results.
  - `/api/data`: Manage financial data sources and uploads.
  - `/api/agents`: Orchestrate multi-agent workflows and status.
  - `/api/watchlists`: CRUD operations for user watchlists.
- **Streaming APIs:**
  - WebSocket endpoints for real-time updates and streaming analysis results.

## 4. Multi-Agent Architecture

```mermaid
graph TB
    subgraph "Agent Orchestration Layer"
        ORCH[Agent Orchestrator]
        QUEUE[Task Queue]
        SCHED[Task Scheduler]
    end
    
    subgraph "Research Agents"
        RA1[Research Agent 1]
        RA2[Research Agent 2]
        RA3[Research Agent N...]
    end
    
    subgraph "Specialized Agents"
        DA[Duplication Audit Agent]
        SA[Streaming Agent]
        ORA[Object Rendering Agent]
        NEWS[News Analysis Agent]
    end
    
    subgraph "Agent Communication"
        MSG[Message Bus]
        STATE[Shared State Store]
    end
    
    ORCH --> QUEUE
    QUEUE --> SCHED
    SCHED --> RA1
    SCHED --> RA2
    SCHED --> RA3
    SCHED --> DA
    SCHED --> SA
    SCHED --> ORA
    SCHED --> NEWS
    
    RA1 --> MSG
    RA2 --> MSG
    DA --> MSG
    SA --> MSG
    ORA --> MSG
    NEWS --> MSG
    
    MSG --> STATE
```

### Agent Communication Flow:
```mermaid
sequenceDiagram
    participant ORCH as Orchestrator
    participant RA as Research Agent
    participant DA as Duplication Agent
    participant SA as Streaming Agent
    participant STATE as Shared State

    ORCH->>RA: Start Analysis Task
    RA->>STATE: Update Status: Processing
    RA->>RA: Perform Research
    RA->>DA: Check for Duplicates
    DA->>STATE: Query Previous Analysis
    STATE-->>DA: Return Results
    DA-->>RA: Duplicate Status
    RA->>SA: Stream Interim Results
    SA->>STATE: Update Stream State
    RA->>ORCH: Final Results
    ORCH->>STATE: Store Final Results
```

### Agent Details:
- **Agent Types:**
  - Research Agent: Performs deep financial analysis and generates reports.
  - Duplication Audit Agent: Detects and resolves data duplication issues.
  - Streaming Agent: Provides real-time data analysis and UI updates.
  - Object Rendering Agent: Renders complex financial objects and charts.
- **Orchestration:**
  - Agents are orchestrated by the backend, which manages agent lifecycles, task assignment, and inter-agent communication.
  - Agents can be scaled independently for performance and reliability.

## 5. Google Cloud Deployment Architecture

```mermaid
graph TB
    subgraph "Google Cloud Platform"
        subgraph "Frontend Tier"
            CDN[Cloud CDN]
            LB[Load Balancer]
            STATIC[Cloud Storage - Static Assets]
        end
        
        subgraph "Application Tier"
            CR[Cloud Run Services]
            GKE[GKE Cluster - Agents]
            FUNC[Cloud Functions]
        end
        
        subgraph "Data Tier"
            SQL[Cloud SQL - PostgreSQL]
            REDIS[Memorystore - Redis]
            GCS[Cloud Storage - Data]
            BQ[BigQuery - Analytics]
        end
        
        subgraph "Infrastructure Services"
            IAM[Identity & Access Management]
            SM[Secret Manager]
            MON[Cloud Monitoring]
            LOG[Cloud Logging]
        end
    end
    
    subgraph "External Services"
        GITHUB[GitHub Actions CI/CD]
        APIS[External Financial APIs]
    end
    
    CDN --> LB
    LB --> CR
    CR --> GKE
    CR --> SQL
    CR --> REDIS
    CR --> GCS
    GKE --> SQL
    GKE --> REDIS
    SQL --> BQ
    
    GITHUB --> CR
    CR --> APIS
    
    IAM --> CR
    IAM --> GKE
    SM --> CR
    SM --> GKE
    MON --> CR
    MON --> GKE
    LOG --> CR
    LOG --> GKE
```

### Deployment Infrastructure:
```mermaid
graph LR
    subgraph "Development"
        DEV[Local Development]
        TEST[Unit Tests]
    end
    
    subgraph "CI/CD Pipeline"
        BUILD[GitHub Actions Build]
        DOCKER[Container Build]
        DEPLOY[Cloud Deploy]
    end
    
    subgraph "Environments"
        STAGING[Staging Environment]
        PROD[Production Environment]
    end
    
    DEV --> BUILD
    TEST --> BUILD
    BUILD --> DOCKER
    DOCKER --> DEPLOY
    DEPLOY --> STAGING
    STAGING --> PROD
```

### Cloud Services Details:
- **Compute:**
  - Google Cloud Run for stateless backend services.
  - Google Kubernetes Engine (GKE) for scalable agent orchestration.
- **Storage:**
  - Google Cloud SQL for relational data.
  - Google Cloud Storage for large datasets and CSV uploads.
  - BigQuery for analytics and reporting.
- **Networking:**
  - Google Cloud Load Balancer for traffic management.
  - VPC for secure internal communication between services.
- **CI/CD:**
  - GitHub Actions for automated build, test, and deployment pipelines.
  - Integration with Google Cloud Build for containerization and deployment.

## 6. Security & Monitoring Architecture

```mermaid
graph TB
    subgraph "Security Layer"
        subgraph "Authentication"
            OAUTH[OAuth2/Google Identity]
            JWT[JWT Tokens]
            RBAC[Role-Based Access Control]
        end
        
        subgraph "Network Security"
            WAF[Web Application Firewall]
            VPC[Private VPC Network]
            SSL[SSL/TLS Encryption]
        end
        
        subgraph "Data Security"
            ENCRYPT[Data Encryption at Rest]
            SM[Secret Manager]
            AUDIT[Audit Logging]
        end
    end
    
    subgraph "Monitoring & Observability"
        subgraph "Application Monitoring"
            APM[Application Performance Monitoring]
            TRACES[Distributed Tracing]
            METRICS[Custom Metrics]
        end
        
        subgraph "Infrastructure Monitoring"
            INFRA[Infrastructure Monitoring]
            ALERTS[Alerting & Notifications]
            DASH[Monitoring Dashboards]
        end
        
        subgraph "Logging"
            CENTRALIZED[Centralized Logging]
            LOG_ANALYSIS[Log Analysis]
            ERROR_TRACKING[Error Tracking]
        end
    end
    
    OAUTH --> JWT
    JWT --> RBAC
    WAF --> VPC
    VPC --> SSL
    ENCRYPT --> SM
    SM --> AUDIT
    
    APM --> TRACES
    TRACES --> METRICS
    INFRA --> ALERTS
    ALERTS --> DASH
    CENTRALIZED --> LOG_ANALYSIS
    LOG_ANALYSIS --> ERROR_TRACKING
```

### Data Flow & Security Model:
```mermaid
sequenceDiagram
    participant USER as User
    participant WAF as Web App Firewall
    participant LB as Load Balancer
    participant AUTH as Auth Service
    participant API as API Gateway
    participant AGENTS as AI Agents
    participant DB as Secure Database

    USER->>WAF: HTTPS Request
    WAF->>LB: Filtered Request
    LB->>AUTH: Validate Token
    AUTH-->>LB: Token Valid
    LB->>API: Authorized Request
    API->>AGENTS: Process Request
    AGENTS->>DB: Encrypted Query
    DB-->>AGENTS: Encrypted Response
    AGENTS-->>API: Processed Result
    API-->>LB: Response
    LB-->>WAF: Secure Response
    WAF-->>USER: HTTPS Response
```

### Security & Monitoring Details:
- **Authentication:** OAuth2/Google Identity for user login with JWT token management.
- **Monitoring:** Google Cloud Monitoring and Logging for system health and audit trails.
- **Secrets Management:** Google Secret Manager for API keys and sensitive credentials.
- **Security Features:**
  - Web Application Firewall (WAF) for request filtering
  - Private VPC networks for service isolation
  - Encryption at rest and in transit
  - Role-based access control (RBAC)
  - Comprehensive audit logging

## 7. Data Architecture & Flow

```mermaid
graph TB
    subgraph "Data Sources"
        AV[Alpha Vantage API]
        SP[S&P Global API]
        CSV[CSV Upload Service]
        NEWS[News APIs]
        SOCIAL[Social Media APIs]
    end
    
    subgraph "Data Ingestion Layer"
        ETL[ETL Pipeline]
        VALID[Data Validation]
        CLEAN[Data Cleaning]
        NORM[Data Normalization]
    end
    
    subgraph "Data Storage Layer"
        RAW[Raw Data Storage]
        PROCESSED[Processed Data Lake]
        CACHE[Redis Cache]
        DB[PostgreSQL Database]
    end
    
    subgraph "Data Processing Layer"
        BATCH[Batch Processing]
        STREAM[Stream Processing]
        ML[ML Feature Engineering]
        ANALYTICS[Analytics Engine]
    end
    
    subgraph "Data Access Layer"
        API[Data APIs]
        QUERY[Query Engine]
        EXPORT[Data Export]
    end
    
    AV --> ETL
    SP --> ETL
    CSV --> ETL
    NEWS --> ETL
    SOCIAL --> ETL
    
    ETL --> VALID
    VALID --> CLEAN
    CLEAN --> NORM
    
    NORM --> RAW
    RAW --> PROCESSED
    PROCESSED --> CACHE
    PROCESSED --> DB
    
    PROCESSED --> BATCH
    PROCESSED --> STREAM
    BATCH --> ML
    STREAM --> ML
    ML --> ANALYTICS
    
    DB --> API
    CACHE --> API
    ANALYTICS --> QUERY
    QUERY --> EXPORT
```

---

## 8. System Performance & Scalability

```mermaid
graph LR
    subgraph "Load Balancing"
        LB[Load Balancer]
        CDN[Content Delivery Network]
    end
    
    subgraph "Auto Scaling"
        HPA[Horizontal Pod Autoscaler]
        VPA[Vertical Pod Autoscaler]
        CA[Cluster Autoscaler]
    end
    
    subgraph "Caching Strategy"
        L1[L1: Browser Cache]
        L2[L2: CDN Cache]
        L3[L3: Application Cache]
        L4[L4: Database Cache]
    end
    
    subgraph "Performance Monitoring"
        METRICS[Performance Metrics]
        SLO[Service Level Objectives]
        ALERTS[Performance Alerts]
    end
    
    LB --> CDN
    HPA --> VPA
    VPA --> CA
    L1 --> L2
    L2 --> L3
    L3 --> L4
    METRICS --> SLO
    SLO --> ALERTS
```

---
For more details, see individual service documentation and deployment guides in the `/docs` and `/cloud-setup.sh` files.