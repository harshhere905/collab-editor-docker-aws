# Collab Editor — Docker & AWS Deployment

A real-time collaborative text editor (Google Docs / Figma style) built to learn and practice **containerization with Docker** and **production deployment on AWS**. Multiple users can edit the same document at the same time, with conflict-free sync powered by CRDTs.

> **Note:** This is a learning-focused project. The main goal was not to build a feature-complete product, but to understand and implement a real Docker + AWS deployment pipeline end to end — from writing a Dockerfile to running a live container behind a load balancer on ECS Fargate.

---

## Table of contents

- [Tech stack](#tech-stack)
- [Architecture](#architecture)
  - [Full pipeline at a glance](#full-pipeline-at-a-glance)
  - [Phase-by-phase breakdown](#phase-1--development)
- [Features](#features)
- [What this project demonstrates](#what-this-project-demonstrates)
- [Getting started (local)](#getting-started-local)
- [Running with Docker](#running-with-docker)
- [Deploying to AWS](#deploying-to-aws-high-level)
- [Security notes](#security-notes)
- [Possible improvements](#possible-improvements)
- [Status](#status)

---

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | React.js + Monaco Editor |
| Backend | Node.js + Express.js |
| Real-time sync | Socket.io + Yjs (CRDT) |
| Containerization | Docker (multi-stage build) |
| Container registry | Amazon ECR |
| Container orchestration | Amazon ECS (Fargate launch type) |
| Traffic routing | Application Load Balancer (ALB) + Target Groups |
| Access control | IAM (scoped ECS/ECR permissions), Security Groups |

---

## Architecture

The project goes through six phases, from writing code to handling a live user request in production.

### Full pipeline at a glance

```
 DEVELOPMENT                          CONTAINERIZATION
 ┌────────────────────────┐          ┌───────────────────────────┐
 │ React + Monaco editor   │          │ Dockerfile (FROM, WORKDIR, │
 │        ↓                │          │ COPY, RUN, CMD)            │
 │ Express backend         │          │        ↓                   │
 │        ↓                │   ──▶    │ Multi-stage build          │
 │ Socket.io + Yjs (CRDT)  │          │        ↓                   │
 │        ↓                │          │ Non-root user               │
 │ npm run build → dist    │          │        ↓                   │
 │        ↓                │          │ Docker image                │
 │ Express serves dist     │          └───────────────────────────┘
 └────────────────────────┘                        │
                                                     ▼
 AWS STORAGE & INFRA SETUP                 DEPLOYMENT
 ┌────────────────────────┐          ┌───────────────────────────┐
 │ Image pushed to ECR     │          │ ECS Fargate runs container │
 │        ↓                │   ──▶    │        ↓                   │
 │ ┌─IAM user             │          │ ALB + Target Group attached│
 │ ├─Security groups       │          └───────────────────────────┘
 │ └─Task Definition       │                        │
 └────────────────────────┘                        ▼
                                       RUNTIME (a real user request)
                              ┌─────────────────────────────────────┐
                              │ User sends request (HTTP/WebSocket)  │
                              │        ↓                             │
                              │ ALB → Target Group → container       │
                              │        ↓                             │
                              │ Express handles it:                  │
                              │   • static file  → dist              │
                              │   • API call     → backend route     │
                              │   • real-time    → Socket.io + Yjs   │
                              │        ↓                             │
                              │ Response sent back to browser        │
                              └─────────────────────────────────────┘
```

### Phase 1 — Development

```
React frontend + Monaco editor (editor UI in the browser)
        │
        ▼
Express backend (Node.js API server)
        │
        ▼
Socket.io + Yjs joined in (adds real-time CRDT sync)
        │
        ▼
npm run build (bundles React into a "dist" folder)
        │
        ▼
Express serves "dist" as static files
(frontend + backend now run on the same origin/port)
```

### Phase 2 — Containerization

```
Dockerfile written (FROM, WORKDIR, COPY, RUN, CMD)
        │
        ▼
Multi-stage build (keeps the final image lightweight)
        │
        ▼
Non-root user configured (security best practice)
        │
        ▼
Docker image produced (the whole app packaged into one image)
```

### Phase 3 — AWS storage

```
Docker image pushed to Amazon ECR
        │
        ▼
Image stored safely in ECR (AWS's version of Docker Hub)
```

### Phase 4 — Infrastructure setup

These three are typically set up in parallel, ahead of deployment:

```
        ┌── IAM user (ECS + ECR scoped permissions)
Image → ├── Security groups (which ports are open)
        └── Task Definition (image, CPU/RAM, container port)
```

### Phase 5 — Deployment

```
ECS pulls the image from ECR using the Task Definition
        │
        ▼
Fargate runs it as a container (serverless — no EC2 server to manage)
        │
        ▼
Application Load Balancer (ALB) placed in front of the container
        │
        ▼
Target Group created (tells the ALB where to send traffic)
```

### Phase 6 — Runtime (a real user request)

```
User opens the app URL in the browser
        │
        ▼
Request reaches the ALB
        │
        ▼
ALB → Target Group → routed to a healthy container
        │
        ▼
Inside the container, Express handles the request:
   • Static file needed?  → served from the "dist" folder
   • API call?            → backend route/controller runs
   • Real-time edit?      → Socket.io + Yjs sync the change
        │
        ▼
Response sent back to the user's browser via the same path
```

---

## Features

- Real-time multi-user document editing with conflict resolution via **Yjs (CRDT)**
- Live cursor/edit sync using **Socket.io**
- Monaco Editor (the same editor that powers VS Code) for a familiar coding/writing experience
- Single-container deployment — frontend and backend served together

---

## What this project demonstrates

- Writing a production-ready, **multi-stage Dockerfile** (`FROM`, `WORKDIR`, `COPY`, `RUN`, `CMD`) to keep image size small
- Running containers as a **non-root user** for better security
- Bundling a React app and serving it through an Express backend on the same origin
- Pushing images to **Amazon ECR**
- Writing an ECS **Task Definition** and deploying it to **Fargate**
- Configuring **IAM users/roles** with least-privilege ECS/ECR permissions
- Setting up **Security Groups** to control inbound/outbound traffic
- Using an **Application Load Balancer** with a **Target Group** to route traffic to running containers

---

## Getting started (local)

```bash
# clone the repo
git clone https://github.com/<your-username>/collab-editor-docker-aws.git
cd collab-editor-docker-aws

# install dependencies (frontend + backend)
npm install

# build the React frontend
npm run build

# start the Express server (serves frontend + backend + sockets)
npm start
```

App will be available at `http://localhost:<PORT>`.

---

## Running with Docker

```bash
# build the image
docker build -t collab-editor .

# run the container
docker run -p 3000:3000 collab-editor
```

---

## Deploying to AWS (high level)

1. Build and tag the Docker image locally.
2. Authenticate Docker to ECR and push the image:
   ```bash
   aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com
   docker tag collab-editor:latest <account-id>.dkr.ecr.<region>.amazonaws.com/collab-editor:latest
   docker push <account-id>.dkr.ecr.<region>.amazonaws.com/collab-editor:latest
   ```
3. Create an ECS **Task Definition** pointing to the ECR image (set CPU/memory, container port, IAM execution role).
4. Create an ECS **Fargate Service** using that task definition.
5. Attach an **Application Load Balancer** with a **Target Group** to route traffic to the running tasks.
6. Configure **Security Groups** to allow traffic on the required ports (e.g. 80/443 → container port).

---

## Security notes

- Container runs as a **non-root user** inside the Dockerfile — reduces blast radius if the container is ever compromised.
- **Multi-stage build** keeps build tools and dev dependencies out of the final image, shrinking the attack surface.
- **IAM user/role** for ECS is scoped to only the ECS + ECR permissions it needs — not broad admin access.
- **Security Groups** restrict inbound traffic to only the required ports (e.g. 80/443 to the ALB, and the ALB to the container port).
- Secrets (API keys, DB credentials, etc., if added later) should go into **AWS Secrets Manager** or **SSM Parameter Store**, not hardcoded in the Dockerfile or repo.

---

## Possible improvements

- [ ] Add a database (e.g. MongoDB/PostgreSQL) for persistent document storage
- [ ] Add authentication (currently anyone with the link can edit)
- [ ] Set up CI/CD (GitHub Actions) to auto-build and push to ECR on merge
- [ ] Add auto-scaling policies on the ECS service based on CPU/memory
- [ ] Add HTTPS via ACM certificate on the ALB
- [ ] Add CloudWatch logging/alarms for the running tasks

---

🎯 Built primarily as a hands-on learning project for Docker containerization and AWS deployment (ECR, ECS Fargate, ALB). The live deployment may be taken down periodically to avoid ongoing AWS charges — the code and architecture remain fully functional and reproducible.
