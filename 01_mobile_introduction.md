# Module 04: Maximo Mobile — Complete Guide

---

## 4.1 Introduction to Maximo Mobile

Maximo Mobile is IBM's **native mobile application** for field technicians and inspectors. It is a first-class MAS application that provides access to core Maximo Manage functions on smartphones and tablets.

### Key Characteristics
- Native apps for iOS and Android
- **Offline capability** — works without network connectivity
- React Native-based (cross-platform from a single codebase)
- Integrated with MAS authentication (SSO)
- Configurable via Maximo Application Framework (App Designer)
- Available from App Store and Google Play

---

## 4.2 Maximo Mobile Apps

MAS includes multiple specialized mobile apps:

| App | Purpose |
|-----|---------|
| **Maximo Mobile (Technician)** | Work order management for field techs |
| **Maximo Inspection** | Inspection forms and results |
| **Maximo Assist** | AI-powered guided troubleshooting |
| **Maximo Asset Monitor** | IoT data viewing |
| **Maximo Health** | Asset health scoring |

This module focuses primarily on **Maximo Mobile (Technician)**.

---

## 4.3 Maximo Mobile — Feature Overview

### Core Features

| Feature | Description |
|---------|-------------|
| **Work Order List** | View assigned and unassigned WOs |
| **Work Order Details** | Full WO information including asset, location, tasks |
| **Asset Lookup** | Search and view asset details |
| **Labor Reporting** | Log time directly from mobile |
| **Material Usage** | Issue/return materials from storeroom |
| **Failure Reporting** | Record failure codes and observations |
| **Attachments** | Take photos, attach files |
| **Signature Capture** | Digital sign-off on completed work |
| **Barcode/QR Scan** | Scan asset tags to quickly find records |
| **Maps** | Asset location on map |
| **Push Notifications** | Alerts for new WO assignments |

### Offline Capabilities
The app operates fully offline when network is unavailable:
- Data is downloaded to device during sync
- All actions are queued locally
- On reconnect, changes are synchronized to Manage
- Conflict detection for concurrent edits

---

## 4.4 Mobile App Architecture

```
Mobile Device (iOS / Android)
    │
    │  HTTPS (when online)
    ▼
Maximo Mobile API Gateway
    │
    ├──► Maximo Manage (work orders, assets, labor)
    ├──► Object Storage (attachments)
    └──► MAS Core (authentication / JWT)

Offline Mode:
    Mobile Device ─── Local SQLite DB (WOs, Assets, etc.)
                   ─── Sync Queue (pending actions)
```

### Technical Stack
- **Frontend:** React Native
- **Offline DB:** SQLite (on device)
- **Sync Protocol:** IBM Sync Service
- **Auth:** OAuth 2.0 / JWT from MAS Core
- **Push Notifications:** APNs (iOS) / FCM (Android)

---

## 4.5 Mobile Configuration in MAS

### Prerequisites
1. Maximo Manage deployed and operational
2. MAS Mobile app operator installed
3. App registration configured in MAS Core

### Enabling Mobile in MAS

1. Log in to **MAS Hub** (https://home.{instance}.apps.{cluster})
2. Navigate to **Applications → Mobile**
3. Install the Mobile application
4. Configure workspace binding to Manage

### Configuring Mobile Users

In Maximo Manage:
1. **Administration → Users** — ensure user has Person and Labor records
2. Assign user to **Security Group** with mobile permissions
3. Enable mobile access flag on user

---

## 4.6 Maximo Mobile — Work Order Workflow

### Technician Daily Workflow

```
1. Open App → Authenticate (SSO)
2. Sync latest assignments
3. View "My Work Orders" list
4. Open Work Order
    ├── View asset details and location
    ├── View job plan tasks
    ├── View attachments/technical docs
    ├── Navigate to asset (Maps)
5. Start work → Log start time
6. Complete tasks (checkboxes)
7. Issue materials from storeroom
8. Record failure codes
9. Take photos (auto-attached to WO)
10. Log actual labor hours
11. Capture supervisor signature
12. Complete / Close work order
13. Sync to Maximo Manage
```

### Work Order Status Changes from Mobile

| Action | Status Change |
|--------|--------------|
| Accept WO | WAPPR → INPRG |
| Start Work | Status = INPRG |
| Complete Work | INPRG → COMP |
| Fail (no parts) | INPRG → WMAT |

---

## 4.7 Maximo Inspection

Maximo Inspection is a specialized app for conducting structured inspections.

### Inspection Components

**Inspection Form** — Template defining what to check:
- Sections and questions
- Response types: Text, Number, Yes/No, Multiple choice, Date, Image
- Required vs optional questions
- Trigger conditions (if answer = X, show question Y)

**Inspection Result** — Completed instance of a form:
- Actual responses
- Photos attached per question
- Scoring (if configured)
- Status: Draft, Submitted, Reviewed, Approved

### Creating Inspection Forms

1. Open **Maximo Manage → Inspections → Inspection Forms**
2. Create new form
3. Add sections and questions
4. Set response types
5. Configure scoring (optional)
6. Activate the form

### Triggering Inspections
Inspections can be triggered from:
- Work Order tasks (link form to job plan task)
- Standalone PM records
- Asset condition events (via Monitor)
- Manual creation by supervisor

---

## 4.8 Offline Sync Configuration

### Data Downloaded to Device

When a technician syncs, the app downloads:
- Assigned Work Orders (configurable number, default 50)
- Referenced Assets and Locations
- Referenced Job Plans and Tasks
- Item catalog (for material selection)
- Active Inspection Forms
- Failure Codes and Domains

### Configuring Sync Settings

In Maximo Manage **System Properties**:

| Property | Description |
|----------|-------------|
| `maximo.mobile.sync.workorders` | Max WOs to sync per device |
| `maximo.mobile.offline.days` | Days ahead to sync PM work orders |
| `maximo.mobile.attachment.size` | Max attachment size (MB) |

### Sync Conflict Resolution

When the same record is modified both on mobile and on the server:
1. **Last Write Wins** (default) — later timestamp wins
2. **Server Wins** — server change always wins
3. **Client Wins** — mobile change always wins

---

## 4.9 Mobile App Customization

### Via App Designer (No-code)

IBM provides a visual App Designer for customizing mobile apps:

1. In Maximo Manage: **System Configuration → Platform Configuration → Application Designer**
2. Select the mobile application (e.g., `MXMOBILE`)
3. Add/remove/rearrange fields and sections
4. Apply changes (may require app sync on devices)

### Custom Sections
You can add sections to work order screens:
- Custom attributes from extended tables
- Integration data from other systems

### Automation Scripts in Mobile Context
Automation Scripts triggered by mobile events:
- `mbo.save` when WO updated from mobile
- `mbo.action` when mobile action fires

---

## 4.10 Maximo Assist (AI-Powered)

**Maximo Assist** brings AI capabilities to the mobile experience:

### Features
- **Guided troubleshooting** — AI walks through diagnosis steps
- **Remote expert video call** — Technician connects to remote expert
- **Document search** — AI searches technical manuals
- **Knowledge articles** — IBM Watson-powered Q&A

### How Assist Works
1. Technician encounters unfamiliar problem
2. Opens Assist app on mobile
3. Describes problem (text or voice)
4. Watson AI searches knowledge base
5. Returns ranked solutions
6. Optionally connects to live expert via video

### Assist Architecture
- Requires **IBM Watson Discovery** for document search
- Requires **IBM Watson Assistant** for conversational AI
- Video calling via **LiveLook** or similar integration

---

## 4.11 Maximo Anywhere (Legacy — Pre-MAS Mobile)

Before the native Maximo Mobile app, IBM had **Maximo Anywhere**:

| Feature | Anywhere | New Maximo Mobile |
|---------|----------|------------------|
| Technology | Dojo/web hybrid | React Native (native) |
| Offline | Limited | Full SQLite offline |
| Performance | Slower | Native performance |
| Customization | App Designer | App Designer + more |
| Platform | Web + wrapped app | True native |
| Status | End-of-life (MAS 8.x) | Current / active |

> **Note:** Maximo Anywhere is NOT supported in MAS. Migration to Maximo Mobile is required.

---

## 4.12 Deployment & Distribution

### Android Distribution
- Google Play Store (enterprise/public)
- Enterprise MDM (Microsoft Intune, Jamf, etc.)
- APK sideloading (for air-gapped environments)

### iOS Distribution
- Apple App Store (public)
- Apple Business Manager (enterprise)
- TestFlight (testing)
- MDM profiles

### MDM Configuration
When deploying via MDM:
- Pre-configure server URL
- Pre-configure authentication settings
- Push certificates
- Restrict uninstall

### App Configuration Profile (iOS)
```xml
<dict>
    <key>serverUrl</key>
    <string>https://manage.inst1.apps.cluster.com</string>
    <key>autoSync</key>
    <true/>
    <key>syncInterval</key>
    <integer>300</integer>
</dict>
```

---

## 4.13 Troubleshooting Mobile Issues

### Common Issues

| Issue | Likely Cause | Resolution |
|-------|-------------|------------|
| App won't connect | SSL cert issue | Trust enterprise CA on device |
| Login fails | Wrong server URL | Verify manage URL in app settings |
| Sync fails | Network timeout | Check network/proxy settings |
| WO not showing | Sync filter | Check assignment and status |
| Photos won't upload | Storage full | Clear app cache or free storage |
| Offline actions not syncing | Sync conflict | Check sync log in app |

### Enable Debug Logging on Mobile

iOS:
1. Settings → Maximo Mobile → Enable Diagnostic Mode
2. Reproduce issue
3. Export logs from app settings

Android:
1. Shake device in app → Debug Menu
2. Enable verbose logging
3. Export log file

### Server-Side Mobile Logs

```bash
# Check mobile API logs
oc logs -f $(oc get pods -n mas-inst1-manage | grep mobile | awk '{print $1}') -n mas-inst1-manage

# Check sync service logs
oc logs -f $(oc get pods -n mas-inst1-manage | grep sync | awk '{print $1}') -n mas-inst1-manage
```

---

## 4.14 Performance Optimization for Mobile

### Reduce Sync Payload
- Limit number of WOs synced (reduce from default 50 to 20)
- Limit sync to current week's work only
- Disable sync for large attachment types (drawings)

### Network Optimization
- Enable HTTP/2 on ingress
- Configure compression on Manage pods
- Use CDN for static assets

### Battery / Data Usage
- Configure background sync only on Wi-Fi
- Reduce sync frequency (e.g., 15 min instead of 5 min)
- Compress images before upload (configurable quality %)

---

## Summary

- Maximo Mobile provides native iOS/Android apps for field technicians
- Full offline capability using on-device SQLite database
- Covers: Work Order management, Inspections, Labor, Materials, Photos
- Maximo Anywhere is DEPRECATED — Mobile is the current standard
- Customizable via App Designer (no-code)
- Maximo Assist adds AI-powered troubleshooting capabilities

**Next:** [MAS Architecture Deep Dive](../05_Architecture/01_architecture_overview.md)
