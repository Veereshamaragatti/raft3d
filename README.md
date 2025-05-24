# Raft3D

**Raft3D** is a distributed system designed to manage 3D printing resources such as printers, filaments, and print jobs using a simplified mock Raft consensus algorithm. The system is Dockerized as a three-node cluster and demonstrates leader election, fault tolerance, RESTful APIs, Prometheus monitoring, and persistent snapshots.

---

## 🚀 Project Overview

### Purpose
Raft3D provides a fault-tolerant platform to manage 3D printing resources across a distributed environment. It ensures consistency via a mock Raft algorithm and supports API-driven interactions and internal metrics.

### Key Objectives

- Deploy a three-node cluster using Docker Compose.
- Implement leader election for a single coordinator.
- Develop RESTful APIs for printers, filaments, and print jobs.
- Provide fault tolerance with leader re-election.
- Expose Prometheus metrics for system observability.
- Periodically save state snapshots for persistence.

---

## ⚙️ Core Components

### 1. Dockerized Cluster

- **Setup:**
  - Services: `raft3d-node1`, `raft3d-node2`, `raft3d-node3`
  - Shared volume `raft-data` mapped to `/raft/data`
  - Bridge network `raft3d-net`
  - Port mapping:
    - API: `8080–8082`
    - Raft: `9090–9092`

- **Deployment:**

  ```bash
  sudo docker-compose up --build -d
  sudo docker ps
