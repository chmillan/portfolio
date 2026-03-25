# Engineering a Production Health Data Update Pipeline

## Context

This work was developed as part of a production data infrastructure for health connectivity analytics. The system is responsible for maintaining an up-to-date analytical database used for monitoring network performance across health units.

The challenge was not only ingesting data, but ensuring that updates were:

- reliable  
- incremental  
- reproducible  
- consistent across dependent structures  

To address this, a multi-job architecture was designed to handle different aspects of the update process.

---

## System Design

The pipeline was structured into three independent but coordinated jobs, each responsible for a specific layer of the system:

### 1. Materialized View Refresh Coordinator

This job is responsible for maintaining consistency in the analytical layer of the database.

- Triggers refresh of materialized views  
- Ensures derived tables reflect the latest base data  
- Controls execution order to avoid inconsistencies  

This component isolates analytical dependencies and prevents stale or partially updated views.

---

### 2. Public Dataset Version Checker

This job automates the detection of new data releases from an external public source.

- Queries a public endpoint to check for new dataset versions  
- Compares with the latest version already ingested  
- Triggers downstream processing only when new data is available  

This avoids unnecessary processing and ensures the system reacts only to real updates.

---

### 3. Incremental PostgreSQL → Parquet Pipeline

This job handles the extraction and persistence of measurement data for scalable analytical use.

- Reads data from the health database (PostgreSQL)  
- Applies incremental loading based on the last processed timestamp  
- Transforms data using Spark  
- Persists results in Parquet format  

This design avoids full reprocessing and enables efficient large-scale data handling.

---

## Key Design Decisions

### Separation of Responsibilities

Instead of a monolithic pipeline, responsibilities were split across independent jobs:

- ingestion trigger logic  
- database consistency  
- data transformation and storage  

This improves maintainability, observability, and failure isolation.

---

### Incremental Processing

Rather than reprocessing the full dataset:

- only new measurements are extracted  
- processing time is reduced  
- system load is minimized  

This is critical for scalability as data volume grows.

---

### Reproducibility and Automation

Each job is designed to be:

- independently executable  
- containerized  
- compatible with scheduled execution (e.g., CronJobs)  

This enables consistent and repeatable updates in production environments.

---

### Distributed Processing with Spark

The transformation layer uses Spark for scalable processing of measurement data.

In a Kubernetes environment, this required explicit handling of driver–executor communication, ensuring that executors can reliably connect back to the driver.

This aspect is detailed in a separate technical note:

→ [Spark on Kubernetes: Driver–Executor Communication](spark-driver-networking.md)

---

## Results

The implementation of this architecture led to a stable and production-ready data pipeline.

Key outcomes:

- Data updates became fully automated  
- Incremental processing reduced unnecessary workload  
- Materialized views remained consistent after updates  
- External dataset updates were detected and handled without manual intervention  
- Pipeline execution became predictable and reproducible  

This transformed the system from a manual and potentially inconsistent process into a robust, scalable data infrastructure.

---

## Engineering Impact

This work reflects a shift from isolated scripts to a structured data engineering system.

It demonstrates:

- system-level thinking  
- separation of concerns  
- production-oriented design  
- integration of distributed processing  

The resulting pipeline supports reliable analytical workflows and can scale with increasing data volume and system complexity.