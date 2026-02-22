# Enterprise Architecture Patterns Repository

Welcome to a curated collection of **production-grade enterprise architecture patterns** for building scalable, resilient data platforms. Each pattern demonstrates proven design decisions, trade-offs, and implementation strategies used in real-world systems handling massive scale, cloud migrations, and complex data workflows.

## 📚 Table of Contents

1. [Distributed Data Ingestion](#1-distributed-data-ingestion)
2. [Content Discovery & Personalization](#2-content-discovery--personalization)
3. [Legacy Monolith to Microservices](#3-legacy-monolith-to-microservices)
4. [Secure Local AI Modernization](#4-secure-local-ai-modernization)
5. [Architecture Principles](#architecture-principles)

---

## Patterns Overview

### 1. Distributed Data Ingestion

**Use Case:** Building a high-throughput, resilient data pipeline to safely ingest unstructured data from heterogeneous external sources into a secure, controlled environment.

**Key Characteristics:**
- Distributed worker nodes for parallel extraction
- Queue-based decoupling for horizontal scalability
- Zero-trust VPC security with mTLS
- Real-time search + historical auditing dual-write pattern

[→ Explore Pattern →](./distributed-crawlers/README.md)

```mermaid
flowchart LR
    WEB[External Sources] -->|Discovery| DISC[Discovery Service]
    DISC -->|URLs| IQ[Input Queue]
    IQ -->|Distribute| Workers[Worker Cluster]
    Workers -->|Standardize| OQ[Output Queue]
    OQ -->|Dual-Write| OS[OpenSearch]
    OQ -->|Dual-Write| MONGO[MongoDB]
    
    style DISC fill:#ede7f6
    style Workers fill:#ede7f6
    style IQ fill:#fff3e0
    style OQ fill:#fff3e0
    style OS fill:#e8f5e9
    style MONGO fill:#e8f5e9
    style WEB fill:#e1f5fe
```

---

### 2. Content Discovery & Personalization

**Use Case:** Building a client-facing platform that serves millions of entities with hybrid discovery (rule-based + ML) and real-time personalization at global scale.

**Key Characteristics:**
- Hybrid search engine (rule-based + ML recommendations)
- Dynamic localization at the serving layer
- Closed-loop telemetry for continuous model tuning
- CDN + SPA frontend for sub-second global loads

[→ Explore Pattern →](./content-discovery-and-personalization/README.md)

```mermaid
flowchart LR
    USER[End Users] -->|Global| CDN[CDN]
    CDN -->|API Calls| GW[API Gateway]
    GW -->|Route| SEARCH[Search Service]
    GW -->|Route| ML[ML Service]
    SEARCH -->|Query| OS[OpenSearch]
    ML -->|User Data| MONGO[MongoDB]
    
    style SEARCH fill:#ede7f6
    style ML fill:#ede7f6
    style OS fill:#e8f5e9
    style MONGO fill:#e8f5e9
    style CDN fill:#fff9c4
    style GW fill:#fff3e0
    style USER fill:#e1f5fe
```

---

### 3. Legacy Monolith to Microservices

**Use Case:** Refactoring a legacy computational monolith into a highly scalable, event-driven microservices architecture on AWS. Details parallel compute orchestration, API-first design, and cloud cost-optimization.

[→ Explore Pattern →](./legacy-monolith-to-microservices/README.md)

```mermaid
flowchart LR
    UI1[Setup UI]
    UI2[Canvas UI]
    UI3[Analytics UI]
    GW[API Gateway]
    SVC1[Job Mgmt Service]
    SVC2[Render Service]
    SVC3[Aggregation Service]
    PULSAR[Apache Pulsar]
    L1[Lambda Worker]
    L2[Lambda Worker]
    
    UI1 --> GW
    UI2 --> GW
    UI3 --> GW
    GW --> SVC1 & SVC2 & SVC3
    SVC1 -->|Publish| PULSAR
    PULSAR -->|Stream| L1 & L2
    L1 & L2 -->|Results| PULSAR
    PULSAR -->|Consume| SVC3
    
    style SVC1 fill:#ede7f6
    style SVC2 fill:#ede7f6
    style SVC3 fill:#ede7f6
    style PULSAR fill:#e0f7fa
    style L1 fill:#fce4ec
    style L2 fill:#fce4ec
    style GW fill:#fff3e0
    style UI1 fill:#fff9c4
    style UI2 fill:#fff9c4
    style UI3 fill:#fff9c4
```

---

### 4. Secure Local AI Modernization

**Use Case:** Modernizing legacy desktop engineering applications into secure, multi-tenant Python web platforms with air-gapped, local GenAI pipelines for multi-modal analysis without exposing proprietary IP to public cloud APIs.

[→ Explore Pattern →](./secure-local-ai-modernization/README.md)

```mermaid
flowchart LR
    USER[End User]
    UI[Web UI]
    API[API Gateway]
    AUTH[Auth Service]
    MATH[Math Service]
    VIZ[Visualization]
    QUEUE[Task Queue]
    WORKER[Background Worker]
    STATS[Context Aggregator]
    OLLAMA[Ollama LLM]
    
    USER -->|Access| UI
    UI --> API
    API --> AUTH & MATH & VIZ
    MATH -->|Enqueue| QUEUE
    QUEUE -->|Consume| WORKER
    WORKER -->|Results| VIZ
    VIZ -->|Charts| STATS
    STATS -->|Prompt| OLLAMA
    OLLAMA -->|Analysis| UI
    
    style AUTH fill:#ede7f6
    style MATH fill:#ede7f6
    style VIZ fill:#ede7f6
    style QUEUE fill:#e0f7fa
    style WORKER fill:#fce4ec
    style STATS fill:#e8f5e9
    style OLLAMA fill:#e8f5e9
    style API fill:#fff3e0
    style UI fill:#fff9c4
    style USER fill:#e1f5fe
```

---

## Architecture Principles

These patterns are built on five core principles:

### 1. **Decoupling via Message Queues**
Services communicate asynchronously to enable independent scaling and failure isolation.

### 2. **Dual-Write Patterns**
Data flows to both operational stores (for low-latency queries) and stateful stores (for historical analysis) simultaneously.

### 3. **Security by Isolation**
Services are deployed in private subnets within secure VPCs with strict Bastion/API Gateway ingress.

### 4. **Microservices with Clear Boundaries**
Each service has a single, bounded responsibility enabling independent deployment.

### 5. **Closed-Loop Feedback**
Telemetry and user signals feedback into the system for continuous model retraining.

---

## Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Compute** | AWS EC2, Docker | Scalable compute and containerized microservices |
| **Messaging** | RabbitMQ / AWS SQS | Asynchronous decoupling and work distribution |
| **Search & DB** | AWS OpenSearch, MongoDB | Real-time vector/text search and write-optimized temporal storage |
| **AI / ML** | LangChain, pgvector, LLMs | Multi-modal intelligence and recommendation engines |
| **Delivery** | CloudFront CDN, Next.js | Global content delivery and decoupled UI |

---

## Real-World Applicability

These patterns are derived from production systems and high-fidelity architectural prototypes handling:

- 10+ million entities indexed and served globally
- High-throughput, event-driven pipelines processing millions of unstructured data points.
- Sub-second latency hybrid search requirements
- Legacy-to-Cloud modernization deployments

---

**Last Updated:** February 2026  
**Author:** Bharanidharan Hemachandran
