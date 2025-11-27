# Quick Start Script for EC2 Deployment

This script helps you quickly get the application running on your EC2 instance.

## Prerequisites
- EC2 instance with Ubuntu 22.04
- Docker and Docker Compose installed
- Docker Hub account with images pushed

## Steps

### 1. SSH into EC2
```bash
ssh -i your-key.pem ubuntu@YOUR_EC2_IP
```

### 2. Create project directory
```bash
mkdir -p ~/crud-dd-task-mean-app
cd ~/crud-dd-task-mean-app
```

### 3. Create docker-compose.yml file

Copy the content from `docker-compose.prod.yml` in this repository, ensuring you replace `YOUR_DOCKERHUB_USERNAME` with your actual Docker Hub username.

### 4. Create nginx-proxy.conf file

Copy the content from `nginx-proxy.conf` in this repository.

### 5. Pull and run

```bash
# Pull the latest images
docker compose pull

# Start all services
docker compose up -d

# Check status
docker compose ps

# View logs
docker compose logs -f
```

### 6. Verify

```bash
# Check all containers are running
docker ps

# Test the API
curl http://localhost/api/tutorials

# You should see: []
```

### 7. Access application

Open browser: `http://YOUR_EC2_PUBLIC_IP`

## Useful Commands

### View logs
```bash
docker compose logs backend
docker compose logs frontend
docker compose logs nginx
docker compose logs mongodb
```

### Restart services
```bash
docker compose restart
```

### Stop services
```bash
docker compose down
```

### Update to latest images
```bash
docker compose pull
docker compose up -d
```

### Clean up old images
```bash
docker image prune -f
```

### Check MongoDB data
```bash
docker exec -it mean-mongodb mongosh
> use dd_db
> db.tutorials.find()
```
