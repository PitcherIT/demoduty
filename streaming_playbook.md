Absolutely! Here’s an example SRE playbook for a fictional streaming API web service, designed for a DevOps team running on Linux EC2 nodes in AWS. This playbook covers troubleshooting API errors and latency/threshold issues. It’s written in a practical, real-world style, but all service names and details are fictional.

SRE Playbook: Troubleshooting the Streamly API Service
Overview

Service: Streamly API
Purpose: Provides real-time streaming data to client applications
Infra: Linux (Ubuntu 22.04) on Amazon EC2, behind AWS Application Load Balancer
Dependencies:

PostgreSQL (RDS)
Redis (ElastiCache)
S3 (media storage)
Internal Auth Service
External 3rd-party Data Provider
1. Initial Triage

When to use this playbook:

API error rates spike (5xx, 4xx)
Latency exceeds 500ms p95 for >5 minutes
Alert from Datadog, CloudWatch, or PagerDuty

Immediate Actions:

Acknowledge the alert in PagerDuty
Check the Streamly API Status Dashboard
Notify #streamly-incident Slack channel
2. Quick Health Checks

a. API Health Endpoint

curl -s https://api.streamly.com/healthz

Should return 200 OK and status: healthy

b. Load Balancer Health

AWS Console → EC2 → Target Groups → Check health of all Streamly API nodes

c. Error & Latency Metrics

Datadog: streamly.api.error_rate, streamly.api.latency_p95
CloudWatch: 5xxErrorCount, TargetResponseTime
3. Common Issues & Troubleshooting Steps
A. High 5xx Error Rate

Check Recent Deploys

git log --since="2 hours ago"
Rollback if a bad deploy is suspected:
./deploy rollback


Check Application Logs

SSH to a node or use CloudWatch Logs:
tail -f /var/log/streamly-api/app.log

Look for stack traces, DB errors, or OOM kills

Check Dependency Health

PostgreSQL:
psql -h <rds-endpoint> -U streamly -c "SELECT 1;"

Redis:
redis-cli -h <redis-endpoint> ping

Auth Service:
curl -s https://auth.streamly.internal/healthz


Check Disk/CPU/Memory

top, htop, df -h
If memory is >90% or disk is full, consider scaling or cleaning up logs
B. High Latency

Check Node Load

uptime, top
If load > 2x vCPU count, consider scaling out

Check Network Latency

ping <rds-endpoint>
traceroute <external-data-provider>

Check for Throttling

AWS Console → RDS/ElastiCache → CloudWatch → CPUUtilization, DatabaseConnections
Look for ThrottlingException in logs

Profile Slow Endpoints

Datadog APM: Identify slowest endpoints
Check for N+1 queries or slow external calls
4. Remediation Actions
Restart a Node:
sudo systemctl restart streamly-api

Scale Up/Out:
Increase EC2 instance size or add more nodes via ASG
Clear Disk Space:
sudo rm -rf /var/log/streamly-api/old/*

Rollback Deploy:
./deploy rollback

Contact On-Call for Dependencies:
If RDS/Redis/Auth Service is down, escalate to respective on-call
5. Escalation
If issue persists >30 minutes or customer impact is severe:
Page the on-call SRE lead
Update the incident in #streamly-incident
Start an incident Zoom bridge
6. Post-Incident
Document root cause in Confluence
Create Jira ticket for any follow-up actions
Update runbooks if new issues/processes were discovered
7. Useful Links
Streamly API Status Dashboard
Datadog Dashboard
CloudWatch Logs
Jira: Streamly Incidents
Confluence: Streamly Runbooks

Let me know if you want this in a specific format (Markdown, Confluence, etc.) or want to add more scenarios!