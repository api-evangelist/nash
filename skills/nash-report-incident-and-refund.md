---
name: Report a Nash delivery incident and track a refund
description: File a delivery incident against a task, optionally request a refund, and track the refund request through review.
api: openapi/nash-openapi-original.json
operations:
- delivery_incident_v1_jobs__string_jobId__tasks__string_taskId__delivery_incident_post
- get_refund_request_v1_refund_request__string_id__get
- get_refund_requests_v1_refund_requests_get
- get_job_v1_jobs__string_id__get
method: generated
source: openapi/nash-openapi-original.json + https://docs.usenash.com/reference/refunds-and-incidents
---

# Report a Nash delivery incident and track a refund

Use this when a delivery had a problem (late, damaged, failed) and you need to record it and possibly recover cost.

## Auth
`Authorization: Bearer $NASH_API_KEY` (+ `Nash-Org-Id` when needed). Base `https://api.usenash.com/v1`.

## Steps
1. **Identify the job and task** — a job (`job_...`) contains one or more tasks (`tsk_...`). Use `get_job_v1_jobs__string_id__get` to inspect tasks and their current delivery status/`failureReason`.
2. **Create the incident** — `delivery_incident_v1_jobs__string_jobId__tasks__string_taskId__delivery_incident_post` files an incident on the specific task; you can optionally request a refund as part of it.
3. **Track the refund** — poll `get_refund_request_v1_refund_request__string_id__get` for one request, or `get_refund_requests_v1_refund_requests_get` filtered by task ID or job ID, to follow it through review.

## Rules
- A job can hold multiple tasks/attempts — file the incident against the correct `taskId`, and handle each task independently.
- `TASK_NOT_FOUND` (404) / `TASK_FORBIDDEN` (403) indicate a wrong or unowned task id.
- Refund requests are reviewed asynchronously; do not assume immediate approval.
- Errors and retry guidance: `errors/nash-error-codes.yml`.
