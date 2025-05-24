Raft3D
Raft3D is a distributed system designed to manage 3D printing resources — such as printers, filaments, and print jobs — using a simplified mock Raft consensus algorithm. The system is Dockerized as a three-node cluster and demonstrates key distributed systems concepts including fault tolerance, leader election, state replication, RESTful API interfaces, and Prometheus-based monitoring.

🚀 Project Overview
Purpose
Raft3D provides a fault-tolerant platform to manage 3D printing resources across a distributed environment. It ensures operational consistency via a mock Raft algorithm, supports API-driven resource interactions, and exposes internal metrics for observability.

Key Objectives

Deploy a three-node cluster using Docker Compose.

Implement leader election to maintain a single active coordinator.

Develop RESTful APIs to manage 3D printers, filaments, and print jobs.

Ensure fault tolerance via dynamic leader election on node failure.

Expose Prometheus metrics to monitor system health.

Support periodic state snapshots for persistent storage.

⚙️ Core Components
1. Dockerized Cluster
Docker Compose Configuration

Services: raft3d-node1, raft3d-node2, raft3d-node3

Environment Variables:

NODE_ID, RAFT_PORT, HTTP_PORT, RAFT_DIR, CLUSTER

Port Mapping:

External ports 8080–8082 (API) and 9090–9092 (Raft) mapped to internal ports

Shared volume raft-data mapped to /raft/data

Deployment

bash
Copy
Edit
sudo docker-compose up --build -d
sudo docker ps
Capabilities

Clustered setup using a bridge network

Shared storage for consensus and snapshots

2. Mock Raft Implementation
MockRaft Class

Simplified Raft using leader.txt file for coordination

Handles leader election and state application

50% randomized election chance to avoid flapping

Leader Election Logic

Check for leader.txt or stale file (>10s)

Attempt election if no valid leader exists

Fault Tolerance

Detect and replace stale leaders

Apply log entries to a finite state machine (FSM)

3. RESTful APIs (FastAPI)
Endpoints (/api/v1)

Printers

POST /printers

GET /printers

Filaments

POST /filaments

GET /filaments

Print Jobs

POST /print_jobs

POST /print_jobs/{id}/status

Capabilities

API operations routed via the leader node

Automatic filament weight updates after print job execution

4. Prometheus Metrics
Metrics Exposed at /metrics

raft3d_is_leader — leader status

raft3d_requests_total — API request count

raft3d_snapshots_total — snapshot count

System-level: CPU, memory, etc.

5. Fault Tolerance
Behavior

Leader failure triggers election

API continues via newly elected leader

Stopped nodes can rejoin the cluster

Test Workflow

bash
Copy
Edit
curl http://localhost:8080/metrics | grep raft3d_is_leader
sudo docker stop <leader_container>
curl http://localhost:8081/metrics | grep raft3d_is_leader
6. Snapshots
Snapshot Manager

Periodic snapshots to /raft/data

Tracked using Prometheus metric raft3d_snapshots_total

View Snapshots

bash
Copy
Edit
sudo docker exec raft3d_raft3d-node2_1 ls /raft/data
📦 Demonstration Commands
1. Cluster Setup
bash
Copy
Edit
sudo docker-compose up --build -d
sudo docker ps
sudo docker-compose logs | grep "elected leader"
2. Metrics Verification
bash
Copy
Edit
curl http://localhost:8080/metrics | grep raft3d_is_leader
curl http://localhost:8081/metrics | grep raft3d_is_leader
curl http://localhost:8082/metrics | grep raft3d_is_leader
3. API Interactions
bash
Copy
Edit
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
4. Fault Tolerance
bash
Copy
Edit
# Check Current Leader
curl http://localhost:8080/metrics | grep raft3d_is_leader
curl http://localhost:8081/metrics | grep raft3d_is_leader
curl http://localhost:8082/metrics | grep raft3d_is_leader

# Stop Current Leader
sudo docker stop raft3d_raft3d-node1_1

# Check New Leader
curl http://localhost:8081/metrics | grep raft3d_is_leader
curl http://localhost:8082/metrics | grep raft3d_is_leader
5. View Snapshots
bash
Copy
Edit
sudo docker exec raft3d_raft3d-node2_1 ls /raft/data
✅ Capabilities Summary
Distributed Cluster — Three-node deployment with shared volume

Leader Election — Managed via a mock Raft algorithm using shared file

Fault Tolerance — Recovers from failures, supports dynamic leadership

RESTful API — Full CRUD support for 3D printing resources

Monitoring — Prometheus metrics for system observability

Snapshot Persistence — State backups for recovery and debugging

📁 Directory Structure (Simplified)
css
Copy
Edit
raft3d/
├── app/
│   ├── api/
│   ├── raft/
│   ├── monitoring/
│   ├── main.py
├── docker-compose.yml
├── Dockerfile
└── README.md
🧪 Tech Stack
Python (FastAPI)

Docker / Docker Compose

Prometheus

Custom Mock Raft Algorithm

REST API / JSON
