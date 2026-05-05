# CornJob

## 1. What is a CronJob?
A CronJob creates Jobs on a repeating schedule. You don't manage the Pods directly; the CronJob creates a Job, and that Job creates the Pod.

A CronJob is exactly what it sounds like: it’s a Job with a calendar. If a Job is a one-time contractor, a CronJob is a Scheduled Service. It manages the time-based execution of Jobs, using the classic Unix cron format.

### Why do we use it?

- Daily Backups: Database dumps to cloud storage every night.
- Report Generation: Sending summary emails to users every Monday morning.
- System Maintenance: Cleaning up temporary files or old logs from a database.
- Cache Refreshing: Rebuilding a search index every hour.

## 2. Key Benefits: Automation and Safety
### A. History Management
CronJobs keep a record of past runs. You can configure how many Successful and Failed jobs to keep in the history so you can go back and check logs if something went wrong yesterday.

### B. Concurrency Control
What if a Job takes 2 hours to finish, but it’s scheduled to run every hour?

- Allow: (Default) Let them run at the same time.
- Forbid: Don't start the new one until the old one finishes.
- Replace: Kill the old one and start the new one.

### C. Suspend Feature
You can "pause" a CronJob without deleting it. This is great for maintenance windows where you don't want background tasks running while you're updating the database.

## 3. The YAML Structure (The Blueprint)
```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: simple-cronjob
spec:
  schedule: "*/1 * * * *"   # runs every minute
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: simple-container
            image: busybox
            command: ["echo", "Hello from CronJob"]
          restartPolicy: Never
```

### Writing a CronJob spec
#### Schedule syntax

```
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday)
# │ │ │ │ │                                   OR sun, mon, tue, wed, thu, fri, sat
# │ │ │ │ │
# │ │ │ │ │
# * * * * *

*/15 * * * * — Every 15 minutes.

0 0 * * * — Every night at midnight.

0 9 * * 1-5 — 9:00 AM every weekday (Mon-Fri).
```

## 4. Essential CronJob Commands

| Command | Purpose |
|--------|--------|
| `kubectl get cj` | List all CronJobs. Shows the schedule and last scheduled time. |
| `kubectl describe cj <name>` | View detailed information including schedule and job history. |
| `kubectl create job --from=cronjob/<name> manual-run-01` | Manually trigger a CronJob immediately without waiting for the schedule. |
| `kubectl patch cj <name> -p '{"spec":{"suspend":true}}'` | Pause (suspend) the CronJob. |
| `kubectl get jobs --watch` | Watch Jobs in real-time as the CronJob creates them. |
