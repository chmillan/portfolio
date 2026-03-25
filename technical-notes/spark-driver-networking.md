# Spark on Kubernetes: Driver–Executor Communication and Network Configuration

## Context

This work was developed as part of a production data pipeline for health measurement ingestion (PostgreSQL → Spark → Parquet) running in a Kubernetes environment.

During implementation, a critical issue emerged: Spark executors were unable to reliably communicate back to the driver, causing intermittent failures and unstable execution.

In distributed Spark systems, network configuration is not a secondary concern — it directly determines whether jobs succeed or fail.


## Problem

Spark executors must communicate back to the driver to return results and complete execution.

In Kubernetes, this becomes non-trivial due to:

- dynamic pod-level IP allocation  
- multiple network interfaces  
- incorrect default host resolution  

As a result, the driver was exposing an unreachable address, leading to:

- connection timeouts  
- executor loss  
- non-deterministic job failures  

This issue made the pipeline unreliable and difficult to debug.

## Solution

The solution involved explicitly controlling how the Spark driver IP is determined and exposed.

### 1. Kubernetes-native approach (preferred)

When running inside Kubernetes, the driver pod IP is injected via environment variables:

```python
driver_client_ip = config["pod_ip"]
```

This ensures that:

the IP is valid within the cluster network
executors can reliably reach the driver
no additional discovery logic is required

### 2. Socket-based IP discovery (fallback)

When pod_ip is not available, the system dynamically determines the correct outbound IP:

```python
s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
s.connect((config["spark_host"], 1))
driver_client_ip = s.getsockname()[0]
s.close()
```

This approach:

- avoids hardcoding network interfaces
- identifies the correct IP used to reach the Spark cluster
- adapts to different environments (local, containerized, hybrid)


**Architecture Insight**

In this setup:

- the **Driver** runs inside a Kubernetes pod and coordinates execution  
- multiple **Executor pods** process data in parallel  
- executors must connect back to the driver using a reachable IP  

Incorrect driver address configuration (e.g., `127.0.0.1`) results in failed communication and job instability.

Explicitly setting the driver host ensures proper routing across the cluster.

```

          Kubernetes Cluster
 ┌───────────────────────────────────┐

   Driver Pod (IP: 10.x.x.x)
          ↑              ↓
          │              │
   Executor Pods     Executor Pods

 └───────────────────────────────────┘

 ```


## Results

Explicit driver IP configuration resolved the communication issues between executors and the driver.

After implementation:

- Spark jobs executed without connection timeouts  
- Executor stability improved across repeated runs  
- "Executor lost" errors were eliminated  
- Pipeline behavior became consistent across environments (local and Kubernetes)  

This change transformed the pipeline from unstable and non-deterministic to reliable and production-ready.