# Module 02: Red Hat OpenShift — Fundamentals for MAS

---

## 2.1 What is Red Hat OpenShift?

Red Hat OpenShift Container Platform (OCP) is an **enterprise Kubernetes distribution** built and supported by Red Hat (an IBM subsidiary). It adds security, developer tools, operations tooling, and a rich GUI on top of upstream Kubernetes.

### OpenShift vs Kubernetes

| Feature | Kubernetes | OpenShift |
|---------|-----------|-----------|
| Core | Kubernetes (self) | Kubernetes (enterprise hardened) |
| GUI | Basic Dashboard | Full Web Console |
| Security | Manual RBAC | SCC (stricter default policies) |
| CI/CD | External tools | Built-in Tekton pipelines |
| Registry | External | Built-in internal registry |
| Routes | Ingress (manual) | Routes (automatic TLS) |
| Operators | OLM optional | OLM built-in |
| Updates | Manual | Over-the-air (OTA) |
| Support | Community | Red Hat enterprise support |

---

## 2.2 OpenShift Core Concepts

### Nodes
Physical or virtual machines in the cluster:

| Node Type | Purpose |
|-----------|---------|
| **Master (Control Plane)** | etcd, API server, scheduler, controller manager |
| **Worker (Compute)** | Runs application workloads (Pods) |
| **Infrastructure (Infra)** | Runs platform services: ingress, monitoring, registry |

### Minimum Cluster for MAS

| Node Type | Count | vCPU | RAM | Disk |
|-----------|-------|------|-----|------|
| Master | 3 | 4 | 16 GB | 100 GB |
| Worker | 3+ | 16 | 64 GB | 200 GB |
| Infra | 3 | 8 | 32 GB | 200 GB |

> **For production MAS with Manage:** Workers should be 32 vCPU / 128 GB RAM.

---

## 2.3 Kubernetes Objects Used by MAS

### Pod
The smallest deployable unit. A Pod contains one or more containers.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: maximo-manage-pod
  namespace: mas-inst1-manage
spec:
  containers:
  - name: manage
    image: cp.icr.io/cp/manage:8.7.0
    ports:
    - containerPort: 9080
```

### Deployment / DeploymentConfig
Manages multiple Pod replicas (auto-restart, rolling update).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mas-manage-ui
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: ui
        image: cp.icr.io/cp/manage-ui:latest
```

### Service
A stable network endpoint for a group of Pods:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: manage-svc
spec:
  selector:
    app: manage
  ports:
  - port: 443
    targetPort: 9443
  type: ClusterIP
```

### Route (OpenShift-specific)
Exposes a Service externally via a hostname:

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: manage-route
spec:
  host: manage.apps.mycluster.example.com
  to:
    kind: Service
    name: manage-svc
  tls:
    termination: edge
```

### Namespace / Project
Logical isolation boundary. OpenShift calls them "Projects" (same as Namespace).

### ConfigMap
Stores non-sensitive configuration as key-value pairs:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: manage-config
data:
  JAVA_OPTS: "-Xms2g -Xmx4g"
  DB_URL: "jdbc:db2://db2-host:50000/MAXDB"
```

### Secret
Stores sensitive data (base64 encoded):

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: bWF4aW1v  # base64 of "maximo"
  password: bWF4cGFzcw==  # base64 of "maxpass"
```

### PersistentVolumeClaim (PVC)
Requests storage for pods:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: manage-doclinks
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Gi
  storageClassName: ocs-storagecluster-cephfs
```

### StatefulSet
Like Deployment but for stateful apps (databases). Pods get stable hostnames.

### Operator / CustomResource
Extends Kubernetes to manage complex applications. MAS uses many operators.

---

## 2.4 OpenShift CLI (oc) — Essential Commands

The `oc` CLI is the OpenShift equivalent of `kubectl` (plus extra features).

### Login and Context

```bash
# Login to cluster
oc login https://api.mycluster.example.com:6443 -u admin -p password

# Login with token
oc login --token=sha256~abcdef... --server=https://api.mycluster.example.com:6443

# Check current context
oc whoami
oc config current-context

# Switch project/namespace
oc project mas-inst1-manage

# List all projects
oc get projects
```

### Viewing Resources

```bash
# List pods
oc get pods -n mas-inst1-manage

# Describe a pod (detailed status)
oc describe pod manage-all-7d4f9b-xkzpq -n mas-inst1-manage

# Get pod logs
oc logs manage-all-7d4f9b-xkzpq -n mas-inst1-manage
oc logs -f manage-all-7d4f9b-xkzpq -n mas-inst1-manage  # follow

# Get all resources in namespace
oc get all -n mas-inst1-manage

# Get nodes
oc get nodes
oc describe node worker-1

# Get events (useful for troubleshooting)
oc get events -n mas-inst1-manage --sort-by='.lastTimestamp'
```

### Managing Pods

```bash
# Execute command in a running pod
oc exec -it manage-all-7d4f9b-xkzpq -n mas-inst1-manage -- /bin/bash

# Copy files to/from pod
oc cp /local/file.txt manage-pod:/remote/path/
oc cp manage-pod:/remote/file.txt /local/path/

# Delete a pod (it will restart if managed by Deployment)
oc delete pod manage-all-7d4f9b-xkzpq -n mas-inst1-manage

# Scale a deployment
oc scale deployment manage-all --replicas=2 -n mas-inst1-manage
```

### Storage Commands

```bash
# List PVCs
oc get pvc -n mas-inst1-manage

# List PVs (cluster-wide)
oc get pv

# Describe PVC
oc describe pvc manage-doclinks -n mas-inst1-manage
```

### Operator Commands

```bash
# List installed operators
oc get operators -A

# List CustomResourceDefinitions
oc get crd | grep mas

# Get MAS Suite CR status
oc get Suite -n mas-inst1-core
oc describe Suite inst1 -n mas-inst1-core

# Get Manage app CR
oc get ManageApp -n mas-inst1-manage
```

### YAML Apply

```bash
# Apply a YAML file
oc apply -f my-resource.yaml

# Delete from YAML file
oc delete -f my-resource.yaml

# Edit a resource in place
oc edit deployment manage-all -n mas-inst1-manage

# Get resource as YAML
oc get deployment manage-all -n mas-inst1-manage -o yaml
```

---

## 2.5 OpenShift Web Console

The Web Console is available at: `https://console-openshift-console.apps.{cluster-domain}`

### Key Sections

| Section | What You Do There |
|---------|------------------|
| **Workloads → Pods** | View, delete pods, see logs |
| **Workloads → Deployments** | Scale, restart deployments |
| **Storage → PVCs** | View storage claims |
| **Networking → Routes** | View application URLs |
| **Operators → Installed Operators** | Manage operators |
| **Administration → Projects** | Create/manage namespaces |
| **Monitoring** | View cluster metrics and alerts |
| **Compute → Nodes** | Node health and capacity |

---

## 2.6 Storage Classes (Critical for MAS)

MAS needs two types of storage:

### ReadWriteOnce (RWO) Storage
- Block storage, mounted by ONE pod at a time
- Used for: Db2 data files, MongoDB data
- Examples: `ibmc-block-gold`, `gp3-csi` (AWS), `managed-premium` (Azure)

### ReadWriteMany (RWX) Storage
- File storage, can be mounted by MANY pods simultaneously
- Used for: Maximo Manage doclinks (attachments)
- Examples:
  - `ocs-storagecluster-cephfs` (ODF)
  - `ibmc-file-gold` (IBM Cloud)
  - `aws-efs` (AWS EFS)
  - NFS-based storage class

### Setting Default Storage Classes

```bash
# Mark a storage class as default
oc patch storageclass ibmc-block-gold \
  -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

---

## 2.7 OpenShift Image Registry

OpenShift has a built-in container image registry. MAS pulls images from **IBM Container Registry (ICR)**.

### IBM Container Registry Pull Secret

```bash
# Create the IBM entitlement key secret (REQUIRED for MAS)
oc create secret docker-registry ibm-entitlement-key \
  --docker-server=cp.icr.io \
  --docker-username=cp \
  --docker-password=<YOUR_ENTITLEMENT_KEY> \
  -n mas-inst1-core
```

### Check Image Pull Status

```bash
# If pods are in ImagePullBackOff state
oc describe pod <pod-name> | grep -A5 "Events:"
```

---

## 2.8 OpenShift Monitoring & Alerting

OpenShift includes Prometheus + Grafana built-in.

### Viewing MAS Metrics

```bash
# Port-forward to Prometheus
oc port-forward svc/prometheus-operated 9090 -n openshift-monitoring

# Port-forward to Grafana
oc port-forward svc/grafana 3000 -n openshift-monitoring
```

### Key Metrics to Watch for MAS

| Metric | Why It Matters |
|--------|---------------|
| `node_memory_MemAvailable_bytes` | Worker node memory pressure |
| `kube_pod_status_phase` | Pod health status |
| `kube_persistentvolumeclaim_status_phase` | Storage health |
| `container_cpu_usage_seconds_total` | CPU consumption per container |

---

## 2.9 Security Context Constraints (SCC)

OpenShift uses SCCs to restrict what Pods can do — more restrictive than vanilla Kubernetes.

### Common SCCs

| SCC | Level of Privilege |
|-----|--------------------|
| `restricted-v2` | Default — most restrictive |
| `anyuid` | Pod can run as any UID |
| `privileged` | Full root access |
| `nonroot` | Must run as non-root UID |

### MAS and SCCs
MAS pods run under `restricted` or `anyuid` depending on the component. The operators handle SCC assignments automatically via ServiceAccounts.

```bash
# Check SCCs assigned to service account
oc describe scc anyuid | grep "Users:"
oc get rolebinding -n mas-inst1-manage -o wide
```

---

## 2.10 Helpful OpenShift Tips for MAS Admins

```bash
# Watch pods come up in real time
watch oc get pods -n mas-inst1-manage

# Get all failing pods across all namespaces
oc get pods -A | grep -v Running | grep -v Completed

# Check operator logs
oc logs -n ibm-mas-operator $(oc get pods -n ibm-mas-operator -o name | head -1)

# Force restart all pods in a deployment
oc rollout restart deployment/manage-all -n mas-inst1-manage

# Check resource usage by node
oc adm top nodes

# Check resource usage by pod
oc adm top pods -n mas-inst1-manage

# Get cluster version
oc get clusterversion

# Get cluster operators status (critical for upgrades)
oc get co
```

---

## Summary

- OpenShift is enterprise Kubernetes — it's what MAS runs on
- Key objects: Pod, Deployment, Service, Route, PVC, Secret, ConfigMap, Operator
- `oc` CLI is the primary tool for cluster administration
- Storage classes (RWO for databases, RWX for shared files) are critical
- SCCs control Pod security permissions
- IBM Entitlement Key is required to pull MAS container images

**Next:** [Maximo Manage — Complete Guide](../03_Maximo_Manage/01_manage_introduction.md)
