# Complete EC2 Setup Guide - Step by Step

This guide will help you launch an EC2 instance from scratch, including creating your SSH key.

---

## 🎯 Step 1: Create AWS Account (If Not Done)

1. Go to https://aws.amazon.com/
2. Click "Create an AWS Account"
3. Follow the signup process (requires credit card, but we'll use free tier)
4. Login to AWS Console

---

## 🔑 Step 2: Create SSH Key Pair

1. **Login to AWS Console:** https://console.aws.amazon.com/

2. **Go to EC2:**
   - Search for "EC2" in the top search bar
   - Click "EC2" (Virtual Servers in the Cloud)

3. **Navigate to Key Pairs:**
   - In the left sidebar, scroll down to "Network & Security"
   - Click **"Key Pairs"**

4. **Create New Key Pair:**
   - Click **"Create key pair"** button (orange button, top right)
   
   **Fill in the details:**
   - **Name:** `mean-app-key` (or any name you prefer)
   - **Key pair type:** RSA
   - **Private key file format:** 
     - **For Windows:** Select `.pem` (works with PowerShell/Git Bash)
     - **For PuTTY users:** Select `.ppk`
   
   - Click **"Create key pair"**

5. **IMPORTANT:** The `.pem` file will **automatically download**
   - Save it in a safe place (e.g., `C:\Users\karth\Downloads\`)
   - **You can NEVER download it again!** Keep it safe!

---

## 🖥️ Step 3: Launch EC2 Instance

### 3.1 Start Launch Process

1. **Go to EC2 Dashboard:**
   - Click "EC2 Dashboard" in the left sidebar
   - Click **"Launch instance"** button (orange button)

### 3.2 Configure Instance

**Step 1: Name and tags**
- **Name:** `MEAN-App-Server`

**Step 2: Application and OS Images (Amazon Machine Image)**
- Click **"Ubuntu"**
- Select **"Ubuntu Server 22.04 LTS"**
- Architecture: **64-bit (x86)**
- ✅ Make sure it says "Free tier eligible"

**Step 3: Instance type**
- Select **t2.medium** (Recommended)
  - OR **t2.small** (Minimum for this project)
  - OR **t2.micro** (Free tier, but might be slow for builds)

**Step 4: Key pair**
- **Select:** The key pair you just created (`mean-app-key`)
- ✅ Should show the key pair name in dropdown

**Step 5: Network settings**

Click **"Edit"** button next to Network settings

**Firewall (security groups):**
- Select **"Create security group"**
- **Security group name:** `mean-app-sg`
- **Description:** `Security group for MEAN app`

**Inbound Security Group Rules:**

Add these rules:

| Type | Protocol | Port Range | Source | Description |
|------|----------|------------|--------|-------------|
| SSH | TCP | 22 | My IP | SSH access |
| HTTP | TCP | 80 | Anywhere (0.0.0.0/0) | Web access |
| Custom TCP | TCP | 8080 | Anywhere (0.0.0.0/0) | Backend API (optional) |

**How to add rules:**
- Rule 1 is already there (SSH)
  - Change **Source** to "My IP" (automatic)
- Click **"Add security group rule"**
  - Type: HTTP
  - Source: Anywhere (0.0.0.0/0)
- Click **"Add security group rule"** again
  - Type: Custom TCP
  - Port: 8080
  - Source: Anywhere (0.0.0.0/0)

**Step 6: Configure storage**
- **Size:** 20 GB (minimum)
- **Volume type:** gp3 (General Purpose SSD)
- Leave other settings as default

**Step 7: Advanced details**
- Leave as default (skip this section)

### 3.3 Launch Instance

1. **Review Summary** on the right side
2. Click **"Launch instance"** (orange button, bottom right)
3. Wait for confirmation message: "Successfully initiated launch of instance"
4. Click **"View all instances"**

---

## ✅ Step 4: Get Your Instance Details

1. **On EC2 Instances page:**
   - You'll see your instance with status "Initializing" or "Running"
   - Wait until **Instance State** = **Running** ✅
   - Wait until **Status check** = **2/2 checks passed** ✅ (takes 2-3 minutes)

2. **Get Public IP:**
   - Click on your instance (checkbox or row)
   - Look for **"Public IPv4 address"** in the details below
   - Example: `54.123.45.67`
   - **Copy this IP address!** You'll need it for SSH

3. **Important Instance Details:**
   - **Instance ID:** i-xxxxxxxxxxxxx
   - **Public IPv4 address:** [Copy this!]
   - **Private IPv4 address:** (ignore for now)
   - **Key pair name:** mean-app-key

---

## 🔐 Step 5: Connect via SSH

Now that you have:
- ✅ EC2 instance running
- ✅ Public IP address
- ✅ Key pair file (.pem)

### 5.1 Prepare Your Key File

**Open PowerShell:**
- Press `Win + X`
- Select "Windows PowerShell" or "Terminal"

**Navigate to where your key is:**
```powershell
cd C:\Users\karth\Downloads
```

**Check if the key file is there:**
```powershell
dir *.pem
```
You should see `mean-app-key.pem` (or whatever you named it)

**Fix permissions (important!):**
```powershell
# Remove inherited permissions
icacls mean-app-key.pem /inheritance:r

# Grant read permission to yourself only
icacls mean-app-key.pem /grant:r "%username%:R"
```

### 5.2 SSH into EC2

```powershell
ssh -i mean-app-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

**Replace:**
- `YOUR_EC2_PUBLIC_IP` with the actual IP you copied earlier

**Example:**
```powershell
ssh -i mean-app-key.pem ubuntu@54.123.45.67
```

### 5.3 First Connection

**First time connecting, you'll see:**
```
The authenticity of host '54.123.45.67' can't be established.
ED25519 key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

**Type:** `yes` and press Enter

**You should now see:**
```
Welcome to Ubuntu 22.04.3 LTS
ubuntu@ip-xxx-xx-xx-xxx:~$
```

**🎉 Congratulations! You're connected to EC2!**

---

## 📦 Step 6: Install Docker (While Connected)

Now run these commands one by one:

```bash
# Update package list
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

**IMPORTANT:** Log out and log back in:
```bash
exit
```

Then SSH back in:
```powershell
ssh -i mean-app-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

---

## 🚀 Step 7: Deploy Your Application

```bash
# Clone your repository
git clone https://github.com/Karthikk29/Crud_Deploy_app.git

# Navigate to project
cd Crud_Deploy_app

# Build and start all services
docker-compose build
docker-compose up -d

# Check if containers are running
docker ps
```

You should see 4 containers:
- mean-mongodb
- mean-backend
- mean-frontend
- mean-nginx-proxy

---

## 🌐 Step 8: Access Your Application

Open your browser and go to:
```
http://YOUR_EC2_PUBLIC_IP
```

**Example:**
```
http://54.123.45.67
```

**You should see your Angular application!** 🎉

---

## 🔍 Troubleshooting

### SSH Connection Issues

**Issue 1: "Permission denied (publickey)"**
```powershell
# Make sure you're using the correct username
ssh -i mean-app-key.pem ubuntu@YOUR_IP
#                          ^^^^^^ - must be "ubuntu" for Ubuntu instances
```

**Issue 2: "WARNING: UNPROTECTED PRIVATE KEY FILE!"**
```powershell
# Fix permissions again
icacls mean-app-key.pem /inheritance:r
icacls mean-app-key.pem /grant:r "%username%:R"
```

**Issue 3: "Connection refused" or "Connection timed out"**
- Check EC2 instance is **Running** (not Stopped)
- Check Security Group allows SSH (port 22) from your IP
- Check you're using the correct Public IP

**Issue 4: "ssh: command not found"**
- You need OpenSSH client
- **Option 1:** Use Git Bash instead of PowerShell
- **Option 2:** Install OpenSSH:
  - Settings → Apps → Optional Features → Add OpenSSH Client

### Application Access Issues

**Issue: Can't access http://EC2_IP**
- Check Security Group allows HTTP (port 80)
- Verify all containers are running: `docker ps`
- Check nginx logs: `docker-compose logs nginx`

**Issue: Containers not starting**
```bash
# Check logs
docker-compose logs

# Restart
docker-compose down
docker-compose up -d
```

---

## 📝 Important Information to Save

**Keep these safe:**

1. **EC2 Public IP:** `_________________`
2. **Key file location:** `C:\Users\karth\Downloads\mean-app-key.pem`
3. **SSH command:**
   ```
   ssh -i C:\Users\karth\Downloads\mean-app-key.pem ubuntu@YOUR_IP
   ```

---

## 💰 Cost Management

**Free Tier Limits:**
- **750 hours/month** of t2.micro (Free for 12 months)
- **30 GB** of EBS storage (Free for 12 months)

**To avoid charges:**
- **Stop instance** when not using (right-click instance → Instance State → Stop)
- **Terminate instance** when completely done (deletes everything!)

**Monthly cost if you exceed free tier:**
- t2.micro: ~$8-10/month
- t2.small: ~$16-20/month
- t2.medium: ~$33-40/month

---

## ✅ Quick Reference

**Start instance:**
```
AWS Console → EC2 → Instances → Select → Instance State → Start
```

**Stop instance:**
```
AWS Console → EC2 → Instances → Select → Instance State → Stop
```

**SSH command:**
```powershell
ssh -i mean-app-key.pem ubuntu@YOUR_EC2_IP
```

**Check application:**
```
http://YOUR_EC2_IP
```

---

**🎉 You're all set!** Your EC2 instance is ready for deployment!
