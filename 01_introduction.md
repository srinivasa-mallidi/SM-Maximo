# Module 01: IBM Maximo Application Suite (MAS) — Introduction

---

## 1.1 What is IBM Maximo Application Suite?

IBM Maximo Application Suite (MAS) is a **unified, cloud-native platform** that brings together a family of asset management, monitoring, predictive maintenance, and reliability applications under one license, one login, and one integrated data model.

It is the evolution of the classic IBM Maximo Enterprise Asset Management (EAM) product into a modern, container-based, AI-enriched suite.

### Key Value Proposition
- Single suite license (called **AppPoint** licensing)
- Unified user experience across all apps
- AI and IoT built-in (IBM Watson, IoT Platform)
- Cloud-native on Red Hat OpenShift (runs anywhere: on-prem, AWS, Azure, GCP, IBM Cloud)
- Single sign-on across all applications

---

## 1.2 History & Evolution

| Era | Product | Notes |
|-----|---------|-------|
| 1990s | PSDI Maximo | Original asset management |
| 2006 | IBM Maximo 6.x | IBM acquires PSDI |
| 2009 | IBM Maximo 7.x | Modern EAM era |
| 2014–2019 | Maximo 7.6.x | Last major traditional releases |
| 2021 | MAS 8.0 | First SaaS/OpenShift release |
| 2022 | MAS 8.6–8.8 | Maximo Manage GA, Mobile apps |
| 2023 | MAS 8.9–8.11 | Health, Predict, Visual Inspection maturity |
| 2024 | MAS 9.x | Next-gen architecture improvements |

---

## 1.3 MAS Applications (Full Suite)

MAS consists of the following applications, all available under a single license:

### Core Applications

| Application | Purpose |
|-------------|---------|
| **Maximo Manage** | Enterprise Asset Management (EAM) — the heart of the suite |
| **Maximo Health** | Asset health scoring, investment optimization |
| **Maximo Predict** | Predictive failure analysis using AI/ML |
| **Maximo Visual Inspection** | AI-powered image/video inspection |
| **Maximo Monitor** | IoT-based real-time asset monitoring |
| **Maximo Assist** | AI-powered technician assistance (remote expert) |
| **Maximo Optimizer** | Schedule and resource optimization |
| **Maximo Mobile** | Mobile apps for field technicians |
| **Maximo Safety** | Safety incident management |

### Supporting Components

| Component | Purpose |
|-----------|---------|
| **MAS Core** | Identity, auth, licensing, routing |
| **IoT Tool** | Device management and data ingestion |
| **AI Manager** | Watson AI model management |
| **CP4D (Cloud Pak for Data)** | Analytics and data science platform |

---

## 1.4 MAS Licensing — AppPoints

Traditional Maximo used **per-user** or **concurrent user** licensing. MAS uses **AppPoints**, which is a flexible consumption-based model.

### How AppPoints Work

- Each MAS application consumes a certain number of AppPoints per user.
- AppPoints are shared across the entire suite — unused capacity in one app can be used by another.
- There are two types: **Limited** and **Base** AppPoints.

### AppPoint Consumption Table (approximate)

| Application | Role | AppPoints Required |
|-------------|------|-------------------|
| Manage | Premium (full) | 150 |
| Manage | Base | 45 |
| Manage | Limited | 10 |
| Health | - | 30 |
| Predict | - | 30 |
| Monitor | - | 10 |
| Visual Inspection | - | 30 |
| Assist | - | 30 |
| Mobile | - | 10 |

### AppPoint vs Traditional Licensing Benefits

- Pay for what you use
- Add apps without new purchases (within purchased points)
- True multi-app access for field workers at low cost

---

## 1.5 MAS Deployment Models

MAS can be deployed in several ways:

### 1. IBM Cloud (SaaS / Managed)
- IBM manages everything (infrastructure, OpenShift, MAS)
- Fastest time-to-value
- Used by mid-market customers

### 2. Self-Managed on IBM Cloud
- Customer manages MAS, IBM manages OpenShift infra
- More control, still cloud-hosted

### 3. Self-Managed on AWS / Azure / GCP
- Customer runs OpenShift on hyperscalers
- Full flexibility, higher operational effort

### 4. On-Premises
- OpenShift on customer's own servers/VMs
- Air-gapped deployments supported
- Maximum data sovereignty

### 5. Dedicated (IBM SaaS)
- Single-tenant managed SaaS
- Best of both worlds

---

## 1.6 MAS Core Concepts

### Workspace
A logical grouping of configurations and users within a MAS instance. Each workspace is essentially a separate Maximo Manage instance.

### Suite License Manager (SLM)
The internal license enforcement engine. It tracks AppPoint consumption in real-time.

### MAS Hub / Admin UI
The central admin panel for:
- Managing workspaces
- Installing/upgrading apps
- Managing users and entitlements
- Viewing health of all components

### Identity Provider (IdP)
MAS supports:
- Built-in user registry
- LDAP integration
- SAML 2.0 (SSO)
- OpenID Connect (OIDC)

---

## 1.7 Key Benefits Over Traditional Maximo

| Feature | Classic Maximo 7.6.x | MAS 8.x/9.x |
|---------|---------------------|-------------|
| Deployment | WAR files on WebSphere | Containers on OpenShift |
| Upgrades | Complex, multi-day | Rolling, automated |
| Scaling | Manual, per-server | Auto-scaling pods |
| Mobile | Separate Anywhere server | Native mobile apps |
| AI/IoT | Bolt-on integrations | Built-in suite |
| Licensing | Per user | AppPoints (flexible) |
| UI | Classic Maximo UI | Refreshed + React-based |
| HA/DR | Complex setup | Native K8s mechanisms |

---

## 1.8 MAS Pre-Requisites Summary

Before installing MAS, you need:

1. **Red Hat OpenShift** 4.10+ cluster
2. **Storage** — at least one StorageClass with:
   - ReadWriteOnce (RWO) — for databases
   - ReadWriteMany (RWX) — for shared file storage
3. **IBM Entitlement Key** — from IBM MyIBM portal
4. **Internet Access** (or mirrored registry for air-gap)
5. **IBM Db2** (for Maximo Manage database)
6. **MongoDB** (for MAS Core)
7. **Cert Manager** — for TLS certificates
8. **Behavior Analytics Services (BAS)** — usage tracking

---

## 1.9 MAS Versions & Compatibility

| MAS Version | OpenShift Version | Notes |
|-------------|------------------|-------|
| 8.8 | 4.10, 4.11 | Stable GA |
| 8.9 | 4.11, 4.12 | Manage 8.6 |
| 8.10 | 4.12, 4.13 | Manage 8.7 |
| 8.11 | 4.13, 4.14 | Manage 8.8 |
| 9.0 | 4.14+ | Long-term support path |

> **Always check the IBM MAS System Requirements page before installing.**

---

## 1.10 Chapter Summary

- MAS is IBM's modern, cloud-native asset management suite
- It runs on Red Hat OpenShift using container technology
- It replaces traditional Maximo 7.6.x with a fully integrated AI/IoT platform
- Licensing is consumption-based (AppPoints)
- Multiple deployment models are available

**Next:** [02 — Red Hat OpenShift Fundamentals](../02_RedHat_OpenShift/01_openshift_basics.md)
