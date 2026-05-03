# Module 06: MAS Installation — Step-by-Step Guide

---

## 6.1 Pre-Installation Checklist

Before starting MAS installation, verify ALL items:

### Infrastructure Requirements

- [ ] Red Hat OpenShift 4.10+ cluster is running and healthy
- [ ] All cluster operators are Available (green)
- [ ] Minimum 3 worker nodes (each 16 vCPU / 64 GB RAM)
- [ ] RWO storage class available (for Db2, MongoDB)
- [ ] RWX storage class available (for Manage doclinks)
- [ ] Default storage class is set
- [ ] Cluster connected to internet OR air-gap mirror configured
- [ ] DNS resolution working (wildcard DNS `*.apps.cluster.domain`)
- [ ] TLS certificates available OR cert-manager can issue them

### IBM Account Requirements
- [ ] IBM ID with entitlement for MAS
- [ ] IBM Entitlement Key retrieved from https://myibm.ibm.com/products-services/containerlibrary
- [ ] MAS license file (AppPoint license) available

### Tools Required
- [ ] `oc` CLI installed and logged into cluster as cluster-admin
- [ ] `kubectl` CLI (optional, `oc` covers most needs)
- [ ] `ansible` (if using Ansible-based installer)
- [ ] `helm` (optional)
- [ ] `jq` (useful for scripting)

---

## 6.2 Installation Methods

### Method 1: IBM MAS CLI (Ansible-based) — Recommended
The **ibm-mas/cli** tool automates the full installation using Ansible playbooks.

### Method 2: Manual via YAML/Operators
Manual step-by-step application of Kubernetes resources.

### Method 3: IBM CloudPak Deployer
IBM's `cloudpak-deployer` tool for end-to-end automated deployments.

---

## 6.3 Method 1: MAS CLI Installation (Ansible)

### Step 1: Install MAS CLI

```bash
# On macOS
brew install ibm-mas/cli/mas

# On Linux
curl -sSL https://ibm.biz/MAS-CLI | bash

# Verify installation
mas --version
```

### Step 2: Set Environment Variables

```bash
export IBM_ENTITLEMENT_KEY="your_entitlement_key_here"
export MAS_INSTANCE_ID="inst1"
export MAS_CONFIG_DIR="/home/user/mas-config"
export STORAGE_CLASS_RWO="ibmc-block-gold"
export STORAGE_CLASS_RWX="ibmc-file-gold"

# For Db2
export DB2_INSTANCE_NAME="db2inst1"
export DB2_NAMESPACE="db2u"

# For MongoDB
export MONGODB_NAMESPACE="ibm-mas-mongo"
```

### Step 3: Run the MAS Setup Wizard

```bash
# Interactive setup (walks through all options)
mas setup-suite

# OR non-interactive (all options as flags)
mas install \
  --instance-id inst1 \
  --storage-class-rwo ibmc-block-gold \
  --storage-class-rwx ibmc-file-gold \
  --mas-channel 8.11.x \
  --accept-license
```

### Step 4: What the Installer Does

The MAS CLI installer runs Ansible playbooks that:

1. Creates `ibm-operator-catalog` CatalogSource
2. Installs cert-manager operator
3. Installs IBM Common Services
4. Installs MAS Core operator
5. Creates MAS Suite CR
6. Deploys MongoDB
7. Deploys Db2 (if installing Manage)
8. Installs Maximo Manage operator and app
9. Configures SLS (Suite License Service)
10. Waits for all components to be Ready

Total installation time: **1.5 – 3 hours** depending on cluster resources.

---

## 6.4 Manual Installation — Step by Step

### Step 1: Create IBM Operator Catalog

```yaml
# catalog-source.yaml
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: ibm-operator-catalog
  namespace: openshift-marketplace
spec:
  displayName: IBM Operator Catalog
  image: icr.io/cpopen/ibm-operator-catalog
  publisher: IBM
  sourceType: grpc
  updateStrategy:
    registryPoll:
      interval: 45m
```

```bash
oc apply -f catalog-source.yaml

# Wait for catalog to be ready
oc get catalogsource ibm-operator-catalog -n openshift-marketplace
```

### Step 2: Create Entitlement Key Secret

```bash
# Create namespace first
oc create namespace mas-inst1-core

# Create entitlement key secret
oc create secret docker-registry ibm-entitlement-key \
  --docker-server=cp.icr.io \
  --docker-username=cp \
  --docker-password="${IBM_ENTITLEMENT_KEY}" \
  --namespace=mas-inst1-core

# Also create in openshift-marketplace namespace
oc create secret docker-registry ibm-entitlement-key \
  --docker-server=cp.icr.io \
  --docker-username=cp \
  --docker-password="${IBM_ENTITLEMENT_KEY}" \
  --namespace=openshift-marketplace
```

### Step 3: Install cert-manager

```yaml
# cert-manager-subscription.yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: cert-manager-operator
  namespace: cert-manager-operator
spec:
  channel: stable-v1
  installPlanApproval: Automatic
  name: openshift-cert-manager-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
```

```bash
oc create namespace cert-manager-operator
oc apply -f cert-manager-subscription.yaml

# Wait for cert-manager to be ready
oc wait --for=condition=Available deployment/cert-manager \
  -n cert-manager --timeout=300s
```

### Step 4: Install IBM Common Services

```yaml
# common-services-subscription.yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ibm-common-service-operator
  namespace: ibm-common-services
spec:
  channel: v3
  installPlanApproval: Automatic
  name: ibm-common-service-operator
  source: ibm-operator-catalog
  sourceNamespace: openshift-marketplace
```

```bash
oc create namespace ibm-common-services
oc apply -f common-services-subscription.yaml
```

### Step 5: Install MAS Core Operator

```yaml
# mas-operator-group.yaml
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: ibm-mas-operatorgroup
  namespace: mas-inst1-core
spec:
  targetNamespaces:
    - mas-inst1-core
---
# mas-subscription.yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ibm-mas
  namespace: mas-inst1-core
spec:
  channel: 8.11.x
  installPlanApproval: Automatic
  name: ibm-mas
  source: ibm-operator-catalog
  sourceNamespace: openshift-marketplace
```

```bash
oc apply -f mas-operator-group.yaml
oc apply -f mas-subscription.yaml

# Wait for operator to be ready
watch oc get pods -n mas-inst1-core
```

### Step 6: Deploy MongoDB

```yaml
# mongodb-subscription.yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ibm-mongodb-operator
  namespace: ibm-mas-mongo
spec:
  channel: stable
  installPlanApproval: Automatic
  name: ibm-mongodb-operator
  source: ibm-operator-catalog
  sourceNamespace: openshift-marketplace
---
# mongodb-cr.yaml
apiVersion: mongodbcommunity.mongodb.com/v1
kind: MongoDBCommunity
metadata:
  name: mongo-ce
  namespace: ibm-mas-mongo
spec:
  members: 3
  type: ReplicaSet
  version: "5.0.0"
  security:
    tls:
      enabled: true
  users:
  - name: admin
    db: admin
    passwordSecretRef:
      name: mongo-admin-password
    roles:
    - db: admin
      name: clusterAdmin
    - db: admin
      name: userAdminAnyDatabase
    scramCredentialsSecretName: my-scram
  statefulSet:
    spec:
      volumeClaimTemplates:
      - metadata:
          name: data-volume
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 20Gi
          storageClassName: ibmc-block-gold
```

```bash
oc create namespace ibm-mas-mongo
oc apply -f mongodb-subscription.yaml
oc apply -f mongodb-cr.yaml

# Wait for MongoDB pods
watch oc get pods -n ibm-mas-mongo
```

### Step 7: Create MAS Suite Custom Resource

```yaml
# mas-suite.yaml
apiVersion: core.mas.ibm.com/v1
kind: Suite
metadata:
  name: inst1
  namespace: mas-inst1-core
spec:
  displayName: "Production MAS"
  license:
    accept: true
  settings:
    icr:
      cp: cp.icr.io/cp
      cpopen: icr.io/cpopen
  template:
    pod:
      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            preference:
              matchExpressions:
              - key: node-role.kubernetes.io/worker
                operator: Exists
```

```bash
oc apply -f mas-suite.yaml

# Watch the Suite come up (can take 20-40 mins)
watch oc get Suite -n mas-inst1-core
oc describe Suite inst1 -n mas-inst1-core
```

### Step 8: Install Db2 for Manage

```bash
# Install Db2 Universal Operator
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: db2u-operator
  namespace: db2u
spec:
  channel: v110508.0
  installPlanApproval: Automatic
  name: db2u-operator
  source: ibm-operator-catalog
  sourceNamespace: openshift-marketplace
EOF
```

```yaml
# db2-instance.yaml
apiVersion: db2u.databases.ibm.com/v1
kind: Db2uCluster
metadata:
  name: db2ucluster-manage
  namespace: db2u
spec:
  account:
    privileged: true
  license:
    accept: true
  size: 1
  storage:
  - name: meta
    spec:
      accessModes: [ReadWriteMany]
      resources:
        requests:
          storage: 20Gi
      storageClassName: ibmc-file-gold
    type: create
  - name: data
    spec:
      accessModes: [ReadWriteOnce]
      resources:
        requests:
          storage: 100Gi
      storageClassName: ibmc-block-gold
    type: create
  - name: backup
    spec:
      accessModes: [ReadWriteMany]
      resources:
        requests:
          storage: 50Gi
      storageClassName: ibmc-file-gold
    type: create
```

### Step 9: Install Maximo Manage Operator

```bash
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ibm-mas-manage
  namespace: mas-inst1-manage
spec:
  channel: 8.7.x
  installPlanApproval: Automatic
  name: ibm-mas-manage
  source: ibm-operator-catalog
  sourceNamespace: openshift-marketplace
EOF
```

### Step 10: Create ManageApp Custom Resource

```yaml
# manage-app.yaml
apiVersion: apps.mas.ibm.com/v1
kind: ManageApp
metadata:
  name: inst1
  namespace: mas-inst1-manage
spec:
  bindings:
    jdbc:
      manage:
        db2uCluster:
          name: db2ucluster-manage
          namespace: db2u
  settings:
    deployment:
      serverBundles:
      - name: allbundle
        isMassage: false
        isDefault: true
        replica: 2
    persistentStorage:
      accessModes: [ReadWriteMany]
      storageClassName: ibmc-file-gold
      volumeSize: 100Gi
```

```bash
oc create namespace mas-inst1-manage
oc apply -f manage-app.yaml
```

---

## 6.5 Post-Installation Verification

```bash
# Check MAS Suite status
oc get Suite inst1 -n mas-inst1-core -o jsonpath='{.status.conditions}' | jq .

# Check all pods are Running
oc get pods -n mas-inst1-core | grep -v Running
oc get pods -n mas-inst1-manage | grep -v Running

# Get MAS Hub URL
oc get route -n mas-inst1-core | grep home
# Output: home  home.inst1.apps.mycluster.com ...

# Get Manage URL
oc get route -n mas-inst1-manage | grep manage
```

### Accessing MAS Hub

1. Open browser: `https://home.inst1.apps.{cluster-domain}`
2. Log in with admin credentials
3. Navigate to **Applications → Manage**
4. Verify Manage workspace is "Ready"

---

## 6.6 Uploading the License File

```bash
# Via MAS CLI
mas update-license --instance-id inst1 --license-file /path/to/license.dat

# OR manually - create SLS config
oc create configmap sls-license --from-file=license.dat=/path/to/license.dat \
  -n mas-inst1-core
```

---

## 6.7 Upgrade Procedure

### Upgrading MAS Core

```bash
# Check current channel
oc get subscription ibm-mas -n mas-inst1-core -o jsonpath='{.spec.channel}'

# To upgrade channel (e.g., 8.10.x → 8.11.x)
oc patch subscription ibm-mas -n mas-inst1-core \
  --type=merge -p '{"spec":{"channel":"8.11.x"}}'

# Monitor upgrade
watch oc get Suite inst1 -n mas-inst1-core
watch oc get pods -n mas-inst1-core
```

### Upgrading Manage

```bash
oc patch subscription ibm-mas-manage -n mas-inst1-manage \
  --type=merge -p '{"spec":{"channel":"8.8.x"}}'

# Watch for updatedb job to complete
oc get jobs -n mas-inst1-manage
watch oc get pods -n mas-inst1-manage
```

---

## Summary

- MAS installation requires OpenShift + storage + MongoDB + Db2
- MAS CLI (Ansible) automates the full process in ~2 hours
- Manual installation follows: Catalog → cert-manager → Common Services → MAS Core → MongoDB → Db2 → Manage
- Post-install: verify routes, upload license, test login
- Upgrades are channel-based (change subscription channel)

**Next:** [MAS Administration Guide](../07_Administration/01_admin_guide.md)
