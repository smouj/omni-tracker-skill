---
name: Omni Tracker
version: 2.1.0
description: AI-powered infrastructure task tracking and automation orchestrator
author: SMOUJBOT
tags:
  - infrastructure
  - ai
  - automation
  - monitoring
  - devops
maintainer: ops@smouj.dev
repo: https://github.com/smouj/openclaw-omni-tracker
documentation: https://docs.openclaw.io/skills/omni-tracker
dependencies:
  - python>=3.9
  - kubernetes>=26.0
  - boto3>=1.34
  - requests>=2.31
  - pydantic>=2.0
  - click>=8.1
  - prometheus-client>=0.19
  - python-dotenv>=1.0
requires_network: true
requires_system_access: true
risk_level: medium
capabilities:
  - track
  - analyze
  - predict
  - automate
  - alert
---

# Omni Tracker

AI-powered infrastructure task tracking, anomaly detection, and automation orchestration for Kubernetes clusters, cloud resources, and on-prem systems.

## Purpose

Omni Tracker continuously monitors infrastructure components, uses AI to detect anomalies, predicts resource exhaustion, and automatically triggers remediation workflows or escalates to human operators. It replaces manual log reviewing and reactive firefighting with proactive, intelligent infrastructure management.

### Real Use Cases

1. **Predictive Scaling**: Analyze pod metrics and HPA behavior to predict when deployments will need scaling 15 minutes before threshold breach, automatically adjusting limits or notifying teams.

2. **Cost Anomaly Detection**: Track cloud spend across AWS/GCP/Azure, detect spending spikes >30% above baseline, correlate with recent deployments, and auto-create Jira tickets with root cause hypotheses.

3. **Deployment Rollback Guardian**: Monitor canary deployments; if AI detects error rate increase >5% or latency p95 increase >200ms, automatically rollback and generate post-mortem.

4. **Security Exposure Tracker**: Scan for exposed services, outdated images with CVEs, and misconfigurations; prioritize based on threat intelligence and asset criticality.

5. **Capacity Planning Assistant**: Monthly forecasting of CPU, memory, and storage needs using time-series analysis; recommend node pool expansions or rightsizing opportunities.

## Alcance

### Commands

```bash
omni-tracker --config /etc/omni-tracker/config.yaml start
omni-tracker --config /etc/omni-tracker/config.yaml status
omni-tracker --config /etc/omni-tracker/config.yaml stop
omni-tracker --config /etc/omni-tracker/config.yaml analyze --period 1h --output json
omni-tracker --config /etc/omni-tracker/config.yaml predict --resource cpu --namespace production --hours 6
omni-tracker --config /etc/omni-tracker/config.yaml anomaly detect --threshold 0.85 --notify slack
omni-tracker --config /etc/omni-tracker/config.yaml remediation run --type deployment --name payment-service --action rollback
omni-tracker --config /etc/omni-tracker/config.yaml cost report --month 2026-03 --provider aws --format table
omni-tracker --config /etc/omni-tracker/config.yaml security audit --scan-type full --output html --email team@company.com
omni-tracker --config /etc/omni-tracker/config.yaml forecast capacity --cluster prod-cluster --quarter Q2-2026
omni-tracker --config /etc/omni-tracker/config.yaml webhook trigger --event deployment_failed --payload '{\"service\":\"auth\"}'
```

### Configuration

`~/.openclaw/omni-tracker/config.yaml`:
```yaml
trackers:
  kubernetes:
    enabled: true
    clusters:
      - name: production
        context: kubeconfig-production
        namespace_filter: [production, staging]
        metrics_interval: 30s
      - name: development
        context: kubeconfig-dev
        namespace_filter: [development]
        metrics_interval: 60s

  cloud:
    enabled: true
    providers:
      aws:
        regions: [us-east-1, eu-west-1]
        access_key: ${AWS_ACCESS_KEY_ID}
        secret_key: ${AWS_SECRET_ACCESS_KEY}
        services: [ec2, rds, s3, lambda]
      gcp:
        project: my-project
        credentials_file: /path/to/gcp-key.json
        services: [compute, storage, bigquery]

  onprem:
    enabled: true
    hosts:
      - db-prod-01.internal
      - cache-prod-02.internal
    ssh_user: omni-tracker
    ssh_key: /home/omni/.ssh/id_rsa_omni

ai:
  anomaly_detection:
    model: isolation_forest
    contamination: 0.01
    train_interval: 24h
    features:
      - pod_restart_rate
      - memory_usage_variance
      - request_latency_p99
      - error_rate_percentage

  prediction:
    enabled: true
    horizon: 6h
    confidence_threshold: 0.8
    algorithms:
      cpu: prophet
      memory: lstm
      storage: linear_regression

  correlation:
    enabled: true
    time_window: 15m
    min_correlation: 0.7

remediation:
  auto_execute: false  # Requires manual approval in production
  approval_webhook: https://openclaw.company.com/approve-remediation
  actions:
    deployment_rollback:
      command: kubectl rollout undo deployment/{name} -n {namespace}
      timeout: 300s
    pod_restart:
      command: kubectl delete pod {pod_name} -n {namespace}
      timeout: 60s
    node_drain:
      command: kubectl drain {node_name} --ignore-daemonsets --delete-emptydir-data
      timeout: 600s

notifications:
  slack:
    webhook_url: ${SLACK_WEBHOOK_URL}
    channel: #infrastructure-alerts
    mention_on:
      - critical
      - warning

  pagerduty:
    integration_key: ${PAGERDUTY_INTEGRATION_KEY}
    escalation_rules:
      - severity: critical
        response_time: 5m
        escalation_after: 15m

  email:
    smtp_server: smtp.company.com
    from: omni-tracker@company.com
    recipients:
      - platform-team@company.com
    daily_report: true
    time: 09:00

storage:
  timeseries:
    type: prometheus
    url: http://prometheus.monitoring.svc.cluster.local:9090

  events:
    type: elasticsearch
    url: https://es-prod.company.com
    index: omni-tracker-events

  models:
    path: /var/lib/omni-tracker/models
    backup_s3: s3://company-omni-tracker-backup/models

security:
  tls:
    cert_file: /etc/omni-tracker/tls/tls.crt
    key_file: /etc/omni-tracker/tls/tls.key

  rbac:
    cluster_role: omni-tracker-operator
    service_account: omni-tracker
```

## Proceso de Trabajo

### 1. Initialization

- Validate configuration file syntax and required secrets
- Connect to all configured trackers (K8s clusters, cloud APIs, SSH hosts)
- Load trained AI models from storage path
- Initialize Prometheus/Elasticsearch clients
- Start metrics collection schedulers
- Register signal handlers for graceful shutdown

### 2. Data Collection

- **K8s**: Watch API server every `metrics_interval`; scrape pod metrics from kube-state-metrics and metrics-server
- **Cloud**: Query CloudWatch, Cloud Monitoring, or AWS Cost Explorer API every 5 minutes
- **On-prem**: SSH and run custom scripts to gather system metrics; cache results for 60s
- All metrics tagged with: cluster, namespace, pod, node, timestamp, labels

### 3. AI Analysis Pipeline

1. **Preprocessing**: Normalize metrics, handle missing values, remove outliers beyond 3 sigma
2. **Feature Engineering**: Compute rolling averages, rates of change, ratios (memory/request), entropy scores
3. **Anomaly Detection**: Run isolation forest on feature vectors; score 0-1 (1=extreme anomaly)
4. **Prediction**: Prophet/LSTM models forecast next 6 hours for tracked resources
5. **Correlation**: Pearson correlation between anomalies across services to identify root cause cascades
6. **Root Cause Analysis**: Build dependency graph from K8s service mesh and trace anomalies upstream

### 4. Remediation Decision

- Assign severity: `critical` (score>0.9), `warning` (0.7-0.9), `info` (<0.7)
- Check `auto_execute` flag:
  - If auto: execute remediation command with timeout, log outcome
  - If manual: send notification with approval webhook, wait for response
- Escalate to PagerDuty if no response within SLA
- Record decision in events store with full trace

### 5. Reporting & Dashboard

- Daily email digest at `notifications.email.daily_report` time
- Real-time Slack alerts with metric charts (generated from Prometheus range queries)
- Weekly capacity forecast PDF sent to platform-team@
- All reports stored in Elasticsearch with Kibana dashboard available at https://omni-tracker.company.com

## Reglas de Oro

1. **Never auto-execute in production**: `remediation.auto_execute` MUST be false for clusters tagged `production`. Auto-rollback only for non-critical namespaces.

2. **Preserve model integrity**: Never delete files from `storage.models.path` without backup. Rotate models only after successful retraining and validation on holdout set.

3. **Rate limit external APIs**: Cloud provider APIs have quotas; use exponential backoff with jitter. Max 1 request per second per region.

4. **Secure secrets**: All API keys and webhooks MUST come from environment variables, never hardcoded. Rotate Slack/PagerDuty integrations quarterly.

5. **Data retention policy**: Timeseries data older than 90 days archived to S3 Glacier. Raw events kept 2 years for audit.

6. **False positive review**: Weekly review of all anomalies scored 0.7-0.85; retrain models if false positive rate >20%.

7. **Change management**: All remediation commands require human approval unless in `remediation.allowed_auto_namespaces` (e.g., `test`, `sandbox`).

## Examples

### Example 1: Detect Memory Leak in Payment Service

**Input**: `omni-tracker anomaly detect --threshold 0.8 --namespace payments --last 2h`

**Background**: Pod `payment-api-7df8c` shows memory increasing 50MB every 10 minutes with no GC pressure.

**Output (Slack notification)**:
```
[CRITICAL] Memory anomaly detected in payment-api (prod)
Namespace: payments
Pod: payment-api-7df8c4f9b-xyz12
Anomaly score: 0.94
Metric: memory_usage_bytes (current: 1.2Gi, predicted: 450Mi)
Correlation: 0.87 with gc_collection_paused_seconds_total increase
Suggested action: restart pod to clear heap
Approve rollback? https://omni-tracker.company.com/approve/req_ABC123
```

**Action**: Human approves via webhook → tracker executes `kubectl delete pod payment-api-7df8c4f9b-xyz12 -n payments --force`

### Example 2: Cost Spike Investigation

**Input**: `omni-tracker cost report --provider aws --range 2026-03-01..2026-03-07 --format table`

**Output (CLI table)**:
```
+---------------------------+----------------+--------------+----------------+-------------------+
| SERVICE                   |  NORMAL COST   |  ACTUAL COST |  DELTA (%)     |  ANOMALY SCORE    |
+---------------------------+----------------+--------------+----------------+-------------------+
| EC2                       |    $12,450     |    $18,230   |    +46.4%      |     0.92          |
| Lambda                    |     $1,200     |     $1,350   |    +12.5%      |     0.34          |
| S3                        |     $3,800     |     $4,100   |     +7.9%      |     0.21          |
+---------------------------+----------------+--------------+----------------+-------------------+

Top correlated event: 2026-03-04 14:30UTC - Deployed data-pipeline v2.1.3 (JIRA: PLAT-2847)
Recommendation: Investigate EC2 instance type changes or runaway spot实例竞价实例.
```

**Action**:Ops team checks deployed change; finds data-pipeline increased node count from 5→20; adjusts HPA limits.

### Example 3: Predictive Storage Exhaustion

**Input**: `omni-tracker predict --resource storage --namespace logs --days 3`

**Output (JSON)**:
```json
{
  \"resource\": \"persistentvolumeclaim/logs-storage\",
  \"current_used_bytes\": 89000000000,
  \"capacity_bytes\": 100000000000,
  \"prediction\": {
    \"timestamp\": \"2026-03-08T00:00:00Z\",
    \"predicted_used_bytes\": 102500000000,
    \"confidence_interval_lower\": 101200000000,
    \"confidence_interval_upper\": 103800000000,
    \"trend\": \"+2.3GB/day\"
  },
  \"severity\": \"critical\",
  \"recommendation\": \"Increase PVC from 100Gi to 150Gi within 48 hours or implement log rotation\",
  \"remediation\": \"kubectl patch pvc logs-storage -n logs --type merge -p '{\\"spec\\":{\\"resources\\":{\\"requests\\":{\\"storage\\":\\"150Gi\\"}}}}'\"
}
```

**Action**: Platform engineer runs remediation command; PVC expanded without downtime.

## Rollback

### Undo Automatic Remediation

If auto-rollback was triggered mistakenly:

```bash
# View recent remediation actions
omni-tracker events list --type remediation --last 10 --output table

# Revert specific action
omni-tracker remediation undo --event-id evt_12345abc --verify

# Manual restore from backup
kubectl apply -f /var/lib/omni-tracker/backups/payment-api-2026-03-05-14-30.yaml -n payments
```

### Restore Previous AI Models

If model retraining degraded performance:

```bash
# List model checkpoints
ls -la /var/lib/omni-tracker/models/checkpoints/

# Restore checkpoint from yesterday
omni-tracker models restore --checkpoint models/checkpoints/anomaly-2026-03-04.pkl --confirm

# Retrain with corrected data
omni-tracker models retrain --dataset s3://omni-tracker-correction/2026-03-05/ --validate
```

### Disable Tracker Temporarily

```bash
# Stop collection (preserves data, stops analysis)
omni-tracker stop --graceful --timeout 30s

# Disable specific tracker in config without full stop
sed -i 's/kubernetes:/# kubernetes:/' ~/.openclaw/omni-tracker/config.yaml
omni-tracker reload-config
```

### Cloud API Rate Limit Recovery

If API calls throttled:

```bash
# View current quota usage
omni-tracker cloud quotas --provider aws --region us-east-1

# Implement backoff immediately
omni-tracker config set ai.anomaly_detection.request_interval 10s
omni-tracker config set cloud.providers.aws.call_rate 0.5

# Resume normal rate after 1 hour
omni-tracker config set cloud.providers.aws.call_rate 1.0
```

## Solución de Problemas

### Metrics Not Appearing in Dashboard

1. Check Prometheus scrape targets: `kubectl get endpoints -n monitoring kube-state-metrics`
2. Verify tracker can reach Prometheus: `omni-tracker test connectivity --target prometheus --timeseries-db`
3. Check timezone alignment: `date && omni-tracker status | grep \"System time\"`
4. Increase log level: `omni-tracker --log-level debug start 2>&1 | tee /tmp/omni-debug.log`

### High False Positive Rate

1. Review recent anomalies: `omni-tracker anomalies list --score 0.7..0.9 --last 24h`
2. Retrain with labeled data: `omni-tracker models retrain --labeled-data /path/to/labeled.csv`
3. Adjust contamination parameter: edit config `ai.anomaly_detection.contamination: 0.005`
4. Add whitelist: `omni-tracker config add ai.anomaly_detection.ignore_patterns 'pod_name regex:test-.*'`

### Remediation Fails

1. Check RBAC: `kubectl auth can-i delete pod --as=system:serviceaccount:omni-tracker:omni-tracker-operator`
2. Verify command syntax: `omni-tracker remediation dry-run --type deployment --name mysvc --action rollback`
3. Ensure target resource exists: `kubectl get deployment mysvc -n target-ns`
4. Review event logs: `omni-tracker events get --id evt_failed_123`

### AI Model Loading Error

```bash
# Check model file integrity
omni-tracker models validate --path /var/lib/omni-tracker/models/anomaly.pkl

# Clear corrupted model and retrain
rm /var/lib/omni-tracker/models/anomaly.pkl
omni-tracker models train --backfill 7d

# Check Python dependencies
pip list | grep -E '(scikit-learn|prophet|tensorflow)'
```

### Data Gaps from Cloud Provider

1. Check API credentials: `omni-tracker cloud test --provider aws --service ec2`
2. Verify service is enabled in IAM: AWS console → IAM role → Access Advisor
3. Increase timeout: `omni-tracker config set cloud.providers.aws.timeout 30s`
4. Switch to fallback region: edit config file, add `fallback_regions: [us-west-2]`

## Environment Variables

Required:

```bash
export AWS_ACCESS_KEY_ID="AKIAIOSFODNN7EXAMPLE"
export AWS_SECRET_ACCESS_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
export SLACK_WEBHOOK_URL="https://hooks.slack.com/services/..."
export PAGERDUTY_INTEGRATION_KEY="abcd1234efgh5678ijkl"
export GCP_APPLICATION_CREDENTIALS="/path/to/key.json"
```

Optional:

```bash
export OMNI_TRACKER_LOG_LEVEL="DEBUG"           # Default: INFO
export OMNI_TRACKER_CONFIG_OVERRIDE="/custom/config.yaml"
export PROMETHEUS_URL="http://custom-prom:9090"
export S3_MODEL_BACKUP_BUCKET="company-omni-backup"
export JIRA_API_TOKEN="token_for_ticket_creation"
```

## Dependencies

```bash
# Python packages (install via pip)
pip install omni-tracker[k8s,cloud,ai,notify]

# System packages
apt-get install -y python3-dev libssl-dev ssh

# Kubernetes access
kubectl config view --minify --flatten > ~/.kube/config-omni
export KUBECONFIG=~/.kube/config-omni:~/.kube/config

# Docker (if running as container)
docker pull smouj/omni-tracker:2.1.0
docker run -d \
  -v ~/.openclaw/omni-tracker:/config \
  -v /var/lib/omni-tracker:/data \
  -e AWS_ACCESS_KEY_ID \
  -e SLACK_WEBHOOK_URL \
  --name omni-tracker \
  smouj/omni-tracker:2.1.0 start --config /config/config.yaml
```

## Verification

```bash
# 1. Check process is running
ps aux | grep omni-tracker | grep -v grep
# Expected: python /usr/local/bin/omni-tracker start ...

# 2. Verify connections
omni-tracker health check-all
# Expected output:
# [OK] Kubernetes cluster production (context kubeconfig-production)
# [OK] AWS region us-east-1 (EC2, RDS, S3)
# [OK] Prometheus http://prometheus.monitoring.svc:9090
# [OK] Slack webhook endpoint

# 3. Test anomaly detection on sample data
omni-tracker analyze test --sample-data tests/samples/memory-leak.json
# Expected: "Anomaly detected: score=0.92 (CRITICAL)"

# 4. Simulate notification
omni-tracker notify test --channel slack --severity warning
# Expected: Message posted to #infrastructure-alerts

# 5. Verify model predictions
omni-tracker predict dry-run --resource cpu --namespace production --hours 6
# Expected: JSON with predictions, no errors
```

## Performance Tuning

- For clusters >100 pods: increase `metrics_interval` to 60s to reduce API server load
- For high-frequency anomalies: reduce `ai.anomaly_detection.train_interval` to 6h
- For multi-region cloud: enable `ai.correlation.time_window: 30m` to account for network latency
- Storage optimization: switch `storage.timeseries.type` to `thanos` for long-term retention
```