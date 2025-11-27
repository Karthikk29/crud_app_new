# MEAN Stack CRUD Application - DevOps Deployment

A full-stack CRUD application built with MongoDB, Express.js, Angular 15, and Node.js, containerized with Docker and deployed with CI/CD automation.

## 📋 Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Local Development](#local-development)
- [Docker Setup](#docker-setup)
- [CI/CD Pipeline](#cicd-pipeline)
- [Deployment Guide](#deployment-guide)
- [Project Structure](#project-structure)
- [Screenshots](#screenshots)

## 🎯 Overview

This application manages a collection of tutorials where each tutorial includes:
- ID (auto-generated)
- Title
- Description
- Published status (boolean)

Users can perform CRUD operations (Create, Read, Update, Delete) and search tutorials by title.

## ✨ Features

- **Create** new tutorials
- **Read** all tutorials or individual tutorial details
- **Update** tutorial information
- **Delete** tutorials (individually or all at once)
- **Search** tutorials by title
- **Responsive UI** built with Bootstrap 4

## 🚀 Tech Stack

### Backend
- Node.js
- Express.js
- MongoDB (with Mongoose ODM)
- CORS enabled

### Frontend
- Angular 15
- TypeScript
- RxJS
- Bootstrap 4
- HttpClient for API communication

### DevOps
- Docker & Docker Compose
- GitHub Actions (CI/CD)
- Nginx (Reverse Proxy)
- AWS EC2 (or any cloud VM)

## 📦 Prerequisites

### For Local Development:
- Node.js 18.x or higher
- MongoDB 6.x
- Angular CLI (`npm install -g @angular/cli`)

### For Docker Setup:
- Docker Desktop (or Docker Engine)
- Docker Compose

### For CI/CD and Deployment:
- GitHub account
- Docker Hub account
- AWS EC2 instance (or any cloud VM with Ubuntu)

## 💻 Local Development

### Backend Setup

```bash
cd backend
npm install
node server.js
```

The backend server will run on `http://localhost:8080`

**MongoDB Configuration:** Update `app/config/db.config.js` if needed (default: `mongodb://localhost:27017/dd_db`)

### Frontend Setup

```bash
cd frontend
npm install
ng serve --port 8081
```

The Angular app will run on `http://localhost:8081`

**API Configuration:** The frontend connects to backend at `/api/tutorials` (relative path for Docker/Nginx setup)

## 🐳 Docker Setup

### Build and Run with Docker Compose

1. **Build all services:**
```bash
docker-compose build
```

2. **Start all services:**
```bash
docker-compose up -d
```

3. **Access the application:**
- **Frontend:** http://localhost (port 80)
- **Backend API:** http://localhost/api/tutorials
- **MongoDB:** localhost:27017

4. **View logs:**
```bash
docker-compose logs -f backend
docker-compose logs -f frontend
```

5. **Stop all services:**
```bash
docker-compose down
```

### Docker Services

The `docker-compose.yml` orchestrates 4 services:

| Service | Image | Port | Description |
|---------|-------|------|-------------|
| mongodb | mongo:6 | 27017 | Database with persistent volume |
| backend | custom | 8080 | Node.js REST API |
| frontend | custom | 4200 | Angular app served by Nginx |
| nginx | nginx:alpine | 80 | Reverse proxy |

### Docker Architecture

```
Port 80 (nginx-proxy) 
    │
    ├─→ /api/*  → backend:8080 (Node.js API)
    │
    └─→ /*      → frontend:80 (Angular SPA)
                      │
                      └─→ /api/* → backend:8080
```

## 🔄 CI/CD Pipeline

The project uses **GitHub Actions** for automated CI/CD.

### Workflow Triggers
- Automatically runs on every push to the `main` branch

### Pipeline Stages

#### 1. **Build** Job
- Checkout code from repository
- Login to Docker Hub
- Build backend Docker image
- Build frontend Docker image
- Push both images to Docker Hub

#### 2. **Deploy** Job
- SSH into EC2 instance
- Pull latest Docker images
- Stop running containers
- Start updated containers with `docker-compose`
- Clean up old images

### Required GitHub Secrets

Configure these in your repository: Settings → Secrets and variables → Actions

| Secret Name | Description | Example |
|-------------|-------------|---------|
| `DOCKER_USERNAME` | Docker Hub username | `yourusername` |
| `DOCKER_PASSWORD` | Docker Hub password/token | `dckr_pat_xxxxx` |
| `EC2_HOST` | EC2 instance public IP | `54.123.45.67` |
| `EC2_USERNAME` | SSH username | `ubuntu` |
| `EC2_SSH_KEY` | Private SSH key content | `-----BEGIN RSA PRIVATE KEY-----...` |

## 🌐 Deployment Guide

### Step 1: GitHub Repository Setup

1. Create a new GitHub repository
2. Push your code:
```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
git push -u origin main
```

### Step 2: Docker Hub Setup

1. Create account at https://hub.docker.com
2. Create two repositories:
   - `YOUR_USERNAME/mean-app-backend`
   - `YOUR_USERNAME/mean-app-frontend`
3. Generate access token: Account Settings → Security → New Access Token
4. Add `DOCKER_USERNAME` and `DOCKER_PASSWORD` to GitHub Secrets

### Step 3: EC2 Instance Setup

#### Launch EC2 Instance
1. Choose **Ubuntu Server 22.04 LTS** AMI
2. Instance type: **t2.medium** or higher (recommended)
3. Configure security group to allow:
   - SSH (port 22) from your IP
   - HTTP (port 80) from anywhere (0.0.0.0/0)
   - Custom TCP (port 8080) from anywhere (optional, for direct backend access)
4. Create or use existing key pair
5. Launch instance

#### Configure EC2 Instance

SSH into your instance:
```bash
ssh -i your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

Install Docker:
```bash
# Update packages
sudo apt-get update

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add ubuntu user to docker group
sudo usermod -aG docker ubuntu

# Install Docker Compose
sudo apt-get install docker-compose-plugin -y

# Verify installation
docker --version
docker compose version
```

Create project directory:
```bash
mkdir -p ~/crud-dd-task-mean-app
cd ~/crud-dd-task-mean-app
```

Create `docker-compose.yml`:
```bash
nano docker-compose.yml
```

Paste the following (replace `YOUR_DOCKERHUB_USERNAME`):
```yaml
version: '3.8'

services:
  mongodb:
    image: mongo:6
    container_name: mean-mongodb
    restart: unless-stopped
    environment:
      - MONGO_INITDB_DATABASE=dd_db
    volumes:
      - mongodb_data:/data/db
    networks:
      - mean-network

  backend:
    image: YOUR_DOCKERHUB_USERNAME/mean-app-backend:latest
    container_name: mean-backend
    restart: unless-stopped
    environment:
      - MONGODB_URI=mongodb://mongodb:27017/dd_db
      - PORT=8080
    depends_on:
      - mongodb
    networks:
      - mean-network

  frontend:
    image: YOUR_DOCKERHUB_USERNAME/mean-app-frontend:latest
    container_name: mean-frontend
    restart: unless-stopped
    depends_on:
      - backend
    networks:
      - mean-network

  nginx:
    image: nginx:alpine
    container_name: mean-nginx-proxy
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - ./nginx-proxy.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - frontend
      - backend
    networks:
      - mean-network

networks:
  mean-network:
    driver: bridge

volumes:
  mongodb_data:
```

Create `nginx-proxy.conf`:
```bash
nano nginx-proxy.conf
```

Paste the Nginx configuration (see `nginx-proxy.conf` in this repo).

Log out and log back in for docker group changes to take effect.

### Step 4: Configure GitHub Actions Secrets

1. Go to your GitHub repository
2. Navigate to: Settings → Secrets and variables → Actions
3. Click "New repository secret" and add:
   - `DOCKER_USERNAME`: Your Docker Hub username
   - `DOCKER_PASSWORD`: Your Docker Hub access token
   - `EC2_HOST`: Your EC2 public IP address
   - `EC2_USERNAME`: `ubuntu`
   - `EC2_SSH_KEY`: Content of your private key file (`.pem` file)

### Step 5: Trigger Deployment

1. Make any change to your code (or just update README)
2. Commit and push to main branch:
```bash
git add .
git commit -m "Trigger CI/CD"
git push origin main
```

3. Check GitHub Actions tab to monitor workflow
4. Once complete, access your application at: `http://YOUR_EC2_PUBLIC_IP`

## 📁 Project Structure

```
crud-dd-task-mean-app/
├── backend/
│   ├── app/
│   │   ├── config/
│   │   │   └── db.config.js          # MongoDB connection
│   │   ├── controllers/
│   │   │   └── tutorial.controller.js
│   │   ├── models/
│   │   │   ├── index.js
│   │   │   └── tutorial.model.js
│   │   └── routes/
│   │       └── tutorial.routes.js
│   ├── server.js                      # Entry point
│   ├── package.json
│   ├── Dockerfile                     # Backend Docker config
│   └── .dockerignore
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   │   ├── components/
│   │   │   ├── models/
│   │   │   └── services/
│   │   │       └── tutorial.service.ts # API client
│   │   ├── assets/
│   │   └── index.html
│   ├── angular.json
│   ├── package.json
│   ├── Dockerfile                     # Frontend Docker config
│   ├── nginx.conf                     # Angular app Nginx config
│   └── .dockerignore
├── .github/
│   └── workflows/
│       └── deploy.yml                 # CI/CD pipeline
├── docker-compose.yml                 # Local orchestration
├── nginx-proxy.conf                   # Reverse proxy config
└── README.md                          # This file
```

## 📸 Screenshots

### CI/CD Pipeline Execution
> Screenshot showing successful GitHub Actions workflow

*[Add screenshot of GitHub Actions workflow success]*

### Docker Hub Images
> Docker Hub repositories showing pushed images

*[Add screenshot of Docker Hub repositories with backend and frontend images]*

### Application UI
> Working application interface

*[Add screenshots of:]*
- *Tutorials list view*
- *Add tutorial form*
- *Edit tutorial*
- *Search functionality*

### Nginx Configuration
> Nginx reverse proxy setup

*[Add screenshot of nginx configuration or docker-compose logs showing nginx running]*

### Infrastructure Setup
> EC2 instance details and running containers

*[Add screenshots of:]*
- *EC2 instance details*
- *SSH session showing `docker ps` output*
- *Application accessible via EC2 public IP*

## 🔧 Troubleshooting

### Container not starting
```bash
docker-compose logs <service-name>
```

### Cannot connect to MongoDB
- Ensure MongoDB container is running: `docker ps | grep mongodb`
- Check connection string in `backend/app/config/db.config.js`

### Frontend not loading
- Check if build was successful: `docker-compose logs frontend`
- Verify nginx configuration is correct

### CI/CD failing
- Verify all GitHub Secrets are correctly set
- Check Docker Hub credentials
- Ensure EC2 instance is accessible via SSH

## 📝 License

This project is open source and available under the [MIT License](LICENSE).

## 👨‍💻 Author

Created for DevOps assignment demonstrating:
- Docker containerization
- CI/CD automation with GitHub Actions
- Cloud deployment on AWS EC2
- Reverse proxy configuration with Nginx

---

**Note:** Remember to replace `YOUR_DOCKERHUB_USERNAME` with your actual Docker Hub username in the docker-compose.yml file on your EC2 instance and in GitHub Actions workflow.
