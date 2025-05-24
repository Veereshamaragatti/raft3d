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
```

---

### 2. Mock Raft Implementation

- **Leader Coordination:**
  - Shared file `leader.txt` stores current leader ID.
  - 50% randomized election to reduce conflicts.
  - Stale file (>10s) triggers new election.

- **Methods:**
  - `start`, `stop`, `become_leader`, `resign_leader`, `run`, `apply`

---

### 3. RESTful APIs (FastAPI)

- **Base URL:** `/api/v1`
- **Endpoints:**

  - **Printers**
    - `POST /printers`
    - `GET /printers`

  - **Filaments**
    - `POST /filaments`
    - `GET /filaments`

  - **Print Jobs**
    - `POST /print_jobs`
    - `POST /print_jobs/{id}/status`

---

### 4. Prometheus Metrics

- **Exposed at:** `/metrics`
- **Custom Metrics:**
  - `raft3d_is_leader`
  - `raft3d_requests_total`
  - `raft3d_snapshots_total`
  - System metrics: CPU, memory, etc.

---

### 5. Fault Tolerance

- Detects and replaces failed leaders via stale `leader.txt`
- Allows followers to take over leadership
- Nodes can rejoin after failure

---

### 6. Snapshots

- **SnapshotManager:**
  - Periodically saves system state
  - Files saved in `/raft/data`
  - Monitored via `raft3d_snapshots_total` metric

---

## 📦 Demonstration Commands

### 1. Cluster Setup

```bash
sudo docker-compose up --build -d
sudo docker ps
sudo docker-compose logs | grep "elected leader"
```

### 2. Metrics Verification

```bash
curl http://localhost:8080/metrics | grep raft3d_is_leader
curl http://localhost:8081/metrics | grep raft3d_is_leader
curl http://localhost:8082/metrics | grep raft3d_is_leader
```

### 3. API Interactions

```bash
# Create Printer
curl -X POST http://localhost:8080/api/v1/printers \
  -H "Content-Type: application/json" \
  -d '{"id": "p1", "company": "Creality", "model": "Ender 3"}'

# Create Filament
curl -X POST http://localhost:8080/api/v1/filaments \
  -H "Content-Type: application/json" \
  -d '{"id": "f1", "type": "PLA", "color": "Blue", "total_weight_in_grams": 1000, "remaining_weight_in_grams": 1000}'

# Create Print Job
curl -X POST http://localhost:8080/api/v1/print_jobs \
  -H "Content-Type: application/json" \
  -d '{"id": "j1", "printer_id": "p1", "filament_id": "f1", "filepath": "prints/sword/hilt.gcode", "print_weight_in_grams": 100, "status": "Queued"}'

# Update Print Job Status
curl -X POST http://localhost:8080/api/v1/print_jobs/j1/status?status=Running
curl -X POST http://localhost:8080/api/v1/print_jobs/j1/status?status=Done

# Retrieve Filaments
curl http://localhost:8080/api/v1/filaments
```

### 4. Fault Tolerance

```bash
# Check Current Leader
curl http://localhost:8080/metrics | grep raft3d_is_leader
curl http://localhost:8081/metrics | grep raft3d_is_leader
curl http://localhost:8082/metrics | grep raft3d_is_leader

# Stop Leader (example assumes node1 is the leader)
sudo docker stop raft3d_raft3d-node1_1

# Check New Leader
curl http://localhost:8081/metrics | grep raft3d_is_leader
curl http://localhost:8082/metrics | grep raft3d_is_leader
```

### 5. Snapshots

```bash
sudo docker exec raft3d_raft3d-node2_1 ls /raft/data
```

---

## ✅ Capabilities Summary

- ✅ **Distributed Cluster**: Three-node setup with shared storage
- ✅ **Leader Election**: Single leader via Mock Raft coordination
- ✅ **Fault Tolerance**: Automatic recovery from node failure
- ✅ **RESTful APIs**: Manage printers, filaments, and print jobs
- ✅ **Monitoring**: Prometheus metrics for system and performance
- ✅ **Snapshot Persistence**: Periodic backups of system state

---

## 📁 Project Structure

```
raft3d/
├── app/
│   ├── api/
│   ├── raft/
│   ├── monitoring/
│   └── main.py
├── docker-compose.yml
├── Dockerfile
└── README.md
```

---

## 🧪 Tech Stack

- **Python (FastAPI)**
- **Docker / Docker Compose**
- **Prometheus**
- **Custom Mock Raft Algorithm**
- **REST API / JSON**
