# End-to-End Engineering of an Analytical Platform  
**From architecture design to performance validation**

---

## Context & Architecture

The system delivers large-scale connectivity indicators through:

- A Shiny interactive dashboard  
- A Plumber API layer  
- A relational database backend  
- A containerized deployment workflow  

To ensure stability under real-world usage, the architecture was structured with clear separation of responsibilities:

- **Database layer** (indexing, pre-aggregation, schema control)  
- **API layer** (structured data access)  
- **Reactive layer** (interactive exploration)  
- **Container layer** (environment isolation)

This modular design allowed performance improvements at the appropriate layer.

---

## Engineering Decisions

### Containerization

- Multi-stage Docker builds (builder/runtime separation)  
- Explicit Linux system dependencies  
- Locked analytical dependencies  
- CI-driven image builds  

### Database Strategy

- Materialized views for expensive aggregations  
- Targeted indexing based on access patterns  
- Schema evolution via version-controlled migrations  

### API & Reactive Optimization

- Database connection management using `dbPool`  
- Controlled resource shutdown  
- `eventReactive()` for user-triggered computation  
- Isolation of heavy reactive dependencies  

---

# **Results**

Performance improvements were validated empirically.

### End-to-End Session Duration

| Architecture | Median (s) | SD (s) |
|---|---:|---:|
| Classic | 115.9 | 0.8 |
| New | 45.2 | 1.2 |

### Interaction Latency Under Concurrency 

| Architecture | Metric | Typical range (s) | Behavior |
|---|---|---:|---|
| New | p50 | ~0.05–0.12 | Low typical latency |
| New | p95 | ~0.10–1.30 | Reduced tail variability |
| Classic | p50 | ~0.03–0.06 | Stable but less optimized flow |
| Classic | p95 | ~0.12–0.20 | Moderate tail latency |

These results validated the layered optimization strategy across database, API, and reactive layers.

---

## Deployment Workflow

For detailed CI/CD and Docker Buildx configuration:

→ [Deployment guide](deployment-guide-buildx-ci.md)