# 🚀 Real-Time Collaborative Editor
### Dockerized Deployment on AWS ECS Fargate

<p align="center">

![React](https://img.shields.io/badge/React-Frontend-61DAFB?logo=react)
![Node.js](https://img.shields.io/badge/Node.js-Backend-339933?logo=node.js)
![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED?logo=docker)
![AWS](https://img.shields.io/badge/AWS-ECS%20Fargate-FF9900?logo=amazonaws)
![Socket.io](https://img.shields.io/badge/Socket.io-Realtime-black?logo=socket.io)
![Yjs](https://img.shields.io/badge/Yjs-CRDT-purple)

</p>

A real-time collaborative editor (similar to Google Docs / Figma) built to learn **Docker containerization** and **production deployment on AWS**.

Multiple users can edit the same document simultaneously with **conflict-free synchronization** powered by **Yjs (CRDT)** and **Socket.io**.

> **Project Goal**
>
> The primary objective of this project was to understand how a full-stack application moves from local development to a production deployment using Docker, Amazon ECR, ECS Fargate and an Application Load Balancer.

---

# 📑 Table of Contents

- Tech Stack
- Screenshots
- Demo
- Features
- Architecture
- Project Workflow
- What This Project Demonstrates
- Running Locally
- Running with Docker
- AWS Deployment
- Security
- Challenges Faced
- Future Improvements
- Status

---

# 🛠 Tech Stack

| Layer | Technology |
|------|-------------|
| Frontend | React.js |
| Editor | Monaco Editor |
| Backend | Node.js + Express |
| Real-time | Socket.io |
| Conflict Resolution | Yjs (CRDT) |
| Containerization | Docker |
| Registry | Amazon ECR |
| Orchestration | Amazon ECS Fargate |
| Load Balancer | AWS ALB |
| Security | IAM + Security Groups |

---

# 📸 Screenshots

## Multiple Users Connected

Two users connected from different browser sessions.

<p align="center">
<img src="./screenshots/users-connected.png" width="900">
</p>

---

## Real-Time Synchronization

Typing from one browser instantly appears in every connected client.

<p align="center">
<img src="./screenshots/realtime-sync.png" width="900">
</p>

---

## AWS Deployment

Application running inside Docker containers on ECS Fargate behind an Application Load Balancer.

<p align="center">
<img src="./screenshots/aws-deployment.png" width="900">
</p>

---

# 🎥 Demo

The application supports:

- Real-time collaborative editing
- Multiple connected users
- Live synchronization
- Conflict-free editing using CRDT
- Dockerized deployment on AWS

---

# ✨ Features

- Real-time collaborative editing
- Multiple users editing the same document
- Conflict-free synchronization using **Yjs (CRDT)**
- Monaco Editor integration
- Socket.io based communication
- Single Docker container deployment
- Frontend and backend served from the same origin

---

# 🏗 Architecture

```
Browser
      │
      ▼
Application Load Balancer
      │
      ▼
ECS Fargate Container
      │
      ├──────── Express Server
      │
      ├──────── React Static Build
      │
      └──────── Socket.io + Yjs
                   │
                   ▼
          Real-time Synchronization
```

---

# ⚙ Project Workflow

```
React + Monaco
        │
        ▼
Express Backend
        │
        ▼
Socket.io + Yjs
        │
        ▼
Docker Multi-stage Build
        │
        ▼
Docker Image
        │
        ▼
Amazon ECR
        │
        ▼
ECS Task Definition
        │
        ▼
Amazon ECS Fargate
        │
        ▼
Application Load Balancer
        │
        ▼
Live Production Application
```

---

# 📚 What This Project Demonstrates

This project demonstrates practical knowledge of:

- Docker multi-stage builds
- Docker image optimization
- Serving React through Express
- Container networking
- Amazon ECR
- ECS Task Definitions
- ECS Fargate deployments
- Application Load Balancers
- Target Groups
- IAM Roles & Policies
- Security Groups
- WebSocket communication in production
- Real-time synchronization using CRDTs

---

# 💻 Running Locally

```bash
git clone https://github.com/<your-username>/collab-editor-docker-aws.git

cd collab-editor-docker-aws

cd Frontend
npm install

cd ../Backend
npm install
```

Run both frontend and backend locally.

---

# 🐳 Running with Docker

```bash
docker build -t collab-editor .

docker run -p 3000:3000 collab-editor
```

---

# ☁ Deploying to AWS

1. Build Docker image

```bash
docker build -t collab-editor .
```

2. Push image to Amazon ECR

```bash
docker tag collab-editor:latest <account-id>.dkr.ecr.<region>.amazonaws.com/collab-editor

docker push <account-id>.dkr.ecr.<region>.amazonaws.com/collab-editor
```

3. Create ECS Task Definition

4. Create ECS Fargate Service

5. Attach Application Load Balancer

6. Configure Target Group

7. Configure Security Groups

---

# 🔐 Security

- Multi-stage Docker build
- Non-root container user
- Least-privilege IAM roles
- Security Groups
- Frontend & backend served from same origin
- Ready for Secrets Manager / Parameter Store integration

---

# 🧩 Challenges Faced

During deployment several production issues were encountered and resolved.

- Debugging WebSocket failures caused by hardcoded localhost URLs
- Multi-stage Docker build issues
- Serving React static files through Express
- Docker image versioning
- ECS Task Definition revisions
- Updating running ECS services
- Application Load Balancer configuration
- Socket.io communication through AWS infrastructure

These debugging experiences provided hands-on exposure to real production deployment workflows beyond local development.

---

# 🚀 Future Improvements

- Persistent document storage (MongoDB/PostgreSQL)
- Authentication & authorization
- GitHub Actions CI/CD pipeline
- HTTPS using ACM
- CloudWatch monitoring
- Auto Scaling
- Custom domain
- Document history
- Cursor presence
- User avatars

---

# 📌 Status

✅ Project completed successfully.

The application has been:

- Dockerized
- Deployed on Amazon ECS Fargate
- Connected through an Application Load Balancer
- Tested with multiple simultaneous users
- Successfully synchronized in real time using Socket.io + Yjs

> **Note:**  
> The live deployment may be taken down periodically to avoid AWS infrastructure costs. The project remains fully reproducible using the provided Dockerfile and deployment instructions.

---

## ⭐ Learning Outcome

This project provided practical experience with:

- Docker
- Containerized application architecture
- AWS ECR
- ECS Fargate
- Application Load Balancer
- IAM
- Production debugging
- WebSockets
- Real-time collaboration systems
- CRDT-based synchronization

Instead of only learning these technologies theoretically, this project involved deploying, debugging, and operating a real production workload on AWS.
