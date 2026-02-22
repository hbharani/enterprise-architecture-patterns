# Scalable Content Discovery & Personalization Platform

## Architecture Overview

This architecture defines the client-facing presentation and data-serving layer. Deployed entirely on AWS EC2 instances, this microservices ecosystem is designed to serve millions of indexed entities to global end-users. It features a hybrid discovery model (combining rule-based search with ML-driven recommendations), dynamic real-time data localization, and a closed-loop telemetry system to continuously tune user profiles and data quality.

```mermaid
---
config:
  layout: elk
---
flowchart TB
 subgraph ComputeLayer["Microservices Cluster - EC2"]
        FEED["Telemetry & Feedback Engine"]
        AUTH["Identity & Profile Service"]
        LIFE["Entity Lifecycle Service"]
        ML["ML Recommendation Engine"]
        LOC["Dynamic Localization Service"]
        SEARCH["Rule-Based Search Engine"]
        TAX["Dynamic Taxonomy Service"]
  end
 subgraph DataLayer["Data Storage Cluster - EC2"]
        MONGO[("MongoDB Cluster<br>Profiles &amp; Telemetry")]
        OS[("AWS OpenSearch<br>Indexed Entities")]
  end
 subgraph VPC["AWS VPC - Application Serving Layer"]
        GW{"API Gateway / Routing"}
        ComputeLayer
        DataLayer
  end
    CLIENT(("Client Applications<br>Web / Mobile")) -- API Requests --> GW
    GW --> TAX & SEARCH & ML & LIFE & AUTH & FEED
    LOC -. Daily Async Pull .-> EXT_API["External Metric/Rate API"]
    SEARCH -. Request Conversion .-> LOC
    TAX -- Aggregate Metadata --> OS
    SEARCH -- "Full-Text Queries" --> OS
    ML -- Vector Search --> OS
    ML -- Read Preferences --> MONGO
    LIFE -- Delete Entities --> OS
    LIFE -- Update Status --> MONGO
    AUTH -- Read/Write Session --> MONGO
    FEED -- Write Engagement --> MONGO
    TAX ~~~ ML
    ML ~~~ AUTH
    OS ~~~ MONGO

     CLIENT:::client
     EXT_API:::ext
     GW:::gw
     TAX:::compute
     SEARCH:::compute
     LOC:::compute
     ML:::compute
     LIFE:::compute
     AUTH:::compute
     FEED:::compute
     OS:::db
     MONGO:::db
    classDef client fill:#e1f5fe,stroke:#0288d1,stroke-width:2px,color:#000
    classDef compute fill:#ede7f6,stroke:#512da8,stroke-width:2px,color:#000
    classDef db fill:#e8f5e9,stroke:#388e3c,stroke-width:2px,color:#000
    classDef ext fill:#f5f5f5,stroke:#9e9e9e,stroke-width:2px,color:#000
    classDef gw fill:#fff3e0,stroke:#f57c00,stroke-width:2px,color:#000
```

### Key Engineering Decisions & Trade-offs

- **Hybrid Discovery Engine:** Balances strict user intent with serendipitous discovery via a dual-engine approach. Rule-based search queries OpenSearch for exact matches and filters, while the ML recommendation engine cross-references user profiles in MongoDB against entity vectors for personalized suggestions.

- **Abstracted Localization (Microservice Pattern):** A dedicated Dynamic Localization Service pulls daily exchange/metric rates from external APIs and provides runtime conversions to the Search Service. This keeps the OpenSearch index standardized and avoids storing localized variants.

- **Automated Entity Lifecycle (Tombstoning):** The Entity Lifecycle Service listens for deprecation signals and synchronously removes or tombstones entries in OpenSearch and flags records in MongoDB to prevent stale results.

- **Closed-Loop Telemetry:** The Telemetry & Feedback Engine captures explicit user feedback (comments, ratings, reward interactions) and writes it to MongoDB. This data is used to retrain ML models, creating an automated improvement loop.