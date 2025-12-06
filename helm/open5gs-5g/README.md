# Open5GS 5G Core Network Helm Chart

A production-ready Helm chart for deploying Open5GS 5G Core Network and UERANSIM on Kubernetes.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- kind (for local deployment) or any Kubernetes cluster
- Docker images built and loaded:
  - `local/open5gs:latest`
  - `local/ueransim:latest`
  - `local/webui:latest`

## Quick Start

### 1. Build Docker Images

```bash
# From repository root
docker build -t local/open5gs:latest -f Dockerfile.open5gs .
docker build -t local/ueransim:latest -f Dockerfile.ueransim .
docker build -t local/webui:latest -f Dockerfile.webui .

# For kind cluster, load images
kind load docker-image local/open5gs:latest --name kind-5gs
kind load docker-image local/ueransim:latest --name kind-5gs
kind load docker-image local/webui:latest --name kind-5gs
```

### 2. Install the Chart

```bash
# Install with default values
helm install open5gs-5g ./helm/open5gs-5g

# Install with custom values
helm install open5gs-5g ./helm/open5gs-5g -f custom-values.yaml

# Install in a specific namespace
helm install open5gs-5g ./helm/open5gs-5g --create-namespace --namespace my-5g
```

### 3. Verify Installation

```bash
# Check all pods are running
kubectl get pods -n open5gs

# Watch pod startup
kubectl get pods -n open5gs -w

# Check UE registration logs
kubectl logs -n open5gs deployment/ueransim-gnb -c ue --tail=50
```

## Configuration

### Global Settings

| Parameter | Description | Default |
|-----------|-------------|---------|
| `global.namespace` | Kubernetes namespace | `open5gs` |
| `global.imagePullPolicy` | Image pull policy | `Never` |

### Network Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `network.plmn.mcc` | Mobile Country Code | `999` |
| `network.plmn.mnc` | Mobile Network Code | `70` |
| `network.tac` | Tracking Area Code | `1` |
| `network.ueSubnet` | UE IP subnet | `10.45.0.0/16` |
| `network.gateway` | Gateway IP | `10.45.0.1` |
| `network.dns.primary` | Primary DNS server | `8.8.8.8` |
| `network.dns.secondary` | Secondary DNS server | `8.8.4.4` |
| `network.mtu` | MTU size | `1400` |

### Subscriber Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `subscriber.imsi` | IMSI | `999700000000001` |
| `subscriber.key` | K (authentication key) | `465B5CE8B199B49FAA5F0A2EE238A6BC` |
| `subscriber.opc` | OPc (operator variant key) | `E8ED289DEBA952E4283B54E88E6183CA` |
| `subscriber.dnn` | Data Network Name | `internet` |

### WebUI Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `webui.enabled` | Enable WebUI | `true` |
| `webui.service.type` | Service type | `NodePort` |
| `webui.service.nodePort` | NodePort | `30999` |
| `webui.admin.username` | Admin username | `admin` |
| `webui.admin.password` | Admin password | `1423` |

### Open5GS Network Functions

Each network function can be configured with:
- `enabled`: Enable/disable the NF
- `replicas`: Number of replicas
- `resources`: Resource requests and limits

Example:
```yaml
open5gs:
  amf:
    enabled: true
    replicas: 1
    resources:
      requests:
        memory: "256Mi"
        cpu: "200m"
      limits:
        memory: "512Mi"
        cpu: "500m"
```

Supported NFs:
- `nrf` - Network Repository Function
- `scp` - Service Communication Proxy
- `ausf` - Authentication Server Function
- `udm` - Unified Data Management
- `udr` - Unified Data Repository
- `pcf` - Policy Control Function
- `nssf` - Network Slice Selection Function
- `bsf` - Binding Support Function
- `amf` - Access and Mobility Management Function
- `smf` - Session Management Function
- `upf` - User Plane Function

### UERANSIM Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ueransim.enabled` | Enable UERANSIM | `true` |
| `ueransim.gnb.replicas` | Number of gNB instances | `1` |
| `ueransim.ue.enabled` | Enable UE simulator | `true` |
| `ueransim.ue.delay` | Seconds to wait before UE starts | `15` |

## Examples

### Custom Network Configuration

```yaml
# custom-values.yaml
network:
  plmn:
    mcc: "001"
    mnc: "01"
  tac: 7
  ueSubnet: "10.60.0.0/16"

subscriber:
  imsi: "001010000000001"
  key: "your-authentication-key"
  opc: "your-operator-key"
```

Deploy:
```bash
helm install my-5g ./helm/open5gs-5g -f custom-values.yaml
```

### Disable UERANSIM

```yaml
# values-no-ueransim.yaml
ueransim:
  enabled: false
```

### Scale Network Functions

```yaml
# values-ha.yaml
open5gs:
  amf:
    replicas: 2
  smf:
    replicas: 2
  upf:
    replicas: 3
```

### Production Resources

```yaml
# values-production.yaml
open5gs:
  amf:
    resources:
      requests:
        memory: "512Mi"
        cpu: "500m"
      limits:
        memory: "1Gi"
        cpu: "1000m"
  smf:
    resources:
      requests:
        memory: "512Mi"
        cpu: "500m"
      limits:
        memory: "1Gi"
        cpu: "1000m"
  upf:
    resources:
      requests:
        memory: "1Gi"
        cpu: "1000m"
      limits:
        memory: "2Gi"
        cpu: "2000m"
```

## Upgrade

```bash
# Upgrade with new values
helm upgrade open5gs-5g ./helm/open5gs-5g -f new-values.yaml

# Upgrade and wait for readiness
helm upgrade open5gs-5g ./helm/open5gs-5g --wait --timeout 10m

# Rollback if needed
helm rollback open5gs-5g
```

## Uninstall

```bash
# Uninstall the release
helm uninstall open5gs-5g

# Uninstall and delete namespace
helm uninstall open5gs-5g
kubectl delete namespace open5gs
```

## Troubleshooting

### Check Helm Release Status

```bash
helm status open5gs-5g
helm get values open5gs-5g
helm get manifest open5gs-5g
```

### Pod Not Starting

```bash
# Check pod status
kubectl get pods -n open5gs

# Describe pod
kubectl describe pod -n open5gs <pod-name>

# Check logs
kubectl logs -n open5gs <pod-name>
```

### UE Registration Failed

```bash
# Check UE logs
kubectl logs -n open5gs deployment/ueransim-gnb -c ue --tail=100

# Check AMF logs
kubectl logs -n open5gs deployment/amf --tail=100

# Check SMF logs
kubectl logs -n open5gs deployment/smf --tail=100
```

### Network Function Discovery Issues

```bash
# Check NRF logs
kubectl logs -n open5gs deployment/nrf --tail=100

# Verify all NFs are registered
kubectl exec -n open5gs deployment/nrf -- curl -s http://localhost:7777/nnrf-nfm/v1/nf-instances
```

## Helm Testing

```bash
# Dry run to see generated manifests
helm install open5gs-5g ./helm/open5gs-5g --dry-run --debug

# Template without installing
helm template open5gs-5g ./helm/open5gs-5g > rendered.yaml

# Lint the chart
helm lint ./helm/open5gs-5g
```

## Chart Development

### Directory Structure

```
helm/open5gs-5g/
├── Chart.yaml              # Chart metadata
├── values.yaml             # Default values
├── README.md               # This file
├── templates/              # Kubernetes manifests templates
│   ├── _helpers.tpl        # Template helpers
│   ├── namespace.yaml      # Namespace
│   ├── NOTES.txt           # Post-install notes
│   ├── open5gs/            # Open5GS NF templates
│   └── ueransim/           # UERANSIM templates
└── charts/                 # Dependency charts (if any)
```

### Package the Chart

```bash
# Package for distribution
helm package ./helm/open5gs-5g

# Creates: open5gs-5g-1.0.0.tgz
```

### Publish to Repository

```bash
# Generate index
helm repo index .

# Upload to chart repository
# (your specific repository instructions)
```

## Access Points

After installation:

- **WebUI**: `http://localhost:30999`
  - Username: `admin`
  - Password: `1423`

- **AMF NGAP**: Port `38412` (SCTP)
- **UPF GTPU**: Port `2152` (UDP)
- **UPF PFCP**: Port `8805` (UDP)

## Verification

Expected success messages in UE logs:
```
[nas] [info] Initial Registration is successful
[nas] [info] PDU Session establishment is successful PSI[1]
[app] [info] Connection setup for PDU session[1] is successful
```

## License

This Helm chart follows the same license as the Open5GS project.

## References

- [Open5GS Documentation](https://open5gs.org)
- [UERANSIM GitHub](https://github.com/aligungr/UERANSIM)
- [Helm Documentation](https://helm.sh/docs/)

## Contributing

Contributions are welcome! Please submit pull requests or issues to the main repository.
