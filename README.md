# Blue-Green Deployment Project

This repository contains a **Docker Compose** setup for a blue-green deployment of Node.js apps with Nginx as a reverse proxy.  
It automates failover between Blue and Green services and supports chaos testing.

---

## Features

- Two identical Node.js services (Blue and Green)  
- `/version` endpoint returns `X-App-Pool` and `X-Release-Id`  
- `/chaos/start` and `/chaos/stop` endpoints to simulate downtime  
- Automatic Nginx failover between Blue and Green  
- Fully parameterized via `.env`  

---

## Prerequisites

Before running the project, ensure you have:

- Linux server (Ubuntu 20.04+ recommended)  
- Docker & Docker Compose installed  
- `.env` file with image names, release IDs, and active pool  

---

## Usage

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/<your-repo-name>.git
cd <your-repo-name>
```

### 2. Create a .env file

Example .env:

```bash
# Active pool (blue or green)
ACTIVE_POOL=blue

# Docker images
BLUE_IMAGE=yimikaade/wonderful:devops-stage-two
GREEN_IMAGE=yimikaade/wonderful:devops-stage-two

# Release identifiers
RELEASE_ID_BLUE=blue-v1.0.0
RELEASE_ID_GREEN=green-v1.0.0
```

### 3. Launch the deployment
```bash
docker compose up -d
```
- Nginx public entrypoint: http://<server-ip>:8080
- Blue direct port: 8081
- Green direct port: 8082

### 4. Test baseline (Blue active)
```bash
curl -i http://localhost:8080/version
```

Expected response headers:
```makefile
X-App-Pool: blue
X-Release-Id: blue-v1.0.0
```

### 5. Trigger Blue downtime
```bash
curl -X POST http://localhost:8081/chaos/start
```

Test again:
```bash
curl -i http://localhost:8080/version
```

Nginx should automatically switch to Green:
```makefile
X-App-Pool: green
X-Release-Id: green-v1.0.0
```

### 6. Restore Blue
```bash
curl -X POST http://localhost:8081/chaos/stop
```

- Nginx will eventually route traffic back to Blue after fail_timeout expires

### 7. Cleanup Old Deployment
```bash
docker compose down -v
```
