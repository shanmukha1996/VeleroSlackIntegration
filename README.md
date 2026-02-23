# Velero Slack Integration

A Kubernetes CronJob that monitors Velero backup schedules and sends notifications to Slack about backup status.

## Overview

This integration provides automated monitoring and alerting for Velero backup operations in Kubernetes clusters. It runs as a lightweight CronJob that periodically checks the status of your Velero backup schedules and delivers real-time notifications to your Slack channels.

**Key Benefits:**
- **Proactive Monitoring**: Stay informed about backup health without manual checks
- **Instant Alerts**: Receive immediate notifications when backups complete, fail, or encounter issues
- **Smart Filtering**: Only notifies for backups completed on the current date, reducing notification noise
- **Pause-Aware**: Automatically skips notifications when backup schedules are paused
- **Minimal Overhead**: Uses Alpine Linux container with only essential tools (kubectl, curl)
- **Flexible Scheduling**: Customize notification frequency to match your backup schedule

The integration intelligently handles different backup states (Completed, Failed, PartiallyFailed, InProgress) and provides detailed context in each notification, including timestamps, bucket information, and error counts. This ensures your team has complete visibility into backup operations and can respond quickly to any issues.

## Features

- ✅ Monitors Velero backup schedules
- 📊 Reports backup status (Success, Failed, PartiallyFailed, In Progress)
- 🔔 Sends notifications to Slack via webhook
- ⏰ Configurable schedule via CronJob
- 🛡️ Handles paused schedules gracefully
- 📅 Only notifies for backups completed on the current date

## Prerequisites

- Kubernetes cluster with Velero installed
- Velero namespace (default: `velero`)
- Slack webhook URL
- kubectl access to the cluster

## Configuration

Edit the `cronjob.yaml` file to customize:

- **Schedule**: Modify the `schedule` field (default: `"20 23 * * 1,4"` - runs at 23:20 UTC on Mondays and Thursdays)
- **CronJob name**: Update the `metadata.name` field
- **Environment variables**: Adjust the secret references as needed

### Required Secret Keys

The `slack-webhook` secret must contain:
- `webhook_url`: Your Slack webhook URL
- `schedule`: Name of the Velero schedule to monitor
- `namespace`: Velero namespace (typically `velero`)
- `cos_bucket`: Cloud Object Storage bucket name

## Installation

1. **Create a Slack webhook secret:**

```bash
kubectl create secret generic slack-webhook \
  --namespace=<namespace_where_velero_is_installed> \
  --from-literal=webhook_url='YOUR_SLACK_WEBHOOK_URL' \
  --from-literal=schedule='YOUR_SCHEDULE_NAME' \
  --from-literal=namespace='velero' \
  --from-literal=cos_bucket='YOUR_COS_BUCKET_NAME'
```

2. **Deploy the CronJob:**

```bash
kubectl apply -f cronjob.yaml
```

## How It Works

1. The CronJob runs on the specified schedule
2. Checks if the Velero schedule exists and is not paused
3. Retrieves the latest completed backup for the schedule
4. Verifies the backup was completed on the current date
5. Sends a Slack notification with backup status:
   - ✅ Success: Backup completed successfully
   - ❌ Failed/PartiallyFailed: Backup encountered errors
   - ℹ️ In Progress: Backup is still running

## Notification Format

Success notification includes:
- Environment name
- Completion timestamp (UTC)
- COS bucket name
- Backup object name
- Schedule name

## Troubleshooting

- Check CronJob logs: `kubectl logs -n <namespace_where_velero_is_installed> -l job-name=<job-name>`
- Verify secret exists: `kubectl get secret slack-webhook -n <namespace_where_velero_is_installed>`
- Ensure Velero schedule exists: `kubectl get schedules -n <namespace_where_velero_is_installed>`

