# Module 02: OpenShift Networking for MAS

---

## 2.11 OpenShift Networking Stack

OpenShift uses **OVN-Kubernetes** (or older OVS-based SDN) for pod networking.

### Network Layers

```
External Users
      │
      ▼
Load Balancer (Cloud LB / MetalLB / F5)
      │
      ▼
OpenShift Router (HAProxy) ← Routes terminate TLS here
      │
      ▼
Services (ClusterIP) ← Stable virtual IPs for Pod groups
      │
      ▼
Pods ← Container network via OVN
```

---

## 2.12 DNS in OpenShift

Every Service gets a DNS name:

```
<service-name>.<namespace>.svc.cluster.local
```

Examples for MAS:
```
# Access Db2 from Manage pods
db2u-db2u-0.db2u.svc.cluster.local:50000

# Access MongoDB from MAS Core
mongo-ce-0.mongo-ce-svc.ibm-mas-mongo.svc.cluster.local:27017

# Access Manage from Monitor
manage-svc.mas-inst1-manage.svc.cluster.local:443
```

---

## 2.13 Ingress / Routes

MAS uses OpenShift Routes for external access.

### Auto-generated Route Pattern
```
https://<app>.<instance>.<cluster-apps-domain>

# Examples:
https://manage.inst1.apps.mycluster.example.com   (Manage UI)
https://home.inst1.apps.mycluster.example.com     (MAS Hub)
https://api.inst1.apps.mycluster.example.com      (MAS API)
```

### Wildcard Certificate
For MAS, a **wildcard TLS certificate** is ideal:
```
*.apps.mycluster.example.com
```

Or cert-manager can issue per-route certificates automatically.

---

## 2.14 Network Policies

MAS creates NetworkPolicies to restrict traffic. Key rules:

1. Only MAS Core can communicate with Manage pods on port 443
2. Manage pods can communicate with Db2 on port 50000
3. Monitor can push data to Manage via API
4. No direct external access to databases

### Viewing Network Policies

```bash
oc get networkpolicy -n mas-inst1-manage
oc describe networkpolicy manage-allow-from-core -n mas-inst1-manage
```

---

## 2.15 Proxy Configuration

If your cluster is behind an HTTP proxy, configure it in OpenShift:

```yaml
apiVersion: config.openshift.io/v1
kind: Proxy
metadata:
  name: cluster
spec:
  httpProxy: http://proxy.company.com:3128
  httpsProxy: https://proxy.company.com:3128
  noProxy: ".cluster.local,.svc,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16"
```

MAS operators inherit this proxy config automatically.

---

## Summary

Networking is critical for MAS:
- Routes expose MAS externally
- Services provide stable internal endpoints
- NetworkPolicies secure component communication
- DNS resolution connects all components internally
