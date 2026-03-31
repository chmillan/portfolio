# 🧾 Engineering Case Studies  
**Design and engineering of reproducible, scalable, and production-ready analytical systems**

This section presents selected projects focused on the engineering of data-driven systems, including architecture, reproducibility, and production-ready design.

The emphasis is on how analytical systems are structured to be scalable, maintainable, and reliable under real-world constraints.

---

## Managerial Dashboard Platform  

Design and evolution of a managerial application to centralize the monitoring of initiatives and projects in a fast, scalable, and production-ready interface.

**Engineering scope:**

- Transition from CSV-based prototype to parquet and API-based architecture  
- Modular Shiny application design with clear separation of concerns  
- Data access abstraction layer (`data_loader`) to decouple sources  
- Spark-based data exploration and local sampling strategy  
- Environment-aware configuration for deployment flexibility  
- Observability and data validation planning  

The project demonstrates how a dashboard can evolve from a local prototype into a production-ready system, balancing performance, scalability, and maintainability while supporting real-world managerial workflows.

→ [Read full case study](managerial-dashboard-platform.md)

---

## End-to-End Analytical Platform Engineering  

Design and evolution of a containerized analytical system combining database optimization, API integration, reactive tuning, and CI-driven deployment.

**Engineering scope:**

- Multi-stage Docker builds and environment isolation  
- Schema versioning and materialized views  
- API development with controlled DB connections  
- Reactive optimization in Shiny  
- Load testing and validation  

Performance improvements were validated empirically, improving responsiveness and stability under concurrent usage.

→ [Read full case study](analytical-platform-engineering.md)

---

## Production Health Data Pipeline  

Multi-job architecture for incremental data ingestion and analytical consistency in a production environment.

**Engineering scope:**

- Coordination of materialized view refreshes  
- Automated detection of new public dataset versions  
- Incremental data extraction and persistence (PostgreSQL → Parquet)  
- Distributed processing using Spark in Kubernetes  
- Separation of concerns across independent jobs  

The system transformed a manual and potentially inconsistent update process into a reliable, automated, and scalable data pipeline.

→ [Read case study](health-data-pipeline.md)

---

## What This Portfolio Demonstrates

- Ability to design scalable data-driven systems  
- Strong understanding of data pipelines and system architecture  
- Experience with production-ready practices (deployment, observability, validation)  
- Clear separation between data, backend, and frontend layers  