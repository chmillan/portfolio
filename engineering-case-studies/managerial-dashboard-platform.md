# Managerial Dashboard Platform: From Prototype to Scalable Production Architecture

##  Problem

The goal of this project was to **provide managers with a single place to monitor initiatives and projects through a fast, reliable, and scalable interface**.

The initial version of the application was built using local CSV files. While effective for early exploration, this approach quickly became limiting as data volume, complexity, and integration needs grew.

Key challenges included:

- fragmented data sources
- slow performance with larger datasets
- lack of scalability
- tight coupling between data and application logic

## 🎯 Objective

The goal of this project was to provide managers an application/dashboard capable of consolidating projects and initiatives tracking into a single, high-performance interface.

The solution needed to:

- provide centralized visibility for managers
- support modular growth across different domains
- enable scalable data access (Spark, parquet, APIs)
- be ready for production deployment and future scaling

## 📊 Business Context
Managers often rely on multiple tools, spreadsheets, and disconnected dashboards to track initiatives and operational metrics.

This application was designed to act as a unified decision-support layer, allowing managers to:

- monitor multiple initiatives in one place
- access consistent and reliable metrics
- reduce time spent navigating fragmented systems


## Solution overview

The application was redesigned into a modular dashboard platform with a clear separation between data access, processing, and presentation layers.

The solution evolved into:

- modular Shiny application design
- parquet-based data storage
- API layer for data access abstraction
- environment-aware deployment configuration

## Architecture 

``` 
[Spark]
   ↓
[Parquet datasets]
   ↓
[API layer]
   ↓
[Shiny App (modular)]

``` 
- Spark is used for large-scale data access and exploration
- Parquet serves as the optimized storage format
- API decouples data processing from the frontend
- Shiny consumes only processed and structured data

## ⚙️ Key Engineering Decisions

- Introduced modular architecture to improve maintainability
- Replaced static files with parquet for performance and scalability
- Designed a data_loader abstraction to decouple data sources
- Introduced an API layer to isolate frontend from backend complexity
- Planned data contracts to ensure schema consistency
- Designed environment-based configuration for deployment flexibility


## Engineering and Data Strategy

- Used Spark for schema exploration and large-scale data access
- Extracted small local samples for efficient development
- Standardized core schema across modules (e.g., dayofyear, agent_id, agent_software)
- Designed aggregated datasets aligned with dashboard requirements

- Introduced containerization with Docker and Kubernetes-ready deployment setup
- Externalized environment configuration (dev, staging, production)

- Implemented structured logging and defined runtime metrics
- Added health checks and data validation to prevent silent failures

## Impact
- Improved performance by eliminating static, file-based workflows
- Reduced coupling between data sources and application logic
- Enabled scalability through parquet and API-based architecture
- Established a foundation for production deployment and future growth