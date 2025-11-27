# Simple Deployment Guide - Clone from GitHub

This guide shows how to deploy by cloning the repository directly on EC2 (simpler approach without Docker Hub).

## Prerequisites

- GitHub repository with your code
- AWS EC2 instance (Ubuntu 22.04)
- SSH access to EC2

---

## Step 1: Push Code to GitHub

On your local machine:

```bash
cd c:\Users\karth\Downloads\crud-dd-task-mean-app\crud-dd-task-mean-app

# Initialize git (if not done)
git init

# Add all files
git add .

# Commit
git commit -m "Initial commit: MEAN app with Docker"

# Add your GitHub repo
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git

# Push to GitHub
git branch -M main
git push -u origin main
```

---

## Step 2: Set Up EC2 Instance

### 2.1 Launch EC2

1. Go to AWS EC2 Console
2. Click "Launch Instance"
3. **Configuration:**
   - Name: `MEAN-App-Server`
   - AMI: **Ubuntu Server 22.04 LTS**
   - Instance type: **t2.medium** (recommended)
   - Key pair: Create new or select existing
   - Security group: Allow **SSH (22)** and **HTTP (80)**
   - Storage: 20 GB minimum

4. Launch and note the **Public IPv4 address**

### 2.2 Connect to EC2

```bash
# Windows (PowerShell or Git Bash)
ssh -i path\to\your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP

# macOS/Linux
chmod 400 /path/to/your-key.pem
ssh -i /path/to/your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

---

## Step 3: Install Docker on EC2

Once connected to EC2, run these commands:

```bash
# Update system
sudo apt-get update

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add ubuntu user to docker group
sudo usermod -aG docker ubuntu

# Install Docker Compose
sudo apt-get install -y docker-compose-plugin

# Verify installation
docker --version
docker compose version
```

**IMPORTANT:** Log out and log back in for group changes to take effect:
```bash
exit
# SSH back in
ssh -i your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

---

## Step 4: Clone Repository on EC2

```bash
# Clone your repository
git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git

# Navigate to project directory
cd YOUR_REPO

# Verify files are there
ls -la
```

You should see:
- `backend/`
- `frontend/`
- `docker-compose.yml`
- `nginx-proxy.conf`
- etc.

---

## Step 5: Build and Run with Docker Compose

```bash
# Build all services (this will take 5-10 minutes)
docker-compose build

# Start all services in background
docker-compose up -d

# Check if all containers are running
docker ps
```

You should see 4 containers running:
- `mean-mongodb`
- `mean-backend`
- `mean-frontend`
- `mean-nginx-proxy`

---

## Step 6: Verify Deployment

### Check Container Status

```bash
# View all containers
docker ps

# Check logs if needed
docker-compose logs backend
docker-compose logs frontend
docker-compose logs nginx
```

### Test the Application

**On EC2:**
```bash
# Test backend API
curl http://localhost/api/tutorials

# Should return: []
```

**On your browser:**
```
http://YOUR_EC2_PUBLIC_IP
```

You should see the Angular application! 🎉

---

## Step 7: Making Updates (Manual Deployment)

When you make code changes:

### On Local Machine:
```bash
# Make your changes
# Commit and push
git add .
git commit -m "Updated feature X"
git push origin main
```

### On EC2:
```bash
# SSH into EC2
ssh -i your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP

# Navigate to project
cd YOUR_REPO

# Pull latest changes
git pull origin main

# Rebuild and restart
docker-compose down
docker-compose build
docker-compose up -d

# Verify
docker ps
```

---

## Useful Commands

### View Logs
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f backend
docker-compose logs -f frontend
```

### Restart Services
```bash
# Restart all
docker-compose restart

# Restart specific service
docker-compose restart backend
```

### Stop Everything
```bash
docker-compose down
```

### Check MongoDB Data
```bash
docker exec -it mean-mongodb mongosh
> use dd_db
> db.tutorials.find()
> exit
```

### Clean Up Old Images
```bash
docker image prune -f
```

---

## Comparison: Simple vs CI/CD Approach

### Simple Approach (This Guide)

**Workflow:**
```
1. Write code locally
2. git push to GitHub
3. SSH to EC2
4. git pull
5. docker-compose down
6. docker-compose build
7. docker-compose up -d
```

**Time per deployment:** ~10-15 minutes (includes build time on EC2)

**Best for:** Learning, testing, small projects

---

### CI/CD Approach (Automated)

**Workflow:**
```
1. Write code locally
2. git push to GitHub
3. Wait 5 minutes
4. Done! ✅
```

**Time per deployment:** 1 minute (your work) + 5 minutes (automatic)

**Best for:** Production, team projects, professional portfolio

---

## Upgrading to CI/CD Later

If you want to switch to CI/CD automation later:

1. Create Docker Hub account
2. Add GitHub Secrets (Docker Hub + EC2 credentials)
3. GitHub Actions will handle deployment automatically
4. See `DEPLOYMENT.md` for full CI/CD setup

---

## Troubleshooting

### Issue: `docker-compose: command not found`

**Solution:**
```bash
# Use docker compose (with space)
docker compose up -d
```

Or install docker-compose v1:
```bash
sudo apt-get install docker-compose
```

### Issue: Permission denied

**Solution:**
Make sure you logged out and back in after adding user to docker group.

### Issue: Containers not starting

**Solution:**
```bash
# Check logs
docker-compose logs

# Check specific service
docker-compose logs backend

# Common issue: MongoDB not ready
# Wait 10 seconds and try again
docker-compose restart backend
```

### Issue: Can't access application

**Solution:**
- Check EC2 security group allows port 80
- Verify nginx container is running: `docker ps | grep nginx`
- Check nginx logs: `docker-compose logs nginx`

---

## Summary

✅ **Pros of this approach:**
- Simple and straightforward
- No Docker Hub needed
- Good for learning
- Full control

⚠️ **Cons:**
- Manual deployment required
- Build takes time on EC2
- Uses EC2 resources for building

**Next Steps:**
- Test the application
- Make changes and practice deploying
- When ready, upgrade to CI/CD for automation!

---

**🎉 Congratulations!** Your MEAN stack app is now running on EC2!
