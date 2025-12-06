# 5G Core Network - Quick Start Guide

Get your 5G network up and running in 10 minutes!

---

## Installation

### 1. Clone this repository
```bash
git clone <your-repo-url>
cd <repo-directory>
```

### 2. Clone source code
```bash
# Clone Open5GS
git clone https://github.com/open5gs/open5gs.git

# Clone UERANSIM
git clone https://github.com/aligungr/UERANSIM.git
```

---

## Two Deployment Options

### Option 1: Automated Script (Recommended)

Use the automated script for one-command deployment:

```bash
kind create cluster --config kind-5gs.yaml
./build-and-deploy.sh
```

This script automatically:
- Builds all Docker images (Open5GS, UERANSIM, WebUI)
- Loads images into kind cluster
- Deploys all Kubernetes manifests
- Creates admin account and subscriber

**Time**: ~30-40 minutes (mostly build time)

---

### Option 2: Manual Step-by-Step

For learning or debugging, follow the manual steps below.

---

## Prerequisites

- Docker installed
- kubectl installed
- kind installed
- 8GB+ RAM

---

## Step 1: Create Cluster (1 minute)

```bash
kind create cluster --config kind-5gs.yaml
```

---

## Step 2: Build Images (5-10 minutes)

```bash
# Build Open5GS (this takes the longest)
docker build -t local/open5gs:latest -f Dockerfile.open5gs .

# Build UERANSIM
docker build -t local/ueransim:latest -f Dockerfile.ueransim .

# Build WebUI
docker build -t local/webui:latest -f Dockerfile.webui .

# Load into kind cluster
kind load docker-image local/open5gs:latest --name kind-5gs
kind load docker-image local/ueransim:latest --name kind-5gs
kind load docker-image local/webui:latest --name kind-5gs
```

**TIP**: Run these builds in parallel to save time!

---

## Step 3: Deploy 5G Core (2 minutes)

```bash
# Create namespace
kubectl create namespace open5gs

# Deploy everything
kubectl apply -f k8s/open5gs/mongodb.yaml
kubectl apply -f k8s/open5gs/nrf.yaml
kubectl apply -f k8s/open5gs/scp.yaml
kubectl apply -f k8s/open5gs/ausf.yaml
kubectl apply -f k8s/open5gs/udm.yaml
kubectl apply -f k8s/open5gs/udr.yaml
kubectl apply -f k8s/open5gs/pcf.yaml
kubectl apply -f k8s/open5gs/nssf.yaml
kubectl apply -f k8s/open5gs/bsf.yaml

# Deploy configs and main NFs
kubectl apply -f k8s/open5gs/configmap-amf.yaml
kubectl apply -f k8s/open5gs/configmap-smf.yaml
kubectl apply -f k8s/open5gs/configmap-upf.yaml
kubectl apply -f k8s/open5gs/configmap-bsf.yaml
kubectl apply -f k8s/open5gs/configmap-nssf.yaml
kubectl apply -f k8s/open5gs/configmap-pcf.yaml

kubectl apply -f k8s/open5gs/amf.yaml
kubectl apply -f k8s/open5gs/smf.yaml
kubectl apply -f k8s/open5gs/upf.yaml
kubectl apply -f k8s/open5gs/webui.yaml

# Wait for pods to be ready
kubectl wait --for=condition=ready pod -l app=mongodb -n open5gs --timeout=120s
kubectl wait --for=condition=ready pod -l app=nrf -n open5gs --timeout=120s
kubectl wait --for=condition=ready pod -l app=amf -n open5gs --timeout=120s
```

---

## Step 4: Initialize Database (30 seconds)

Add admin user for WebUI:
```bash
kubectl exec -n open5gs deployment/webui -- sh -c "cat > /webui/create-admin.js << 'EOF'
const mongoose = require('mongoose');
const Schema = mongoose.Schema;
const passportLocalMongoose = require('passport-local-mongoose');

const Account = new Schema({
  roles: [String]
});

Account.plugin(passportLocalMongoose);
const AccountModel = mongoose.model('Account', Account);

mongoose.connect('mongodb://mongodb:27017/open5gs');

AccountModel.register(new AccountModel({
  username: 'admin',
  roles: ['admin']
}), '1423', function(err, account) {
  if (err) {
    console.error(err);
    process.exit(1);
  }
  console.log('Admin account created successfully');
  process.exit(0);
});
EOF
cd /webui && node create-admin.js"
```

Add subscriber:
```bash
kubectl run -it --rm debug --image=local/open5gs:latest --restart=Never -n open5gs -- \
  /bin/open5gs-dbctl add 999700000000001 465B5CE8B199B49FAA5F0A2EE238A6BC E8ED289DEBA952E4283B54E88E6183CA
```

You should see: `succeed`

---

## Step 5: Deploy UE/gNB (1 minute)

```bash
# Deploy configs
kubectl apply -f k8s/ueransim/configmap-gnb.yaml
kubectl apply -f k8s/ueransim/configmap-ue.yaml

# Deploy UERANSIM (combined gNB+UE pod)
kubectl apply -f k8s/ueransim/gnb-ue.yaml

# Wait for it to start
sleep 30
```

---

## Step 6: Verify! ✅

Check UE registration:
```bash
kubectl logs -n open5gs deployment/ueransim-gnb -c ue --tail=50
```

**Look for these SUCCESS messages:**
```
[nas] [info] Initial Registration is successful
[nas] [info] PDU Session establishment is successful PSI[1]
[app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.X] is up.
```

Check all pods running:
```bash
kubectl get pods -n open5gs
```

**Expected**: All pods should show `Running` and `1/1` or `2/2` READY

---

## Access WebUI

Open browser: http://localhost:30999

**Login:**
- Username: `admin`
- Password: `1423`

---

## Troubleshooting

### Pods not starting?
```bash
kubectl get pods -n open5gs
kubectl describe pod -n open5gs <pod-name>
```

### UE registration failed?
Check subscriber exists:
```bash
kubectl exec -n open5gs deployment/mongodb -- mongosh open5gs --eval "db.subscribers.find()"
```

### PDU session failed?
Restart UPF and UERANSIM:
```bash
kubectl rollout restart deployment/upf -n open5gs
sleep 30
kubectl rollout restart deployment/ueransim-gnb -n open5gs
```

---

## Clean Up

### Option 1: Using cleanup script (Recommended)
```bash
./cleanup.sh
```

This script will:
- Delete all UERANSIM resources
- Delete all Open5GS network functions
- Delete all ConfigMaps
- Delete WebUI and MongoDB
- Delete the namespace

### Option 2: Manual cleanup
```bash
kubectl delete namespace open5gs
kind delete cluster --name kind-5gs
```

---

## Deployment Scripts Reference

### `build-and-deploy.sh` - Complete Build & Deploy
**Use when**: First-time setup or rebuilding images

```bash
./build-and-deploy.sh
```

**What it does:**
1. Builds Open5GS Docker image (~10-20 min)
2. Builds UERANSIM Docker image (~5-10 min)
3. Builds WebUI Docker image (~5-10 min)
4. Loads all images into kind cluster
5. Updates Kubernetes manifests
6. Calls deploy.sh to deploy everything

**Time**: 30-40 minutes

---

### `deploy.sh` - Deploy Only (Fast)
**Use when**: Images already built, just deploying/redeploying

```bash
./deploy.sh
```

**What it does:**
1. Creates namespace
2. Deploys MongoDB
3. Deploys WebUI
4. Deploys all Open5GS ConfigMaps
5. Deploys all Open5GS network functions (in correct order)
6. Deploys UERANSIM gNB+UE
7. Creates admin account for WebUI
8. Adds default subscriber to database

**Time**: 3-5 minutes

**Perfect for:**
- Redeploying after cleanup
- Testing configuration changes
- When images are already built

---

### `cleanup.sh` - Clean Removal
**Use when**: Removing deployment (keeps cluster and images)

```bash
./cleanup.sh
```

**What it does:**
- Safely removes all deployed resources
- Keeps kind cluster running
- Keeps Docker images for fast redeployment
- Prompts for confirmation

**After cleanup, redeploy with:**
```bash
./deploy.sh  # Fast, uses existing images
```

---

## Common Workflows

### First Time Setup
```bash
kind create cluster --config kind-5gs.yaml
./build-and-deploy.sh
```

### Clean & Redeploy (Testing configs)
```bash
./cleanup.sh
./deploy.sh  # Much faster than rebuild!
```

### Rebuild Single Component
```bash
# Rebuild just WebUI
docker build -t local/webui:latest -f Dockerfile.webui .
kind load docker-image local/webui:latest --name kind-5gs
kubectl rollout restart deployment/webui -n open5gs
```

### Complete Teardown
```bash
./cleanup.sh
kind delete cluster --name kind-5gs
```

---

## What's Working?

✅ **Fully Functional:**
- UE Registration
- Authentication
- PDU Session Establishment
- GTP-U tunnels
- PFCP sessions
- All 5G control plane procedures

❌ **Not Working:**
- Internet access from UE (kind/Docker networking limitation)

**This is perfect for:**
- 5G protocol testing
- Development
- Learning 5G architecture
- Integration testing

---

## Next Steps

- Read `README.md` for detailed documentation
- Read `DEPLOYMENT_SUMMARY.md` for technical details
- Try adding more subscribers via WebUI
- Monitor logs to understand 5G procedures

---

## Quick Reference

| Component | Namespace | Check Logs |
|-----------|-----------|------------|
| UE | open5gs | `kubectl logs -n open5gs deployment/ueransim-gnb -c ue` |
| gNB | open5gs | `kubectl logs -n open5gs deployment/ueransim-gnb -c gnb` |
| AMF | open5gs | `kubectl logs -n open5gs deployment/amf` |
| SMF | open5gs | `kubectl logs -n open5gs deployment/smf` |
| UPF | open5gs | `kubectl logs -n open5gs deployment/upf` |
| WebUI | open5gs | `kubectl logs -n open5gs deployment/webui` |

---

**That's it! You now have a working 5G network!** 🎉
