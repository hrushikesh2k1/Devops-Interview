1. Suddenly a severe incident raised. What is your action for First 15min?

   I would immediately acknowledge the incident and will work on collecting information about the issue.
   I would check the pod health by kubectl describe to check for the errors (crashLoopBackOff, OOMKilled, Readiness/Liveness probe failure, Recent pod restarts,
   service endpoints, Node pressure using Kubectl top node, Any HPA scaling issues, Ingress errors or 5xx error, check for recent rollouts using kubectl rollout undo deployment).
   By checking all of these I will collect the all possible information and primarily decide, is this infra issue, application issue, networking issue or capacity issue?

   Minutes 2–5: Declare the Incident Early 
   I would declare it early — even if I'm not 100% sure of the severity. 
   This gets the right people looped in faster.I will open a dedicated Slack channel or war room, assign an Incident Commander 
   (or take that role temporarily), and notify stakeholders."

  Minutes 5–10: Communicate and Escalate
  "I'd post a quick initial message to the incident channel: 'Production incident declared. Symptom: [X]. 
  Impact: [Y users/services]. Currently investigating. Next update in 10 minutes.' 
  Then I'd escalate based on what I'm seeing. 
  If it's a Kubernetes cluster issue, I'm paging the platform team. 
  If it's an identity/auth failure, I'm pulling in the IAM team. 
  In Azure environments I've worked in, this could mean escalating to the networking or AAD team depending on the error."

  Minutes 10–15: Attempt Generic Mitigation, Not Root Cause
  "This is critical — and something I've studied closely. In the first 15 minutes, your goal is mitigation, not root cause analysis. 
  Root cause can come later. So I'd look for quick levers: Can I roll back the last deployment? In Kubernetes I'd run kubectl rollout undo deployment/<name>. 
  Can I increase replicas to handle load? Can I reroute traffic away from a failing region? 
  Can I temporarily disable a feature flag? These are blunt instruments but they stop the bleeding fast. 
  The SRE principle here is: stop customer pain first, understand why later."

  What I'd Avoid
  "I'd avoid two traps: First, going deep into root cause debugging before mitigation — that costs time while users are suffering. 
  Second, having multiple people investigating independently without coordination — that creates noise and miscommunication. 
  I'd keep everyone synchronized through the incident channel with updates every few minutes."


2. What security measures you have taken for infrastructure provisioning like to control drift?

   a) Statefile stored in Azure Blob Storage, provides native state locking preventing conflicting changes.
   b) All terraform changes go through GitHub Actions with mandatory PR check at terraform plan- direct CLI apply on prod is blocked via Azure RBAC (only the pipeline service principal has contributor access).
   c) Schedule drift detection pipeline
   ```bash
      schedule * */6 * * *
      terraform plan -detailed-exitcode

      0- no drfit
      1-
      2- 
   ```
   d) Implemented Azure policy for continuous complaince.
     "All VMs must use approved VM sizes only".
     "Storage accounts must have public access disabled".
     "All resources must have mandatory tags: env, prod, ".
     "NSGs must not allow inbound 0.0.0.0/0 on port 22".
     "Allowed location policy"
     "Resource tagging like costCenter, Onwer, Environment, Application Name
     "Enforcing encryption at rest"
     "Allowed VM sku's"
     "Public IP restriction, VM should not be associated with public IP's"

   e) Even after the resources are deployed, OS configuration can drift. This can be handled by Azure machine configuration service. We will define the desired OS state in azure machine configuration file.
      Azure automation pulls this config onto every registered VM every 15-30 min. if someone manually uninstalls IIS or modifies the config file, AMC remediates it on the next pull cycle.
   f) GitOps with ArgoCD on AKS for kubernetes drift. ArgoCD deployed on AKS monitors your GitHub repos as the source of truth. With auto-sync enabled, every 3 minutes, argocd compares live AKS cluster state. If someone runs kubectl apply manually or edits anything directly, ArgoCD reconciles back to Git state automatically.
   ```yaml
    # ArgoCD Application manifest
    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
      name: myapp
    spec:
      source:
        repoURL: https://github.com/yourorg/k8s-manifests
        targetRevision: main
        path: apps/myapp
      destination:
        server: https://kubernetes.default.svc
        namespace: production
      syncPolicy:
        automated:
          prune: true      # Remove resources not in Git
          selfHeal: true   # Auto fix drift
    ```

g) Azure Monitor + Activity Log Alerts for Rogue Changes

  - **Azure Activity Log** records every single change made to any Azure resource — who did it, when, from where
  - Set up **Azure Monitor Alert Rules** on the Activity Log:
  - *"Alert me if any resource is modified outside of the pipeline service principal"*
  - *"Alert if any NSG rule is added manually via portal"*
  - Alerts sent to **Microsoft Teams channel or email** via Action Groups
  - Integrate with **Azure Sentinel (Microsoft Defender for Cloud)** for security- level drift monitoring

h) Enforce "No Manual Changes" via Azure RBAC

The organizational guardrail that prevents drift at the source:

  - **Production subscription:** Only the **pipeline service principal** has            Contributor` role
  - All humans get `Reader` role on production — they can see but not touch
  - Developers work on **dev/staging subscriptions** with full access
  - All production changes **must go through Azure DevOps Pipeline** — this is          enforced by RBAC, not just policy

### The Full Drift Protection Architecture on Azure
```
Git Repository (Source of Truth)
        ↓
Azure DevOps Pipeline (PR triggers terraform plan)
        ↓
Terraform → Azure Blob Storage (state lock)
        ↓
Resources deployed in Azure
        ↓
Azure Policy → continuous compliance check → auto-remediate
Azure Monitor → activity log alerts → notify on manual changes
Azure DSC → OS-level config enforced every 30 mins
ArgoCD → K8s drift reconciled every 3 mins
        ↓
Drift Report → Teams Alert + ADO Work Item
```
"On Azure, drift protection is a layered approach. At the provisioning layer we use Terraform with Azure Blob backend for state locking. At the compliance layer, Azure Policy continuously evaluates and auto-remediates resource drift. At the OS layer, Azure Automation DSC enforces desired state on every VM. For Kubernetes on AKS, ArgoCD handles GitOps reconciliation. And organizationally, we enforce immutability through RBAC — only the pipeline service principal can modify production, so manual changes are architecturally impossible."


## 3. What monitoring tools you have used, how do you implemented alerting?
### How to Open Your Answer
> *"Our monitoring strategy was layered — we used **Prometheus + Grafana** for Kubernetes-level and application-level observability, and **Azure Monitor** for infrastructure-level and platform-level monitoring across VMs, VMSS, and managed databases. Alerting was not just email notifications — we built a full alerting pipeline with severity classification, routing, and auto-remediation."*
---

## Layer 1 — Kubernetes Monitoring with Prometheus + Grafana (AKS)

### How Prometheus Was Deployed
```bash
# Deployed via Helm using kube-prometheus-stack
# This installs Prometheus, Alertmanager, Grafana, and all default K8s exporters in one shot

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --values custom-values.yaml
```

The `kube-prometheus-stack` automatically deploys:
- **Prometheus** — scrapes metrics
- **Alertmanager** — routes and deduplicates alerts
- **Node Exporter** — exposes VM/node level metrics (CPU, memory, disk)
- **kube-state-metrics** — exposes Kubernetes object states (pod restarts, deployment status)
- **Grafana** — visualization

### What Metrics Were Scraped
Prometheus works on a **pull model** — it scrapes `/metrics` endpoints at defined intervals:

```yaml
# prometheus scrape config example
scrapeConfigs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
```

For your **application pods**, you'd annotate them:

```yaml
annotations:
  prometheus.io/scrape: "true"
  prometheus.io/port: "8080"
  prometheus.io/path: "/metrics"
```

Prometheus then **auto-discovers** and scrapes your app metrics.

### Alerting Rules in Prometheus

```yaml
# PrometheusRule CRD — defined as Kubernetes manifest
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: app-alerts
  namespace: monitoring
spec:
  groups:
    - name: pod-alerts
      rules:

        # Alert if pod keeps restarting
        - alert: PodCrashLooping
          expr: rate(kube_pod_container_status_restarts_total[5m]) * 60 > 2
          for: 5m
          labels:
            severity: critical
            team: backend
          annotations:
            summary: "Pod {{ $labels.pod }} is crash looping"
            description: "Pod has restarted more than 2 times in last 5 mins"

        # Alert if node CPU > 85%
        - alert: HighNodeCPU
          expr: 100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 85
          for: 10m
          labels:
            severity: warning
          annotations:
            summary: "Node {{ $labels.instance }} CPU above 85%"

        # Alert if memory usage > 90%
        - alert: HighMemoryUsage
          expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100 > 90
          for: 5m
          labels:
            severity: critical
          annotations:
            summary: "Memory usage critical on {{ $labels.instance }}"
```

> **Key Point:** The `for` field means Prometheus fires the alert **only if the condition persists** for that duration, avoiding false positives from transient spikes.

### Alertmanager — Routing Alerts to the Right Team

```yaml
# alertmanager.yaml
global:
  resolve_timeout: 5m

route:
  receiver: 'default'
  group_by: ['alertname', 'team']
  group_wait: 30s       # Wait before sending first alert (groups related alerts)
  group_interval: 5m    # How long before sending new alerts for same group
  repeat_interval: 4h   # Re-notify if alert still firing after 4 hours

  routes:
    - match:
        severity: critical
      receiver: 'pagerduty-critical'

    - match:
        severity: warning
      receiver: 'teams-warning'

    - match:
        team: backend
      receiver: 'backend-team-slack'

receivers:
  - name: 'pagerduty-critical'
    pagerduty_configs:
      - service_key: '<key>'

  - name: 'teams-warning'
    webhook_configs:
      - url: 'https://outlook.office.com/webhook/...'

  - name: 'backend-team-slack'
    slack_configs:
      - api_url: '<slack-webhook>'
        channel: '#backend-alerts'
        text: '{{ .CommonAnnotations.summary }}'
```
> *"We used Alertmanager routing trees to ensure critical alerts went to PagerDuty for on-call engineers, warnings went to Microsoft Teams, and team-specific alerts were routed to the relevant Slack channel — so the right people got the right alerts."*
---

## Layer 2 — Azure Monitor for VMs, VMSS, and Databases

### VM / VMSS Monitoring

- Enabled **Azure Monitor Agent (AMA)** on all VMs — the modern replacement for the old MMA agent
- Created **Data Collection Rules (DCR)** to define exactly what to collect:

```json
{
  "dataSources": {
    "performanceCounters": [
      {
        "name": "VMPerformanceCounters",
        "samplingFrequencyInSeconds": 60,
        "counterSpecifiers": [
          "\\Processor Information(_Total)\\% Processor Time",
          "\\Memory\\Available Bytes",
          "\\LogicalDisk(_Total)\\% Free Space",
          "\\Network Interface(*)\\Bytes Total/sec"
        ]
      }
    ],
    "windowsEventLogs": [
      {
        "name": "eventLogsDataSource",
        "streams": ["Microsoft-Event"],
        "xPathQueries": ["Application!*[System[(Level=1 or Level=2)]]"]
      }
    ]
  }
}
```

All metrics flow into **Log Analytics Workspace** — the central store for all logs and metrics.

### Log-Based Alerts with KQL

Azure Monitor alerts are written in **Kusto Query Language (KQL)**:

```kusto
// Alert: Detect application errors in logs
AppExceptions
| where TimeGenerated > ago(5m)
| where SeverityLevel == "Error"
| summarize ErrorCount = count() by AppRoleName, bin(TimeGenerated, 1m)
| where ErrorCount > 10
```

```kusto
// Alert: VM CPU above 90% sustained
Perf
| where ObjectName == "Processor Information"
| where CounterName == "% Processor Time"
| where TimeGenerated > ago(10m)
| summarize AvgCPU = avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
| where AvgCPU > 90
```

```kusto
// Alert: Detect failed login attempts (security)
SecurityEvent
| where EventID == 4625
| where TimeGenerated > ago(10m)
| summarize FailedLogins = count() by Account, Computer
| where FailedLogins > 5
```

These KQL queries are attached to **Scheduled Query Alert Rules** in Azure Monitor with a defined evaluation frequency (every 5 mins) and threshold.

### Database Monitoring — Azure SQL & Cosmos DB

**For Azure SQL:**
- Enabled **SQL Insights** via Azure Monitor
- Key metrics monitored: DTU consumption, deadlocks, failed connections, query duration
- Alert rule: *"If DTU percentage > 80% for 15 minutes → warning alert"*
- Enabled **Query Performance Insight** to identify top resource-consuming queries

**For Cosmos DB:**
- Monitored **RU/s consumption** — if consumed RUs approach provisioned RUs, throttling (429 errors) starts
- Alert: *"If NormalizedRUConsumption > 80% → scale up RU/s via Azure Automation runbook"* — this is auto-remediation

```kusto
// Cosmos DB throttling alert
AzureDiagnostics
| where ResourceType == "DATABASEACCOUNTS"
| where statusCode_s == "429"
| where TimeGenerated > ago(5m)
| summarize ThrottleCount = count() by databaseName_s
| where ThrottleCount > 50
```

---

## Layer 3 — Grafana Dashboards (Unified View)

- Used **Grafana with multiple data sources** — Prometheus for K8s metrics AND Azure Monitor as a data source
- This gave a **single pane of glass** — one dashboard showing both K8s pod health and Azure VM metrics side by side
- Built dashboards for:
  - **Infrastructure dashboard** — node CPU, memory, disk per VM/node
  - **AKS dashboard** — pod restarts, deployment health, HPA scaling events
  - **Application dashboard** — request rate, error rate, response time (RED method)
  - **Database dashboard** — query latency, connection pool, DTU/RU consumption
- Set up **Grafana Alerting** as a secondary layer with alert rules directly on dashboard panels

---

## Alerting Best Practices

### Severity Classification

| Severity | Tool | Audience | SLA |
|----------|------|----------|-----|
| P1 Critical | PagerDuty | On-call engineer | Woken up immediately |
| P2 Warning | Microsoft Teams | Team channel | Acknowledged within 30 mins |
| P3 Info | Email digest | Team lead | Reviewed next business day |

### Avoiding Alert Fatigue

- Used `for` duration in Prometheus rules to avoid firing on transient spikes
- Set `group_wait` and `repeat_interval` in Alertmanager to batch related alerts
- Regularly reviewed and **pruned noisy alerts** that had low action rate

### Runbooks Attached to Every Alert

Every alert had an annotation with a **Confluence runbook link** — so on-call engineers knew exactly what to do when they received the alert.

---

## Full Architecture Overview

```
AKS Pods / Nodes
      ↓ scrape /metrics
Prometheus (pull model)
      ↓ evaluate rules
Alertmanager
      ↓ route by severity/team
PagerDuty / Teams / Slack

Azure VMs / VMSS / SQL / Cosmos
      ↓ Azure Monitor Agent (AMA)
Log Analytics Workspace
      ↓ KQL Scheduled Query Rules
Azure Monitor Alerts
      ↓ Action Groups
Teams / Email / Azure Automation (auto-remediation)

Grafana ← Prometheus + Azure Monitor (single pane of glass)
```

---

## Closing Statement for the Interview

> *"Our monitoring was a two-layer system. Prometheus with Alertmanager handled all Kubernetes and application-level observability — with PromQL-based alert rules, severity-based routing, and Alertmanager grouping to prevent alert fatigue. Azure Monitor with Log Analytics handled infrastructure and platform monitoring — using KQL-based scheduled query alerts for VMs, VMSS, SQL, and Cosmos DB. Everything was visualized in Grafana as a single pane of glass. Alerts were classified by severity and routed to PagerDuty, Teams, or Slack depending on urgency — and every alert had a runbook attached so on-call response was fast and consistent."*


## 4. How do you handle documentation in your organization? which automation tool did you use for it? 
