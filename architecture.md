# NeuGraph System Architecture and Workflows

This document provides a comprehensive overview of the NeuGraph system architecture, detailing the components, their interactions, and the end-to-end workflows for data processing, analysis, and user interaction.

## 1. Unified System Architecture

This diagram illustrates the multi-layered architecture of the NeuGraph platform, from data ingestion to the user-facing application.

```mermaid
graph TD
    subgraph "Data Sources"
        A[Public Databases: PubMed, Uniprot]
        B[Internal LIMS/ELN Systems]
        C[Proprietary Research Data]
    end

    subgraph "Data Ingestion & ETL (n8n & Python)"
        D[ETL Workflows: n8n]
        E[Custom Python Scripts]
        F[Data Validation & Cleaning]
        G2[Real-time Streaming: Kafka]
        H2[Change Data Capture CDC]
        D -- Manages --> E
        A -- Processed by --> D
        B -- Processed by --> D
        C -- Processed by --> E
        E -- Feeds into --> F
        G2 -- Streams to --> F
        H2 -- Triggers --> G2
    end

    subgraph "Graph Construction Layer"
        G[Graph Database: AWS Neptune]
        H[Schema & Ontology Management]
        F -- Populates --> G
        H -- Defines structure of --> G
    end

    subgraph "AI & Analytics Layer"
        I[AI Reasoning Engine: AWS Bedrock - Llama 3.1]
        J[Graph Machine Learning: GNNs]
        K[LlamaIndex for RAG]
        L[Vector Database: Pinecone/Weaviate]
        M[Multi-Agent AI System]
        G -- Queried by --> I
        G -- Analyzed by --> J
        I -- Uses --> K
        L -- Enhances --> K
        M -- Orchestrates --> I
    end

    subgraph "Application & API Layer"
        L[Backend API: FastAPI]
        M[Governance Engine]
        N[User Management & Auth]
        I -- Exposed via --> L
        J -- Exposed via --> L
        G -- Accessed via --> L
        M -- Enforces policies on --> L
        N -- Secures --> L
    end

    subgraph "User Interface (UI) Layer"
        O[Frontend Framework: Streamlit/Dash]
        P[Graph Visualization Library]
        Q[AI Chat Interface]
        R[Dashboard & Analytics View]
        L -- Serves --> O
        O -- Contains --> P
        O -- Contains --> Q
        O -- Contains --> R
    end

    subgraph "Cross-Cutting Concerns"
        S[Data Governance & Compliance]
        T[Monitoring & Logging]
        M -- Implements --> S
        L -- Integrates --> T
    end
```

### 2.2. AI-Powered Query Workflow

This sequence shows how a user's natural language query is processed by the system to return a governed, contextualized answer.

```mermaid
sequenceDiagram
    participant U as 👤 User
    participant UI as 🖥️ NeuGraph UI
    participant API as 🔌 FastAPI Backend
    participant GOV as 🛡️ Governance Engine
    participant AI as 🤖 AWS Bedrock (Llama 3.1)
    participant DB as 🏛️ AWS Neptune

    U->>UI: "Find genes related to Alzheimer's"
    UI->>API: Send query + user context
    API->>GOV: 1. Check user permissions
    GOV-->>API: User clearance level (e.g., INTERNAL)
    API->>AI: 2. Generate graph query from NL
    AI-->>API: Cypher/Gremlin query: "MATCH (g:Gene)-[:ASSOCIATED_WITH]->(d:Disease {name:'Alzheimer\'s'}) RETURN g"
    API->>GOV: 3. Apply access filters to query
    GOV-->>API: Filtered query (adds data classification checks)
    API->>DB: 4. Execute filtered query
    DB-->>API: Raw graph results
    API->>GOV: 5. Filter results by classification
    GOV-->>API: Authorized results (e.g., only PUBLIC, INTERNAL data)
    API->>AI: 6. Generate natural language explanation
    AI-->>API: "Results..."
    API-->>UI: 7. Display response with governance notes
```

### 2.3. UI Interaction Flow

This diagram outlines the primary navigation paths and interactions within the NeuGraph user interface.

```mermaid
flowchart LR
    subgraph "User Interaction Flow"
        A["👤 User Login"] --> B["🛡️ Authentication"]
        B --> C["📊 Dashboard"]

        C --> D["🕸️ Graph Explorer"]
        C --> E["🤖 AI Assistant"]
        C --> F["🛡️ Governance Panel"]

        D --> G["🎨 Visual Graph Exploration"]
        E --> H["💬 Natural Language Queries"]
        F --> I["👥 Access Control Management"]

        G --> J["📄 Node Details & Lineage"]
        H --> K["🔍 AI-Generated Results"]
        I --> L["📋 Compliance Reports & Audits"]
    end

    subgraph "Backend Services"
        P["🏛️ Graph Database"]
        Q["🤖 AI Engine"]
        R["🛡️ Governance Service"]

        G --> P
        H --> Q
        I --> R
    end
```

## 3. Advanced Technical Capabilities

### 3.1. Real-Time Data Processing

NeuGraph supports real-time data ingestion and processing through Apache Kafka streaming architecture.

```mermaid
graph LR
    subgraph "Real-Time Pipeline"
        A[Data Sources] --> B[Kafka Producer]
        B --> C[Kafka Topics]
        C --> D[Stream Processor]
        D --> E[Graph Database]
        D --> F[Vector Database]
    end
```
---

### Explanation

- **Data Sources:** Both external (literature, trials, regulatory) and internal (operational, warehouse, images).
- **Data Mesh & Connectors:** Data is accessed in place via APIs or ETL, indexed and enriched with metadata.
- **Semantic Layer:** Ontologies and a knowledge graph provide structure, context, and validation (SHACL). Metadata repository links everything.
- **AI/LLM Layer:** Semantic search and LLMs work together (RAG) to answer questions, generate content, and provide context-aware results. Human-in-the-loop ensures quality and curation.
- **User & Analytics:** Users interact through search UIs, dashboards, document generation, and collaboration tools. Full traceability and provenance are provided for every answer or document.
- **Feedback Loops:** User feedback and curation update the ontology/knowledge graph and improve the system iteratively.

---