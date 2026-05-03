# Module 03: Maximo Manage — Administration & Configuration

---

## 3.12 Security Groups

Security in Maximo Manage is role-based:

### Security Components

| Component | Description |
|-----------|-------------|
| **User** | Individual login account |
| **Security Group** | Collection of permissions |
| **Application** | Maximo application (e.g., WOTRACK) |
| **Option** | Action within an app (e.g., NEW, SAVE, DELETE) |
| **Site Access** | Which sites the group can access |

### Assigning Permissions

1. Create Security Group (e.g., TECHNICIAN)
2. Add Application access (WOTRACK, LABREP, etc.)
3. Set Options per application (Read, Insert, Save, Delete)
4. Add Site access (BEDFORD, DALLAS)
5. Assign users to the group

### Data Restriction
Groups can have **Query-based Restrictions** to limit which records users see:
```sql
-- Example: Restrict to own work orders
WONUM IN (SELECT WONUM FROM WORKORDER WHERE PERSON.PERSONID = :&LOGINID)
```

---

## 3.13 System Properties

**System Properties** control Maximo Manage behavior without code changes.

Access: **Administration → System Properties**

### Critical Properties for MAS

| Property | Description | Typical Value |
|----------|-------------|--------------|
| `mxe.hostname` | Server hostname | manage.inst1.apps.cluster.com |
| `mxe.db.url` | JDBC connection URL | jdbc:db2://db2:50000/MAXDB |
| `mxe.int.dfltuser` | Default integration user | MAXADMIN |
| `mxe.email.smtp.server` | SMTP host for email | smtp.company.com |
| `mxe.doclink.doctypes.defpath` | Doclinks base path | /doclinks |
| `webclient.defaultstartcenter` | Default start center | STARTCNTR |
| `mxe.security.auth.default` | Auth method | LDAP or LOCAL |
| `mxe.useAppServerSecurity` | Use AppServer auth | 1 (yes) or 0 |
| `mxe.workflow.admin` | Workflow admin user | MAXADMIN |

### Refreshing Properties

After changing system properties, you need to either:
1. **Refresh the JVM** (affects this server only, no downtime)
2. **Restart** Manage pods (full reload)

---

## 3.14 Domains

**Domains** define lists of valid values for fields (like a dropdown list).

Types:
- **ALN Domain** — Text values
- **NUM Domain** — Numeric values
- **Table Domain** — Values from a database table
- **Crossover Domain** — Lookup from another application

Example: WOSTATUS domain contains WAPPR, APPR, COMP, CLOSE, etc.

---

## 3.15 Cron Tasks

**Cron Tasks** are automated background jobs:

| Cron Task | Function |
|-----------|---------|
| `PMGENERATION` | Generates Work Orders from PM records |
| `LDAPSYNC` | Synchronizes LDAP users |
| `ESCALATION` | Processes SLA escalations |
| `REORDER` | Triggers inventory reorder |
| `FULLTEXTSRCH` | Refreshes full-text search index |
| `DBCLEAN` | Cleans temporary tables |
| `JMSQSEQCONSUMER` | Processes integration queue |

### Cron Task Configuration
Each cron task has:
- **Schedule** (cron expression, e.g., `0 0 * * *` = daily at midnight)
- **Active** flag
- **Run as User** (user context for permission checking)
- **Parameters** (task-specific settings)

---

## 3.16 Escalations

**Escalations** automate notifications and actions based on conditions and time.

Example escalation:
- If a Work Order stays in WAPPR status for more than 24 hours
- Then: Send email to manager
- Then: After another 24h, escalate to director

### Escalation Components
1. **Escalation** record — defines the condition
2. **Points** — time checkpoints
3. **Actions** — what to do (email, status change, etc.)

---

## 3.17 Workflows

**Workflows** automate approval and routing processes.

### Common Workflows
- Purchase Order Approval (route based on value)
- Work Order Approval (route based on priority)
- Service Request Assignment

### Workflow Components
| Component | Description |
|-----------|-------------|
| Process | The overall workflow definition |
| Node | A step in the process |
| Action | What happens at the node (email, assignment) |
| Condition | Decision logic (if/else) |
| Role | Who receives the task |

### Workflow Statuses
```
Draft → Revision → Active → Inactive
```

---

## 3.18 Actions and Communication Templates

### Actions
Actions are reusable code/logic snippets:
- Send email
- Change field value
- Execute custom class
- Set person to role
- Launch another workflow

### Communication Templates
Email templates that can include dynamic data:
```
Subject: Work Order %%WONUM%% has been approved

Dear %%REPORTEDBY%%,

Your work order %%WONUM%% - %%DESCRIPTION%% has been approved.
Scheduled Start: %%SCHEDSTART%%
Location: %%LOCATION%%
```

---

## 3.19 Maximo Application Framework (MAF) — Customization

Maximo Manage can be deeply customized without modifying base code.

### Application Designer
- Add/modify/hide fields on screens
- Create custom tabs
- Add custom sections
- Rearrange layouts
- Implement conditional visibility

### Database Configuration
- Add custom columns to tables (persistent custom attributes)
- Add custom tables entirely
- Custom indexes

### Automation Scripts
**Automation Scripts** (AS) are Groovy/Jython scripts triggered by events:

```groovy
// Example: Auto-set WO description from Asset
// Trigger: WOTRACK.save event

def wo = mbo

if (!wo.isNull("ASSETNUM")) {
    def asset = mbo.getMboSet("ASSET").getMbo(0)
    def assetDesc = asset.getString("DESCRIPTION")
    wo.setValue("DESCRIPTION", "Repair: " + assetDesc)
}
```

### Automation Script Launch Points

| Launch Point | Triggered When |
|--------------|---------------|
| Object | Before/after save, init, delete |
| Attribute | Field value changes |
| Action | Custom action button clicked |
| Custom Condition | Condition evaluation needed |
| Web Service | SOAP/REST called |
| Integration | Message processed |

---

## 3.20 Maps / Spatial Integration

Maximo Manage includes **Maximo Spatial** capability:

- View assets on a map
- Create work orders from the map
- Route technicians (integration with GIS)
- Integration with ESRI ArcGIS / OpenLayers

In MAS, Spatial is configured under **Administration → Spatial**:
- Define map provider (ArcGIS, Bing Maps, OpenStreetMap)
- Configure tile services
- Map layers for assets, locations, work orders

---

## 3.21 Maximo Manage — Industry Solution: Linear Assets

For utilities, transportation, pipelines:

### Linear Asset Concepts
- **Linear Location** — Asset described by start/end measure (e.g., road mile 0 to mile 10)
- **Feature** — Specific element on a linear asset (sign, crack, pothole)
- **Measurement Unit** — Miles, kilometers, stations

---

## 3.22 Maximo Manage — Report Management

### Built-in Reporting (BIRT)
- Business Intelligence and Reporting Tools
- Reports defined in BIRT Report Designer
- Deployed to Manage's report server pod
- Parameters, grouping, charts supported

### Report Types
- **List Reports** — Tabular data
- **Detail Reports** — Individual record with all fields
- **Analytics Reports** — Aggregated/statistical
- **KPI Reports** — Scorecard format

### Cognos (Optional)
For enterprise analytics, IBM Cognos Analytics can be integrated.

---

## 3.23 Manage Server Bundles

When Manage is installed on MAS, you configure **Server Bundles** — sets of pods for different functions:

| Bundle Type | Description |
|-------------|-------------|
| `all` | Handles all request types (default for simple deployments) |
| `ui` | Handles web UI requests |
| `mea` | Handles integration messages (MEA) |
| `report` | Handles report generation |
| `cron` | Handles cron task execution |
| `standby` | Hot standby (no user requests unless primary fails) |

### Bundle Configuration in Manage CR

```yaml
spec:
  components:
    manage:
      podTemplates:
        - name: allbundle
          replicas: 2
          resources:
            limits:
              cpu: "4"
              memory: 8Gi
            requests:
              cpu: "2"
              memory: 4Gi
```

---

## Summary

- Manage administration covers security, properties, domains, crons, escalations
- Customization options: App Designer, DB Config, Automation Scripts
- Server bundles determine how Manage pods are distributed
- Workflows automate approvals; Escalations handle SLA violations
- Industry Solutions extend for specialized verticals

**Next:** [Maximo Mobile — Complete Guide](../04_Maximo_Mobile/01_mobile_introduction.md)
