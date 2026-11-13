flowchart TB
    %% Styling
    classDef client fill:#f9f9f9,stroke:#333,stroke-width:2px;
    classDef api fill:#e1f5fe,stroke:#0288d1,stroke-width:2px;
    classDef cache fill:#ffe0b2,stroke:#f57c00,stroke-width:2px;
    classDef async fill:#e8f5e9,stroke:#388e3c,stroke-width:2px;
    classDef ml fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px;

    %% Client Zone
    subgraph Client_Layer ["Client & Load Testing Layer"]
        Clients["Concurrent Web Clients\n(Locust / JMeter)"]:::client
        DataTypes["Heterogeneous Payloads\n(Images, Text, Tabular)"]:::client
        Clients -->|HTTP POST (JSON)| DataTypes
    end

    %% API / Gateway Zone
    subgraph Gateway_Layer ["Ingestion & Routing Layer (FastAPI)"]
        FastAPI["FastAPI ASGI Server\n(Non-blocking Event Loop)"]:::api
        Hasher["SHA-256 Hasher\n(Payload Serialization)"]:::api
        Router["Response Router"]:::api
    end

    %% Caching Zone
    subgraph Cache_Layer ["In-Memory Caching Layer"]
        Redis[("Redis Datastore\n(TTL: 300s)")]:::cache
    end

    %% Asynchronous Queuing Zone
    subgraph Queue_Layer ["Asynchronous Buffering"]
        AsyncQueue["asyncio.Queue\n(Thread-Safe Waiting Line)"]:::async
        Future["asyncio.Future\n(Connection Pointers)"]:::async
    end

    %% Machine Learning Zone
    subgraph Compute_Layer ["CPU-Bound Machine Learning Layer"]
        BatchEngine["Micro-Batching Engine\n(Worker Thread: Max 8 or 50ms)"]:::ml
        
        subgraph Pipelines ["Supported ML Pipelines"]
            CNN["Medical Diagnostics\n(CNN 64x64 Array)"]:::ml
            XGB["Engine Maintenance\n(XGBoost 1D Array)"]:::ml
            NLP["Sentiment Analysis\n(TF-IDF + Naive Bayes)"]:::ml
        end
    end

    %% Flow Connections
    DataTypes -->|Concurrent Requests| FastAPI
    FastAPI --> Hasher
    Hasher -->|Query Digest| Redis
    
    %% Cache Hit Flow
    Redis -.->|Cache Hit (<1ms)| Router
    
    %% Cache Miss Flow
    Redis -->|Cache Miss| AsyncQueue
    AsyncQueue --> Future
    
    %% Batching Flow
    Future -->|Pulls Items| BatchEngine
    BatchEngine -->|Stacks into Tensor| CNN
    BatchEngine -->|Stacks into 2D Matrix| XGB
    BatchEngine -->|Stacks into String List| NLP
    
    %% Response Routing Flow
    CNN & XGB & NLP -->|Vectorized Predictions| Router
    Router -->|1. Update Keys| Redis
    Router -->|2. Resolve Futures| FastAPI
    FastAPI -->|HTTP 200 OK| Clients
