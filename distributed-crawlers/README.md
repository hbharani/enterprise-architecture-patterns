# Distributed High-Throughput Data Ingestion Engine

## Architecture Overview
This repository details the architecture for a highly scalable, distributed data ingestion pipeline deployed entirely within a secure AWS VPC. The system is designed to independently discover, rate-limit, and extract unstructured data from highly variable external web architectures, standardize the payloads, and route them for both real-time search and historical state tracking.

```mermaid
graph TD
    subgraph Public Internet
        T1[Target Architecture Type A]
        T2[Target Architecture Type B]
        T3[Custom Enterprise Targets]
    end

    subgraph AWS VPC [AWS Virtual Private Cloud - Private Subnets]
        direction TB
        
        BASTION[EC2 Bastion Host<br/>Secure Admin Tunneling]
        
        subgraph Discovery Layer
            DISC[Discovery Microservice<br/>Parses Sitemaps & robots.txt]
        end

        subgraph Queueing & Decoupling Layer
            IQ[(Input Queue<br/>URL Dispatch)]
            OQ[(Output Queue<br/>Standardized Payloads)]
        end

        subgraph Distributed Crawler Cluster
            C1[Scrapy Worker Node A<br/>Rate-Limited]
            C2[Scrapy Worker Node B<br/>Rate-Limited]
            C3[Custom Scrapy Node<br/>Rate-Limited]
        end

        subgraph Data Storage & Search
            OS[(AWS OpenSearch<br/>Real-Time Index)]
            
            subgraph Sharded Database Cluster
                MONGO[(MongoDB Sharded Cluster<br/>Historical State Tracking)]
                CERT[mTLS / Self-Signed Certs<br/>App-Specific Authentication]
                MONGO --- CERT
            end
        end
    end

    %% The Flow
    DISC -->|Crawls site structures| T1
    DISC -->|Crawls site structures| T2
    DISC -->|Crawls site structures| T3
    
    DISC -->|Publishes Target URLs| IQ
    
    IQ -->|Consumes URLs| C1
    IQ -->|Consumes URLs| C2
    IQ -->|Consumes URLs| C3
    
    C1 -->|Ingests & Standardizes Data| T1
    C2 -->|Ingests & Standardizes Data| T2
    C3 -->|Ingests & Standardizes Data| T3
    
    C1 -->|Publishes Extracted Payloads| OQ
    C2 -->|Publishes Extracted Payloads| OQ
    C3 -->|Publishes Extracted Payloads| OQ
    
    OQ -->|Syncs Search Index| OS
    OQ -->|Appends Change History| MONGO
    
    %% Security & Infrastructure Rules
    BASTION -.->|Encrypted SSH Access| DISC
    BASTION -.->|Encrypted SSH Access| MONGO
    
    classDef vpc fill:#f9f2ec,stroke:#d28e5d,stroke-width:2px;
    classDef internet fill:#
```
### Key Engineering Decisions & Trade-offs

- **Asynchronous Decoupling:** The Discovery Microservice (handling robots.txt and sitemap traversal) is strictly decoupled from the worker nodes via an Input Queue. This allows the extraction layer to scale horizontally based on queue depth without overwhelming target servers.

- **Write-Optimized Storage:** Because the system tracks entity mutations over time (historical change tracking), a fully sharded MongoDB cluster was deployed to handle the massive write-throughput, rather than a traditional relational database.

- **Zero-Trust VPC Security:** The entire database and queueing architecture is locked within private subnets with zero external ingress. Administration is strictly routed through an EC2 Bastion host, and database access is secured via mutual TLS (mTLS) using application-specific, self-signed certificates for granular access control.

- **Dual-Write Output Routing:** The Output Queue fans out standardized data. It synchronizes the latest state into AWS OpenSearch for sub-second retrieval, while simultaneously appending the temporal data to MongoDB for historical auditing and longitudinal analysis.