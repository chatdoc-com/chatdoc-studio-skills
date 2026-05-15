# Agent App API

The Agent App API allows you to submit document-processing tasks to a published Agent App and poll for task status and final results.

## Base Path

```
/agent/apps/tasks
```

## Prerequisites

Before calling this API:

1. Create and publish the Agent App in ChatDOC Studio.
2. Use an API key that has `AGENT` capability.
3. Upload the source document to your team first via the Uploads API.
4. Check the Agent App's document slot labels. Task documents must be grouped by these labels.

## Important Notes

- Use `app_id` to identify the target Agent App.
- Use `documents` to pass uploads grouped by document slot label.
- Each `documents[].label` must match a slot label defined by the Agent App plan.
- Each `documents[].upload_ids` list must satisfy that slot's required document count.

## Supported File Types

Agent App tasks currently accept PDF-like uploaded files:

- `PDF`
- `DOC`
- `DOCX`

## Task Management

### 1. Create Task

Create a new Agent App task for a published app.

**Endpoint:** `POST /agent/apps/tasks`

**Request:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `app_id` | string | Yes | Public Agent App ID |
| `documents` | array[object] | Yes | Uploaded documents grouped by Agent App document slot |
| `documents[].label` | string | Yes | Document slot label from the Agent App plan |
| `documents[].upload_ids` | array[string] | Yes | Upload IDs for this slot |

**Response:**

| Field | Type | Nullable | Description |
|-------|------|----------|-------------|
| `id` | string | No | Task ID |
| `app_id` | string | No | Public Agent App ID |
| `status` | string | No | Task status: `pending`, `processing`, `success`, `failed` |
| `uploads` | array | No | Associated uploads |
| `uploads[].id` | string | No | Upload ID |
| `uploads[].name` | string | No | Upload file name |
| `uploads[].file_type` | string | No | Upload file type |
| `uploads[].label` | string | Yes | Document slot label this upload belongs to |
| `created_at` | integer | No | Creation timestamp |
| `updated_at` | integer | No | Update timestamp |
| `meta` | object | No | Task metadata. Currently only exposes `error` when present |

**Response Example:**

```json
{
"type": "System",
"code": "success",
"data":{
  "id": "task_abc123",
  "app_id": "agent_app_xyz",
  "status": "pending",
  "uploads": [
    {
      "id": "F1CMSW",
      "name": "contract.pdf",
      "file_type": "pdf",
      "label": "文档"
    }
  ],
  "created_at": 1757320181,
  "updated_at": 1757320181,
  "meta": {}
},
 "detail": "string"
}
```

**Status Codes:**

| HTTP Code | Error Code | Description |
|-----------|------------|-------------|
| 201 | - | Success |
| 400 | `no_published_version` | The Agent App is not published |
| 400 | `unknown_document_label` | A document group label does not match any Agent App document slot |
| 400 | `insufficient_document_count` | A required document slot is missing or has too few uploaded documents |
| 400 | `too_many_document_count` | A fixed-count document slot has too many uploaded documents |
| 400 | `maximum_source_count_exceeded` | An unlimited-count document slot exceeds the maximum allowed upload count |
| 400 | `unsupported_file_type` | The uploaded file is not PDF/DOC/DOCX-like |
| 400 | `task_upload_parse_failed` | The uploaded file is already in a failed parse state |
| 402 | - | Insufficient credits |
| 403 | `not_authorized` / `no_permission` | The API key cannot access the app or upload |
| 404 | `not_found` | Agent App not found |
| 404 | `upload_ids_not_found` | Upload ID not found |
| 404 | `plan_not_found` | The Agent App has no saved plan |

### 2. Get Task

Retrieve task status by task ID.

**Endpoint:** `GET /agent/apps/tasks/{task_id}`

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `task_id` | string | Task ID |

**Response:** Same schema as Create Task
```json
{
  "type": "System",
  "code": "success",
  "data": {
    "id": "string",
    "app_id": "string",
    "status": "pending",
    "uploads": [
      {
        "id": "string",
        "name": "string",
        "file_type": "string",
        "label": "文档"
      }
    ],
    "created_at": 1757320181,
    "updated_at": 1757320181,
    "meta": {}
  },
  "detail": "string"
}
```

**Task Status Values:**

| Status | Description |
|--------|-------------|
| `pending` | Task has been created and is waiting to run |
| `processing` | Task is currently running |
| `success` | Task completed successfully |
| `failed` | Task finished with an error |

**Status Codes:**

| HTTP Code | Error Code | Description |
|-----------|------------|-------------|
| 200 | - | Success |
| 404 | `task_not_found` | Task not found |

### 3. Get Task Result

Retrieve the final text result of a completed task.

**Endpoint:** `GET /agent/apps/tasks/{task_id}/result`

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `task_id` | string | Task ID |

**Response:** Final result text

```json
{
  "type": "System",
  "code": "success",
  "data": "Here is the final analysis produced by the Agent App.",
  "detail": null
}
```

**Status Codes:**

| HTTP Code | Error Code | Description |
|-----------|------------|-------------|
| 200 | - | Success |
| 400 | `task_invalid_status` | Task is not finished yet |
| 400 | `task_failed` | Task failed, so no final result is available |
| 404 | `task_not_found` | Task not found |

## Recommended Workflow

1. Upload the required source files with the Uploads API.
2. Create a task with the Agent App `app_id` and `documents` grouped by slot label.
3. Poll `GET /agent/apps/tasks/{task_id}` until status becomes `success` or `failed`.
4. If status is `success`, call `GET /agent/apps/tasks/{task_id}/result`.

## Additional Notes

1. **Published App Required**: The target Agent App must already be published.
2. **Plan Required**: The Agent App must already have a saved plan, or task creation will fail.
3. **Document Slots**: Uploads must be grouped by the slot labels generated in the Agent App plan.
4. **Automatic Processing**: If the uploaded file has not started parsing yet, task creation can trigger the required background processing automatically.
5. **Failed Uploads Are Rejected**: Files already in a failed parse state cannot be used to create tasks.
6. **Polling Required**: Do not call the result endpoint immediately after task creation. Poll task status first.
