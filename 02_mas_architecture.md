# Module 01: MAS Architecture Deep Dive

---

## 1.11 MAS Architecture Overview

MAS is built on a **microservices architecture** running on Kubernetes (via Red Hat OpenShift). Every component runs as one or more Pods within OpenShift namespaces.

```
┌─────────────────────────────────────────────────────────────┐
│                    User Browsers / Mobile Apps               │
└─────────────────────┬───────────────────────────────────────┘
                      │ HTTPS
┌─────────────────────▼───────────────────────────────────────┐
│              OpenShift Router (HAProxy Ingress)              │
└──────┬───────────────┬──────────────────┬───────────────────┘
       │               │                  │
┌──────▼──────┐ ┌──────▼──────┐  ┌───────▼──────┐
│  MAS Core   │ │   Manage    │  │  Monitor/    │
│  (Auth/Hub) │ │   (EAM)     │  │  Predict etc │
└──────┬──────┘ └──────┬──────┘  └───────┬──────┘
       │               │                  │
┌──────▼───────────────▼──────────────────▼──────┐
│            Shared Platform Services              │
│  MongoDB | Db2 | Kafka | CP4D | Cert-Manager    │
└─────────────────────────────────────────────────┘
       │
┌──────▼──────────────────────────────────────────┐
│            Red Hat OpenShift Cluster             │
│   Worker Nodes  |  Master Nodes  |  Infra Nodes │
└─────────────────────────────────────────────────┘
```

---

## 1.12 MAS Namespaces (OpenShift Projects)

When MAS is installed, it creates several OpenShift **namespaces (projects)**:

| Namespace | Contents |
|-----------|---------|
| `mas-{instance}-core` | MAS Core services, Hub UI |
| `mas-{instance}-manage` | Maximo Manage application pods |
| `mas-{instance}-monitor` | Maximo Monitor |
| `mas-{instance}-predict` | Maximo Predict |
| `ibm-common-services` | IBM Cloud Pak Common Services |
| `cert-manager` | Certificate management |
| `ibm-mas-mongo` | MongoDB operator + instances |
| `db2u` | Db2 Universal operator |
| `ibm-operator-catalog` | IBM operator catalog |

---

## 1.13 MAS Core Components In Detail

### MAS Core Pod Groups

| Pod Prefix | Function |
|------------|---------|
| `coreapi` | REST APIs for MAS management |
| `coreidp` | Identity Provider (authentication) |
| `frontend` | Admin Hub web UI |
| `usersync` | User synchronization service |
| `licensing` | AppPoint license management |
| `mongomigrate` | Database migration jobs |

### MongoDB Usage in MAS Core
MongoDB stores:
- User registrations and entitlements
- Suite configuration
- Application workspace settings
- Usage/telemetry data

MongoDB is NOT used for Manage's operational data (that's Db2).

---

## 1.14 Maximo Manage Architecture

Manage is the most complex application in MAS — it is the evolution of Maximo 7.6.x.

### Manage Pod Groups

| Pod Group | Function |
|-----------|---------|
| `{workspace}-all` | "All" server bundle (default, handles everything) |
| `{workspace}-ui` | UI-only pods (dedicated web tier) |
| `{workspace}-mea` | Maximo Enterprise Adapter (integration) |
| `{workspace}-report` | BIRT Report server |
| `{workspace}-cron` | Cron task runner |
| `{workspace}-standby` | Hot standby pods |

### Manage Database
- Uses **IBM Db2** (traditional RDBMS)
- Schema is the classic Maximo database schema (hundreds of tables)
- Deployed via the Db2 Universal Operator OR external Db2

### Manage File Storage (Doclinks)
- Attachments (Doclinks) stored on ReadWriteMany (RWX) PVC
- Requires shared filesystem (NFS, ODF, EFS, etc.)

### Manage Upgrade Process
- "Updatedb" job runs database migrations on upgrade
- Schema changes are tracked internally
- Zero-downtime upgrades via rolling pod restart

---

## 1.15 Data Flow in MAS

```
User Browser
    │
    ▼
OpenShift Router ──► MAS Core (Auth) ──► JWT Token issued
    │
    ▼
Maximo Manage UI Pod
    │
    ├──► Db2 Database (operational data: assets, work orders)
    ├──► MongoDB (session/config data)
    ├──► File Storage (doclinks/attachments)
    └──► Integration Framework (MIF) ──► External systems
```

---

## 1.16 High Availability (HA) in MAS

MAS achieves HA through Kubernetes/OpenShift native mechanisms:

### Pod-Level HA
- Multiple replicas for each critical service
- `PodDisruptionBudget` (PDB) ensures minimum available pods
- `ReadinessProbe` and `LivenessProbe` for auto-recovery

### Database HA
- Db2 HADR (High Availability Disaster Recovery) for synchronous replication
- MongoDB ReplicaSet with 3 members (primary + 2 secondaries)

### Storage HA
- ODF (OpenShift Data Foundation) for replicated block/file storage
- IBM Cloud: IBM Cloud File Storage (Gold tier for RWX)

### Cluster-Level HA
- OpenShift masters in 3-node cluster (quorum-based)
- Worker nodes spread across availability zones

---

## 1.17 Disaster Recovery (DR)

| Strategy | RTO | RPO |
|----------|-----|-----|
| Active-Passive (backup/restore) | Hours | Hours |
| Db2 HADR across sites | Minutes | Seconds |
| Full cluster mirroring | Minutes | Near-zero |

### Backup Components
1. Db2 database (full + incremental backups)
2. MongoDB (mongodump or Velero)
3. PVC contents (doclinks)
4. OpenShift etcd backup
5. MAS configuration (Kubernetes secrets/configmaps)

Tools: **Velero** for K8s-level backup, **Db2 BACKUP** for database.

---

## 1.18 Security Architecture

### Authentication Flow
```
User → MAS Login Page → IDP (LDAP/SAML/OIDC) → JWT Token → App Access
```

### Authorization Levels
1. **Suite Level** — Can the user log in to MAS?
2. **Application Level** — Can the user access Manage vs Monitor vs Health?
3. **Workspace Level** — Which workspace can they access?
4. **Manage Security** — Groups, Sites, Applications in classic Maximo security

### TLS/Certificate Management
- cert-manager handles all TLS certificates
- Supports Let's Encrypt, custom CA, self-signed
- Certificates stored as OpenShift Secrets
- Auto-renewal supported

### Network Policies
- OpenShift NetworkPolicy restricts pod-to-pod communication
- Only required service ports are exposed
- Egress policies for external integration control

---

## 1.19 MAS Operators

MAS is fully managed by **Kubernetes Operators**. Operators are controllers that manage the lifecycle of complex applications.

| Operator | Manages |
|----------|---------|
| `ibm-mas-operator` | MAS Suite CR |
| `ibm-mas-manage-operator` | Manage app CR |
| `ibm-mas-monitor-operator` | Monitor app CR |
| `ibm-db2u-operator` | Db2 instances |
| `ibm-mongodb-operator` | MongoDB instances |
| `cert-manager-operator` | TLS certificates |
| `ibm-common-services-operator` | Shared platform services |

### Custom Resources (CRs)
Each component has a Custom Resource that defines its desired state:

```yaml
# Example: MAS Suite CR
apiVersion: core.mas.ibm.com/v1
kind: Suite
metadata:
  name: inst1
  namespace: mas-inst1-core
spec:
  displayName: "My MAS Instance"
  license:
    accept: true
  settings:
    icr:
      cp: cp.icr.io/cp
```

---

## Summary

- MAS is a microservices architecture on OpenShift
- Multiple namespaces isolate different components
- MongoDB for configuration, Db2 for operational data
- Kubernetes Operators manage all lifecycle events
- HA is native through K8s replication mechanisms

**Next:** [MAS Components Catalog](./03_mas_components.md)
