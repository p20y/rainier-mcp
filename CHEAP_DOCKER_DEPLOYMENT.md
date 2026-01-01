# Cheap Docker Deployment Guide for Amazon Ads MCP Server

This guide covers the most cost-effective ways to deploy the Amazon Ads MCP Server using Docker.

## Resource Requirements

**Minimum Requirements:**
- **CPU**: 1 vCPU (shared is fine)
- **RAM**: 512MB - 1GB (1GB recommended)
- **Storage**: 2-5GB (for container + data)
- **Network**: Outbound HTTPS to Amazon Ads API and OpenBridge (if used)
- **Port**: 9080 (or any port you configure)

**Recommended for Production:**
- **CPU**: 1-2 vCPU
- **RAM**: 1-2GB
- **Storage**: 10GB (with room for exports/reports)

## Deployment Options (Cheapest to More Expensive)

### 1. 🆓 Free Tier Options

#### Option A: Fly.io (Recommended - Free Tier)
**Cost**: FREE (up to 3 shared-cpu-1x VMs with 256MB RAM)

```bash
# Install flyctl
curl -L https://fly.io/install.sh | sh

# Login
fly auth login

# Launch app
fly launch --name amazon-ads-mcp

# Set environment variables
fly secrets set AMAZON_AD_API_CLIENT_ID=xxx
fly secrets set AMAZON_AD_API_CLIENT_SECRET=xxx
fly secrets set AUTH_METHOD=direct

# Deploy
fly deploy
```

**Pros:**
- ✅ Free tier: 3 VMs, 3GB storage, 160GB outbound data/month
- ✅ Global edge network
- ✅ Automatic HTTPS
- ✅ Easy scaling

**Cons:**
- ⚠️ 256MB RAM per VM (may need to upgrade for large packages)
- ⚠️ Shared CPU (can be slow under load)

**Cost**: $0/month (free tier) or ~$5/month for 1GB RAM upgrade

#### Option B: Railway (Free Trial)
**Cost**: FREE trial ($5 credit), then ~$5/month

```bash
# Install Railway CLI
npm i -g @railway/cli

# Login
railway login

# Initialize project
railway init

# Add Dockerfile
# Railway auto-detects Dockerfile

# Set environment variables in Railway dashboard
# Deploy
railway up
```

**Pros:**
- ✅ $5 free credit
- ✅ Simple deployment
- ✅ Automatic HTTPS
- ✅ Good free tier

**Cons:**
- ⚠️ Limited free tier after trial

**Cost**: $0 (trial) → ~$5-10/month

#### Option C: Render (Free Tier)
**Cost**: FREE (with limitations)

```yaml
# render.yaml
services:
  - type: web
    name: amazon-ads-mcp
    dockerfilePath: ./Dockerfile
    envVars:
      - key: AMAZON_AD_API_CLIENT_ID
        sync: false
      - key: AMAZON_AD_API_CLIENT_SECRET
        sync: false
      - key: AUTH_METHOD
        value: direct
    healthCheckPath: /mcp/
```

**Pros:**
- ✅ Free tier available
- ✅ Automatic HTTPS
- ✅ Easy setup

**Cons:**
- ⚠️ Spins down after 15min inactivity (free tier)
- ⚠️ Limited resources

**Cost**: $0/month (free tier) or $7/month (starter)

### 2. 💰 Ultra-Low-Cost VPS Options

#### Option A: DigitalOcean App Platform
**Cost**: ~$5/month (Basic plan)

**Setup:**
1. Create account at digitalocean.com
2. Go to App Platform
3. Connect GitHub repo
4. Select Dockerfile
5. Set environment variables
6. Deploy

**Specs**: 512MB RAM, 1GB storage, 1 vCPU

#### Option B: Hetzner Cloud
**Cost**: ~€4/month (~$4.50/month)

**Setup:**
```bash
# Create VM (CPX11: 1 vCPU, 2GB RAM, 20GB SSD)
# SSH into server
ssh root@your-server-ip

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh

# Clone repo
git clone https://github.com/your-repo/amazon-ads-mcp.git
cd amazon-ads-mcp

# Create .env file
nano .env  # Add your credentials

# Start with docker-compose
docker-compose up -d
```

**Specs**: 1 vCPU, 2GB RAM, 20GB SSD
**Cost**: €4.15/month (~$4.50/month)

#### Option C: Vultr
**Cost**: $2.50/month (Regular Performance)

**Setup**: Similar to Hetzner
**Specs**: 1 vCPU, 512MB RAM, 10GB SSD
**Cost**: $2.50/month

#### Option D: Oracle Cloud Always Free
**Cost**: FREE forever

**Setup:**
1. Sign up for Oracle Cloud (requires credit card, but free tier is truly free)
2. Create Always Free VM (ARM or x86)
3. Install Docker
4. Deploy container

**Specs**: 
- ARM: 4 vCPU, 24GB RAM (Ampere A1)
- x86: 1 vCPU, 1GB RAM

**Pros:**
- ✅ Completely free (forever)
- ✅ Generous resources (ARM)

**Cons:**
- ⚠️ Account approval can take time
- ⚠️ Credit card required (but not charged)

**Cost**: $0/month (truly free)

### 3. 🐳 Container Platforms

#### Option A: Google Cloud Run
**Cost**: FREE tier (2 million requests/month), then pay-per-use

```bash
# Build and push to GCR
gcloud builds submit --tag gcr.io/PROJECT-ID/amazon-ads-mcp

# Deploy to Cloud Run
gcloud run deploy amazon-ads-mcp \
  --image gcr.io/PROJECT-ID/amazon-ads-mcp \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --set-env-vars AUTH_METHOD=direct
```

**Cost**: $0 (free tier) → ~$0.10-0.50/month (light usage)

#### Option B: AWS App Runner
**Cost**: ~$7/month minimum

**Setup**: Via AWS Console or CLI
**Cost**: $7.20/month (0.5 vCPU, 1GB RAM)

#### Option C: Azure Container Instances
**Cost**: Pay-per-use (~$5-10/month)

## Recommended: Cheapest Production Setup

### Option 1: Oracle Cloud Always Free (Best Value)
**Cost**: $0/month

```bash
# On Oracle Cloud VM
sudo apt update
sudo apt install docker.io docker-compose -y
sudo systemctl start docker
sudo systemctl enable docker

# Clone and deploy
git clone https://github.com/your-repo/amazon-ads-mcp.git
cd amazon-ads-mcp
nano .env  # Configure credentials
sudo docker-compose up -d
```

### Option 2: Fly.io Free Tier (Easiest)
**Cost**: $0/month (or $5/month for 1GB RAM)

```bash
# One-time setup
fly launch
fly secrets set AMAZON_AD_API_CLIENT_ID=xxx
fly secrets set AMAZON_AD_API_CLIENT_SECRET=xxx
fly deploy
```

### Option 3: Hetzner Cloud (Most Reliable Low-Cost)
**Cost**: €4.15/month (~$4.50/month)

Best balance of cost, performance, and reliability.

## Cost Optimization Tips

### 1. Minimize Package Loading
Only load packages you actually use:
```bash
# Minimal (cheaper)
AMAZON_AD_API_PACKAGES=profiles,exports-snapshots

# Full (more expensive - uses more RAM/CPU)
AMAZON_AD_API_PACKAGES=profiles,campaign-manage,sponsored-products,...
```

### 2. Disable Token Persistence
Saves storage I/O:
```bash
AMAZON_ADS_TOKEN_PERSIST=false
```

### 3. Use Minimal Logging
```bash
LOG_LEVEL=WARNING  # Instead of INFO or DEBUG
```

### 4. Resource Limits
Set Docker resource limits to prevent overuse:
```yaml
# docker-compose.yaml
services:
  amazon-ads-mcp:
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
```

### 5. Use Health Checks
Prevent unnecessary restarts:
```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:9080/mcp/"]
  interval: 30s
  timeout: 10s
  retries: 3
```

## Quick Deployment Scripts

### Fly.io Deployment
```bash
#!/bin/bash
# deploy-fly.sh

fly launch --name amazon-ads-mcp --region ord
fly secrets set AMAZON_AD_API_CLIENT_ID="$AMAZON_AD_API_CLIENT_ID"
fly secrets set AMAZON_AD_API_CLIENT_SECRET="$AMAZON_AD_API_CLIENT_SECRET"
fly secrets set AUTH_METHOD=direct
fly secrets set AMAZON_AD_API_PACKAGES="profiles,accounts-ads-accounts,exports-snapshots"
fly deploy
```

### Docker Compose on VPS
```bash
#!/bin/bash
# deploy-vps.sh

# Create .env from template
cat > .env << EOF
AUTH_METHOD=direct
AMAZON_AD_API_CLIENT_ID=$AMAZON_AD_API_CLIENT_ID
AMAZON_AD_API_CLIENT_SECRET=$AMAZON_AD_API_CLIENT_SECRET
AMAZON_AD_API_PACKAGES=profiles,accounts-ads-accounts,exports-snapshots
PORT=9080
TRANSPORT=http
EOF

# Start with docker-compose
docker-compose up -d

# Check logs
docker-compose logs -f
```

## Security Considerations for Public Deployment

### 1. Use HTTPS
- Fly.io, Railway, Render: Automatic HTTPS
- VPS: Use nginx reverse proxy with Let's Encrypt

### 2. Firewall Rules
Only expose port 9080 if needed, or use reverse proxy:
```bash
# UFW on Ubuntu/Debian
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

### 3. Environment Variables
Never commit `.env` file. Use:
- Platform secrets (Fly.io, Railway, etc.)
- Docker secrets
- External secret managers (for production)

## Cost Comparison Summary

| Provider | Cost/Month | RAM | CPU | Storage | Best For |
|----------|-----------|-----|-----|---------|----------|
| **Oracle Cloud Free** | $0 | 24GB (ARM) | 4 vCPU | 200GB | Best value |
| **Fly.io Free** | $0 | 256MB | Shared | 3GB | Easiest setup |
| **Hetzner** | €4.15 | 2GB | 1 vCPU | 20GB | Best reliability |
| **Vultr** | $2.50 | 512MB | 1 vCPU | 10GB | Cheapest paid |
| **Railway** | $5 | 512MB | Shared | 5GB | Developer-friendly |
| **Render** | $7 | 512MB | Shared | - | Simple deployment |

## Recommended Setup for Your Use Case

Based on your current setup (reporting + AMC packages), I recommend:

**For Development/Testing:**
- **Fly.io Free Tier** - Easiest, free, good enough for testing

**For Production:**
- **Hetzner Cloud** (€4.15/month) - Best balance of cost and reliability
- **Oracle Cloud Free** - If you can get approved, completely free

## Next Steps

1. **Choose a provider** from the options above
2. **Build Docker image** locally to test:
   ```bash
   docker build -t amazon-ads-mcp .
   docker run -p 9080:9080 --env-file .env amazon-ads-mcp
   ```
3. **Deploy** using provider-specific instructions
4. **Update Claude Desktop config** with your public URL

Would you like me to help you deploy to a specific provider?

