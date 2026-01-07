# 🚀 Event-Driven Neural Search Engine

![Python](https://img.shields.io/badge/Python-3.11+-blue.svg)
![FastAPI](https://img.shields.io/badge/FastAPI-Production-green.svg)
![Architecture](https://img.shields.io/badge/Architecture-Event--Driven-orange.svg)
![Database](https://img.shields.io/badge/Vector_DB-Qdrant-red.svg)

> **A high-performance, asynchronous RAG (Retrieval-Augmented Generation) pipeline designed to handle large-scale document ingestion without blocking the main application thread.**

---

## 📖 Project Overview

This system solves a common bottleneck in AI applications: **Latency during document processing.**

Instead of processing uploads synchronously (which blocks the server), this project implements an **Event-Driven Architecture**. When a user uploads a PDF, the API accepts the request immediately and offloads the heavy lifting (chunking, embedding, indexing) to a durable background queue managed by **Inngest**. This ensures the application remains responsive and scalable, even under heavy load.

---

## 🏗️ System Architecture

The system is decoupled into an **Ingestion Pipeline** (Write Path) and a **Retrieval Pipeline** (Read Path).

```mermaid
graph TD
    User([User]) -->|POST /upload| API[FastAPI Server]
    API -->|Trigger Event| Inngest[Inngest Event Bus]
    API --x|Immediate 200 OK| User
    
    Inngest -->|Async Dispatch| Worker[Background Worker]
    
    subgraph "Ingestion Pipeline (Async)"
    Worker -->|1. Parse| Parser[PDF Parser]
    Parser -->|2. Chunk| Chunks[Recursive Text Splitter]
    Chunks -->|3. Embed| OpenAI[OpenAI Embeddings]
    OpenAI -->|4. Index| VectorDB[(Qdrant Vector DB)]
    end
    
    subgraph "Retrieval Pipeline (Sync)"
    User -->|POST /chat| API
    API -->|Semantic Query| VectorDB
    VectorDB -->|Context| LLM[GPT-4o]
    LLM -->|Answer| User
    end