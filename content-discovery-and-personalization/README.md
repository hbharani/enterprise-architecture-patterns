# Scalable Content Discovery & Personalization Platform

## Architecture Overview

This architecture defines the client-facing presentation and data-serving layer. Deployed entirely on AWS EC2 instances, this microservices ecosystem is designed to serve millions of indexed entities to global end-users. It features a hybrid discovery model (combining rule-based search with ML-driven recommendations), dynamic real-time data localization, and a closed-loop telemetry system to continuously tune user profiles and data quality.

```mermaid
---
config:
  layout: elk
---
flowchart TD
    classDef edge fill:#fff9c4,stroke:#fbc02d,stroke-width:2px,color:#000
    classDef client fill:#e1f5fe,stroke:#0288d1,stroke-width:2px,color:#000
    classDef compute fill:#ede7f6,stroke:#512da8,stroke-width:2px,color:#000
    classDef db fill:#e8f5e9,stroke:#388e3c,stroke-width:2px,color:#000
    classDef gw fill:#fff3e0,stroke:#f57c00,stroke-width:2px,color:#000

    USER((End User)):::client

    subgraph EdgeLayer [Edge & Delivery Layer]
        CDN[CloudFront / CDN<br/>Static Assets]:::edge
        WUI[Web Interface<br/>Next.js / SPA]:::edge
    end

    subgraph VPC [AWS VPC - Application Serving Layer]
        
        GW{API Gateway / Routing}:::gw

        subgraph ComputeLayer [Microservices Cluster - EC2]
            TAX[Dynamic Taxonomy Service]:::compute
            SEARCH[Rule-Based Search Engine]:::compute
            LOC[Dynamic Localization Service]:::compute
            ML[ML Recommendation Engine]:::compute
            LIFE[Entity Lifecycle Service]:::compute
            AUTH[Identity & Profile Service]:::compute
            FEED[Telemetry & Feedback Engine]:::compute
        end

        subgraph DataLayer [Data Storage Cluster - EC2]
            OS[(AWS OpenSearch<br/>Indexed Entities)]:::db
            MONGO[(MongoDB Cluster<br/>Profiles & Telemetry)]:::db
        end
    end
    USER -->|1. Request Site| CDN
    CDN -->|2. Serve SPA| WUI
    WUI -->|3. Authenticated API Calls| GW
    GW --> TAX & SEARCH & ML & AUTH & FEED
    SEARCH -.->|Currency/Metric Conversion| LOC
    
    SEARCH -->|Query| OS
    TAX -->|Aggregates| OS
    ML -->|Vector Match| OS
    ML -->|User History| MONGO
    
    AUTH -->|Session Management| MONGO
    FEED -->|Telemetry Stream| MONGO
    LIFE -->|Index Cleanup| OS
    LIFE -->|State Sync| MONGO
    WUI ~~~ GW
    TAX ~~~ ML ~~~ AUTH
    OS ~~~ MONGO
```

### Key Engineering Decisions & Trade-offs

- **Hybrid Discovery Engine:** Balances strict user intent with serendipitous discovery via a dual-engine approach. Rule-based search queries OpenSearch for exact matches and filters, while the ML recommendation engine cross-references user profiles in MongoDB against entity vectors for personalized suggestions.

- **Abstracted Localization (Microservice Pattern):** A dedicated Dynamic Localization Service pulls daily exchange/metric rates from external APIs and provides runtime conversions to the Search Service. This keeps the OpenSearch index standardized and avoids storing localized variants.

- **Automated Entity Lifecycle (Tombstoning):** The Entity Lifecycle Service listens for deprecation signals and synchronously removes or tombstones entries in OpenSearch and flags records in MongoDB to prevent stale results.

- **Closed-Loop Telemetry:** The Telemetry & Feedback Engine captures explicit user feedback (comments, ratings, reward interactions) and writes it to MongoDB. This data is used to retrain ML models, creating an automated improvement loop.