---
title: "End-to-End Engineering of an Analytical Platform"
format: html

---
**From architecture design to performance validation**

> Case study based on professional work developed in institutional/private repositories.  
> This document focuses on architectural decisions, engineering trade-offs, and validation practices rather than source code.

---

## 1. Context

The system was designed to deliver large-scale connectivity indicators through:

- A Shiny-based interactive dashboard  
- A Plumber API layer  
- A relational database backend  
- A containerized deployment workflow  

As usage increased, engineering decisions became critical to ensure:

- Stable performance under concurrent access  
- Reproducibility across environments  
- Controlled schema evolution  
- Predictable deployment behavior  

---

## 2. System Architecture

The application was structured with clear separation of responsibilities:

- **Database layer** for storage, indexing, and pre-aggregation  
- **API layer** for structured data access  
- **Reactive layer (Shiny)** for interactive exploration  
- **Container layer** for environment isolation  

This modular design reduced coupling and allowed performance improvements at the appropriate layer.

---

## 3. Containerization and Environment Control

The system was deployed using multi-stage Docker builds.

**Key decisions:**

- Builder/runtime separation  
- Explicit Linux system dependency installation  
- Locked analytical dependencies  
- Controlled runtime configuration (ports, environment variables, library paths)  
- CI-driven automated image builds  

The objective was reproducibility and stability rather than minimal image size alone.

---

## 4. Database Strategy

As query complexity increased, performance optimization focused first on the data layer.

### Materialized Views

Expensive aggregations were moved to materialized views in order to:

- Reduce repeated computation at request time  
- Stabilize response time variability  
- Provide consistent structures for API consumption  

Refresh strategy was aligned with acceptable data freshness constraints.

### Indexing

Indexes were introduced or adjusted based on:

- Frequent filter patterns  
- Join keys  
- Grouping variables used in dashboards  

The goal was targeted optimization rather than excessive indexing.

### Schema Evolution

Database changes were applied via version-controlled migration files (e.g., Goose), enabling:

- Reproducible schema state  
- Reviewable DDL changes  
- Controlled updates across environments  

---

## 5. API Design and Resource Management

The API layer was built using Plumber.

**Engineering considerations:**

- Database connection management with `dbPool`  
- Controlled shutdown of database resources  
- Avoiding per-request connection creation  
- Limiting payload size to required fields  
- Aligning endpoint logic with pre-aggregated database structures  

This ensured stable behavior under repeated and concurrent requests.

---

## 6. Reactive Optimization (Shiny Layer)

Performance improvements were not limited to the backend.

Reactive strategy included:

- Using `eventReactive()` for user-triggered actions  
- Isolating heavy computations to avoid broad invalidations  
- Reducing unnecessary recomputation  
- Minimizing UI re-rendering  

This reduced latency spikes during interactive use.

---

## 7. Performance results
---
Performance improvements were validated empirically after architectural changes.


### End-to-End Session Duration (per user session)

| Architecture | Median (s) | SD (s) |
|---|---:|---:|
| Classic | 115.9 | 0.8 |
| New | 45.2 | 1.2 |


---

### Interaction Latency Under Concurrency 

| Architecture | Metric | Typical range (s) | Notable behavior |
|---|---|---:|---|
| New | p50 (median) | ~0.05–0.12 | Low typical latency across concurrency |
| New | p95 (tail) | ~0.10–1.30 | Spike around concurrency ~2 (tail instability) |
| Classic | p50 (median) | ~0.03–0.06 | Stable and consistently low |
| Classic | p95 (tail) | ~0.12–0.20 | Relatively stable tail latency |

These results validated database pre-aggregation, API refinement, and reactive optimization decisions.

---

## Appendix: Deployment Workflow

For a detailed reference implementation of the CI/CD and Docker Buildx setup:

→ [Deployment guide (Docker Buildx + CI/CD)](deployment-guide-buildx-ci.md)