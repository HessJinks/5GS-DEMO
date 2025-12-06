# 5G Core Network Deployment - Documentation Index

Welcome to your Open5GS + UERANSIM 5G Core Network deployment!

---

## 📚 Documentation Files

### 🚀 Start Here
1. **[QUICKSTART.md](QUICKSTART.md)** - Get running in 10 minutes!
   - Fastest way to deploy
   - Step-by-step commands
   - No explanations, just actions

### 📖 Main Documentation
2. **[README.md](README.md)** - Complete deployment guide
   - Full architecture overview
   - Detailed component descriptions
   - Configuration explanations
   - Troubleshooting guide
   - Use cases and limitations

### 📊 Technical Summary
3. **[DEPLOYMENT_SUMMARY.md](DEPLOYMENT_SUMMARY.md)** - What was accomplished
   - Deployment status and test results
   - All issues resolved during setup
   - Configuration highlights
   - Critical patterns and lessons learned
   - Performance metrics

---

## 🗂️ Project Structure

```
5GS/
├── 📄 Documentation
│   ├── INDEX.md                    ← You are here
│   ├── QUICKSTART.md               ← Start here for quick deploy
│   ├── README.md                   ← Complete guide
│   └── DEPLOYMENT_SUMMARY.md       ← Technical details
│
├── 🐳 Docker Images
│   ├── Dockerfile.open5gs          ← Open5GS build (all NFs)
│   ├── Dockerfile.ueransim         ← UERANSIM build (UE + gNB)
│   ├── Dockerfile.webui            ← WebUI build
│   └── upf-entrypoint.sh           ← UPF startup script
│
├── ☸️ Kubernetes Cluster
│   └── kind-5gs.yaml               ← Kind cluster config
│
├── 📦 Kubernetes Manifests
│   ├── k8s/open5gs/
│   │   ├── MongoDB
│   │   │   └── mongodb.yaml
│   │   │
│   │   ├── Core NFs (no config needed)
│   │   │   ├── nrf.yaml
│   │   │   ├── scp.yaml
│   │   │   ├── ausf.yaml
│   │   │   ├── udm.yaml
│   │   │   ├── udr.yaml
│   │   │   ├── pcf.yaml
│   │   │   ├── nssf.yaml
│   │   │   └── bsf.yaml
│   │   │
│   │   ├── Main NFs (with ConfigMaps)
│   │   │   ├── configmap-amf.yaml
│   │   │   ├── amf.yaml
│   │   │   ├── configmap-smf.yaml
│   │   │   ├── smf.yaml
│   │   │   ├── configmap-upf.yaml
│   │   │   ├── upf.yaml
│   │   │   └── webui.yaml
│   │   │
│   │   └── Additional ConfigMaps
│   │       ├── configmap-ausf.yaml
│   │       ├── configmap-bsf.yaml
│   │       ├── configmap-nrf.yaml
│   │       ├── configmap-nssf.yaml
│   │       ├── configmap-pcf.yaml
│   │       ├── configmap-scp.yaml
│   │       ├── configmap-udm.yaml
│   │       └── configmap-udr.yaml
│   │
│   └── k8s/ueransim/
│       ├── configmap-gnb.yaml      ← gNB configuration
│       ├── configmap-ue.yaml       ← UE configuration
│       └── gnb-ue.yaml             ← Combined gNB+UE pod
│
├── 🛠️ Deployment Scripts
│   ├── build-and-deploy.sh         ← Complete build & deploy (first-time setup)
│   ├── deploy.sh                   ← Fast deploy only (redeploy/testing)
│   └── cleanup.sh                  ← Clean removal of all resources
│
└── 📁 Source Code
    ├── open5gs/                    ← Open5GS source
    └── UERANSIM/                   ← UERANSIM source
```

---

## 🎯 Quick Navigation

### I want to...

**Deploy the 5G network**
→ Go to [QUICKSTART.md](QUICKSTART.md)

**Understand the architecture**
→ Go to [README.md - Architecture](README.md#architecture)

**Troubleshoot issues**
→ Go to [README.md - Troubleshooting](README.md#troubleshooting)
→ Or [DEPLOYMENT_SUMMARY.md - Issues Resolved](DEPLOYMENT_SUMMARY.md#key-issues-resolved)

**Add more subscribers**
→ Go to [README.md - Testing](README.md#add-subscriber-to-database)

**See what's working**
→ Go to [DEPLOYMENT_SUMMARY.md - Test Results](DEPLOYMENT_SUMMARY.md#test-results)

**Learn about limitations**
→ Go to [README.md - Known Limitations](README.md#known-limitations)

**Understand the configuration**
→ Go to [DEPLOYMENT_SUMMARY.md - Configuration Highlights](DEPLOYMENT_SUMMARY.md#configuration-highlights)

**Access the WebUI**
→ http://localhost:30999 (admin / 1423)

---

## 📋 Deployment Checklist

### Automated Deployment (Recommended)
- [ ] Docker installed
- [ ] kubectl installed
- [ ] kind installed
- [ ] Created kind cluster (`kind create cluster --config kind-5gs.yaml`)
- [ ] Run `./build-and-deploy.sh` (handles all steps below automatically)
- [ ] Verified UE registration successful
- [ ] Verified PDU session established
- [ ] Accessed WebUI at http://localhost:30999

### Manual Deployment
- [ ] Docker installed
- [ ] kubectl installed
- [ ] kind installed
- [ ] Created kind cluster (`kind create cluster --config kind-5gs.yaml`)
- [ ] Built Docker images (open5gs, ueransim, webui)
- [ ] Loaded images into kind cluster
- [ ] Created open5gs namespace
- [ ] Deployed all Kubernetes manifests
- [ ] Added admin account and subscriber to database
- [ ] Verified UE registration successful
- [ ] Verified PDU session established
- [ ] Accessed WebUI at http://localhost:30999

---

## 🔍 Current Deployment Status

**Cluster**: kind-5gs
**Namespace**: open5gs
**Status**: ✅ Fully Operational

**What's Working:**
- ✅ UE Registration
- ✅ Authentication (AUSF/UDM/UDR)
- ✅ PDU Session Establishment
- ✅ GTP-U Tunnels
- ✅ PFCP Sessions
- ✅ All SBI Communication

**What's Not Working:**
- ❌ Internet access from UE (kind networking limitation)

**Suitable For:**
- 5G protocol testing
- Development and debugging
- Learning 5G architecture
- Integration testing

---

## 📞 Quick Commands

### Check deployment status
```bash
kubectl get pods -n open5gs
```

### View UE logs
```bash
kubectl logs -n open5gs deployment/ueransim-gnb -c ue --tail=50
```

### View AMF logs
```bash
kubectl logs -n open5gs deployment/amf --tail=50
```

### View SMF logs
```bash
kubectl logs -n open5gs deployment/smf --tail=50
```

### View UPF logs
```bash
kubectl logs -n open5gs deployment/upf --tail=50
```

### Restart UE/gNB
```bash
kubectl rollout restart deployment/ueransim-gnb -n open5gs
```

### Clean up deployment
```bash
./cleanup.sh
```

### Delete everything (including cluster)
```bash
./cleanup.sh
kind delete cluster --name kind-5gs
```

---

## 🛠️ Deployment Scripts

### `build-and-deploy.sh` - First-Time Setup
```bash
./build-and-deploy.sh
```
**Purpose**: Complete build and deployment from scratch
**Time**: 30-40 minutes
**Use when**: First-time setup or rebuilding images
**Does**:
- Builds all Docker images (Open5GS, UERANSIM, WebUI)
- Loads images into kind cluster
- Deploys all resources
- Creates admin account and subscriber

---

### `deploy.sh` - Fast Redeploy
```bash
./deploy.sh
```
**Purpose**: Deploy using existing images
**Time**: 3-5 minutes
**Use when**: Testing, redeploying after cleanup
**Does**:
- Creates namespace
- Deploys all Kubernetes resources in correct order
- Creates admin account and subscriber
- Waits for components to be ready

---

### `cleanup.sh` - Clean Removal
```bash
./cleanup.sh
```
**Purpose**: Remove deployment (keep cluster and images)
**Time**: 1 minute
**Use when**: Testing different configs, cleaning up
**Does**:
- Deletes all deployed resources
- Keeps kind cluster running
- Keeps Docker images for fast redeployment

**Redeploy after cleanup:**
```bash
./deploy.sh  # Fast! Uses existing images
```

---

## 🔧 Key Configuration Values

| Parameter | Value |
|-----------|-------|
| **PLMN** | MCC=999, MNC=70 |
| **TAC** | 1 |
| **UE Subnet** | 10.45.0.0/16 |
| **Gateway** | 10.45.0.1 |
| **DNS** | 8.8.8.8, 8.8.4.4 |
| **IMSI** | 999700000000001 |
| **K** | 465B5CE8B199B49FAA5F0A2EE238A6BC |
| **OPc** | E8ED289DEBA952E4283B54E88E6183CA |
| **DNN** | internet |
| **WebUI Port** | 30999 |
| **WebUI User** | admin |
| **WebUI Pass** | 1423 |

---

## 🌟 Key Insights

### Critical Success Factors

1. **FQDN Advertise Addresses**
   - All NFs must use `<name>.open5gs.svc.cluster.local`
   - Required on SBI, PFCP, and GTPU interfaces

2. **UERANSIM Pod Design**
   - gNB and UE must be in same pod
   - UE connects to gNB via 127.0.0.1

3. **UPF Network Setup**
   - Entrypoint script creates ogstun interface
   - NAT rules configured automatically
   - iptables required in container

4. **Timing**
   - UE starts 15 seconds after gNB
   - PFCP association needs ~30 seconds

---

## 📚 External References

- [Open5GS Official Docs](https://open5gs.org)
- [UERANSIM GitHub](https://github.com/aligungr/UERANSIM)
- [3GPP 5G Specs](https://www.3gpp.org)
- [Kubernetes Docs](https://kubernetes.io)
- [Kind Docs](https://kind.sigs.k8s.io)

---

## 🎓 Learning Resources

**5G Architecture:**
- [README.md - Architecture Section](README.md#architecture)
- Network Functions diagram and descriptions

**Deployment Process:**
- [DEPLOYMENT_SUMMARY.md - Issues Resolved](DEPLOYMENT_SUMMARY.md#key-issues-resolved)
- Learn from actual problems encountered and solved

**Configuration Patterns:**
- [DEPLOYMENT_SUMMARY.md - Critical Patterns](DEPLOYMENT_SUMMARY.md#critical-configuration-patterns)
- FQDN advertising, multi-interface setup, pod architecture

---

## ✅ Verification

Your deployment is working if you see:

```
[nas] [info] Initial Registration is successful
[nas] [info] PDU Session establishment is successful PSI[1]
[app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.X] is up
```

---

**Questions?**
- Check [README.md - Troubleshooting](README.md#troubleshooting)
- Review [DEPLOYMENT_SUMMARY.md](DEPLOYMENT_SUMMARY.md)
- Inspect pod logs with kubectl

**Ready to start?**
→ Head to [QUICKSTART.md](QUICKSTART.md)

---

Last Updated: December 2025
Deployment Status: ✅ Production-Ready for Testing
