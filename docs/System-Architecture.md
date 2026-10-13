# 🏗️ V1 System Architecture & Technical Specification

> **Project:** Competitive Coding Platform (Java V1 MVP)  
> **Author:** @YOKESH T  
> **Milestone:** Day 3 — Architecture, Execution Sandbox & Security Specification  
> **Status:** Final Approved Architecture  

---

## 📌 Executive Summary

This document details the complete end-to-end V1 system architecture for our competitive coding platform. It provides a production-grade blueprint for handling user authentication, contest management, code submission queuing, isolated container sandboxing, strict security boundaries, and full verdict management—all strictly runnable **locally and completely free** for V1 development.

---

## 🗺️ System Architecture Visual Diagram

```mermaid
flowchart TD
    classDef client fill:#2563eb,color:#fff,stroke:#1d4ed8,stroke-width:2px;
    classDef backend fill:#059669,color:#fff,stroke:#047857,stroke-width:2px;
    classDef queue fill:#d97706,color:#fff,stroke:#b45309,stroke-width:2px;
    classDef storage fill:#4f46e5,color:#fff,stroke:#4338ca,stroke-width:2px;
    classDef execution fill:#dc2626,color:#fff,stroke:#b91c1c,stroke-width:2px;

    Client["💻 React Frontend\n(TypeScript)"]:::client
    API["⚙️ Backend API\n(Node.js / Express)"]:::backend
    DB[("🗄️ Primary Database\n(PostgreSQL 16)")]:::storage
    Redis[("⚡ Task Queue & Broker\n(Redis 7 + BullMQ)")]:::queue
    Worker["🛠️ Judge Worker Service\n(Node.js Process)"]:::backend
    Disk[("📁 Test Case Storage\n(Local Disk / MinIO)")]:::storage
    Sandbox["🔒 Docker Sandbox\n(Java 21 OpenJDK)"]:::execution

    %% Submission & Execution Flow
    Client -->|"1. POST /api/submissions\n(Java Code)"| API
    API -->|"2. Write Pending Status"| DB
    API -->|"3. Push Execution Job"| Redis
    Redis -->|"4. Pop Job / Retry Handler"| Worker
    Worker -->|"5. Fetch Test Cases (.in / .out)"| Disk
    Worker -->|"6. Mount Code & Spawn Container"| Sandbox
    Sandbox -->|"7. Stream Execution Results (stdout/stderr)"| Worker
    Worker -->|"8. Persist Verdict & Metrics"| DB
    DB -.->"9. Poll / WebSocket Status Update"| Clientile Error`), and updates the PostgreSQL database record.
7. **Result Notification:** The Frontend receives the updated verdict via REST polling or WebSockets.
