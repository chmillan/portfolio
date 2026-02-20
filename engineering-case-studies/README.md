# ⚙️ Engineering Case Studies  
**Reproducible analytical systems and production-oriented design**

This section presents selected projects focused on the engineering layer of analytical work: system architecture, reproducibility, containerization, and deployment-ready design.

While the analytical track emphasizes statistical modeling, this track highlights how analytical systems are structured to be stable, scalable, and reproducible under real-world constraints.

---

## Containerized Analytical Platform for Connectivity Monitoring

Design and implementation of a containerized Shiny and API-based analytical system serving large-scale connectivity indicators.

**Engineering focus:**

- Multi-stage Docker builds (builder + runtime separation)
- Explicit Linux system dependency management (GDAL, libpq, etc.)
- Reproducible environments with locked dependencies
- Git-based workflow with structured commits
- GitLab CI pipeline for automated image build and registry publishing
- Separation between data processing, API, and visualization layers
- Runtime stability and resource cleanup

→ (coming soon)

---

## Production API with Database Integration

Development of a containerized API service integrated with relational databases.

**Engineering focus:**

- Plumber-based API design
- Connection pooling and controlled resource management
- Environment isolation inside Linux containers
- Clean shutdown hooks
- Production-ready image configuration

→ (coming soon)

---

## Load Testing and Stability Validation

Performance validation of interactive analytical systems under concurrent access.

**Engineering focus:**

- Load testing using shinycannon
- Identification of runtime bottlenecks
- Iterative refinement of container configuration
- Ensuring parity between local and CI environments

→ (coming soon)