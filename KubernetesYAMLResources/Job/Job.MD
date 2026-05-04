# Job

## 1. What is a Job?
A Job creates one or more Pods and ensures that a specified number of them successfully terminate. Once the code inside the container finishes (exits with code 0), the Job is considered complete.

In Kubernetes, a Job is used for tasks that have a definite beginning and end. While a Deployment is designed to keep a Pod running forever, a Job is designed to run a Pod until it successfully completes its task and then stops.

### Why use it?
You use a Job for Batch Processing:

- Database Migrations: Running a script to update your DB schema before a new app version starts.
- Backups: Taking a snapshot of your data and uploading it to S3.
- Data Crunching: Processing a large CSV file or generating a monthly PDF report.
- ML Training: Running a heavy computation task that ends once the model is trained.

## 2. Key Benefits: The "Task-Oriented" Logic
### A. Reliability (Retry Logic)
If the Pod fails (the code crashes or the node dies), the Job controller will automatically start a new Pod to try again. You can configure exactly how many times it should try before giving up.

### B. Parallelism
You can tell a Job to run 10 instances of a task. You can even specify that it should run 2 at a time until all 10 are done. This is great for processing large queues of data.

### C. Completion Guarantee
Unlike a simple Pod, a Job tracks the exit status. It won't stop until it sees a "Success" signal.

## 4. Essential Job Commands

| Command | Purpose |
|--------|--------|
| `kubectl get jobs` | List all jobs. Check the **COMPLETIONS** column (e.g., `1/1`). |
| `kubectl describe job <name>` | Inspect a job in detail, including failure reasons and retry attempts. |
| `kubectl logs -f -l job-name=<name>` | Stream logs from pods created by the job. |
| `kubectl delete job <name>` | Delete the job along with any completed pods it created. |
