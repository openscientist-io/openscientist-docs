# REST API

OpenScientist exposes a REST API for programmatic access to jobs, API keys, job sharing and Agent Skills. Everything the web UI can do with a job can be done from a script.

- Base URL: `https://www.openscientist.io/api/v1` (replace the host for a self-hosted instance)
- Interactive reference: [Swagger UI](https://www.openscientist.io/api-docs) to try calls, [ReDoc](https://www.openscientist.io/api-redoc) for the full endpoint reference, [OpenAPI JSON](https://www.openscientist.io/openapi.json) for code generation

## Authentication

Every endpoint requires an API key sent as a Bearer token:

```
Authorization: Bearer <name>:<secret>
```

1. Sign in to the web UI and open the [API Keys](https://www.openscientist.io/api-keys) page.
2. Create a key. The full `name:secret` value is shown **once**; store it somewhere safe. Only the name is listed afterwards.
3. Keys act as the user who created them, so your account must be approved by an administrator before the key can start jobs.

Keys can also be listed and revoked through the API itself (`GET /keys`, `DELETE /keys/{key_id}`).

## Quick start

Create a job, poll it, and download the report.

```bash
export OS_KEY="my-key:abc123..."
export OS_API="https://www.openscientist.io/api/v1"

# 1. Create a job (JSON, no data files)
curl -s -X POST "$OS_API/jobs" \
  -H "Authorization: Bearer $OS_KEY" \
  -H "Content-Type: application/json" \
  -d '{
        "research_question": "Which plasma biomarkers best predict amyloid PET status?",
        "short_title": "SEABIRD biomarkers",
        "max_iterations": 10
      }'
# -> {"id": "4f2c...", "status": "pending", ...}

# 1b. Or create a job with data files (multipart; repeat data_files for each file)
curl -s -X POST "$OS_API/jobs" \
  -H "Authorization: Bearer $OS_KEY" \
  -F research_question="Investigate how proteostasis is rewired by tau pathology" \
  -F max_iterations=10 \
  -F data_files=@neurons.h5ad \
  -F data_files=@metadata.csv

# 2. Poll until it finishes
curl -s "$OS_API/jobs/$JOB_ID/status" -H "Authorization: Bearer $OS_KEY"
# -> {"id": "...", "status": "running", "current_iteration": 3, "max_iterations": 10, "error_message": null}

# 3. Fetch the final report (PDF) and all artifacts (ZIP)
curl -s "$OS_API/jobs/$JOB_ID/report"    -H "Authorization: Bearer $OS_KEY" -o report.pdf
curl -s "$OS_API/jobs/$JOB_ID/artifacts" -H "Authorization: Bearer $OS_KEY" -o artifacts.zip
```

The same in Python:

```python
import time, requests

API = "https://www.openscientist.io/api/v1"
H = {"Authorization": "Bearer my-key:abc123..."}

job = requests.post(f"{API}/jobs", headers=H, json={
    "research_question": "Which plasma biomarkers best predict amyloid PET status?",
    "max_iterations": 10,
}).json()

while True:
    s = requests.get(f"{API}/jobs/{job['id']}/status", headers=H).json()
    if s["status"] in ("completed", "failed", "cancelled"):
        break
    time.sleep(60)

if s["status"] == "completed":
    pdf = requests.get(f"{API}/jobs/{job['id']}/report", headers=H)
    open("report.pdf", "wb").write(pdf.content)
```

## Job fields

`POST /jobs` accepts these fields (JSON body or multipart form fields):

| Field | Type | Default | Notes |
|---|---|---|---|
| `research_question` | string | required | The task given to the agent |
| `short_title` | string | none | Display label, max 100 characters |
| `description` | string | none | Longer description |
| `max_iterations` | int | 5 | 1 to 20 research iterations |
| `use_hypotheses` | bool | false | Enable hypothesis tracking and testing tools |
| `investigation_mode` | string | `autonomous` | `autonomous` or `coinvestigate` (pauses for user feedback) |
| `pdb_code` | string | none | PDB code when analysing an existing structure |
| `space_group` | string | none | Crystal space group |
| `data_files` | file(s) | none | Multipart only; repeat the field for several files |

Job `status` moves through `pending`, `queued`, `running`, `awaiting_feedback` (co-investigate mode only), `generating_report`, and ends in `completed`, `failed` or `cancelled`.

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/health` | Liveness check |
| `POST` | `/jobs` | Create and queue a job |
| `GET` | `/jobs` | List your jobs; `status`, `limit` (max 1000), `offset` query parameters |
| `GET` | `/jobs/{job_id}` | Job detail, including the result summary when finished |
| `GET` | `/jobs/{job_id}/status` | Lightweight status for polling |
| `POST` | `/jobs/{job_id}/cancel` | Cancel a pending, queued or running job (stops at the next iteration checkpoint) |
| `GET` | `/jobs/{job_id}/report` | Final report as PDF (completed jobs only) |
| `GET` | `/jobs/{job_id}/artifacts` | ZIP of plots, data files and reports |
| `GET` | `/keys` | List your API keys (names only) |
| `POST` | `/keys` | Create an API key |
| `DELETE` | `/keys/{key_id}` | Revoke an API key |
| `POST` | `/shares` | Share a job with another user |
| `GET` | `/shares/job/{job_id}` | List shares for a job |
| `DELETE` | `/shares/{share_id}` | Revoke a share |
| `GET` | `/shares/search/users` | Find users by email or name |
| `GET` | `/skills` | List Agent Skills; `GET /skills/categories` for the category list |
| `GET` | `/skills/{skill_id}` | Skill detail; also `GET /skills/by-slug/{category}/{slug}` |
| `GET` | `/skills/sources` | List skill sources (Git repositories the skills are synced from) |
| `POST` | `/skills/sources` | Add a skill source |
| `POST` | `/skills/sources/{source_id}/sync` | Trigger a sync of a skill source |
| `DELETE` | `/skills/sources/{source_id}` | Remove a skill source |

All paths are relative to `/api/v1`. Request and response schemas for every endpoint are in [ReDoc](https://www.openscientist.io/api-redoc).

## Errors

Errors use standard HTTP status codes with a JSON body of the form `{"detail": "..."}`. The ones you will meet most often:

- `401` missing or invalid API key
- `403` your account is not yet approved to start jobs, or the job belongs to someone else
- `404` unknown job, key or share
- `409` the job is not in a state that allows the action, for example cancelling a finished job
- `422` validation error in the request body; the `detail` field lists the offending fields
