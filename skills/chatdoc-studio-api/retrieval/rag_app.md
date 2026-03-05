# Content Retrieval App API (RAG App)

The Content Retrieval App API enables semantic search and content retrieval from your documents.

## Base Path

```
/rag/apps
```

## App Management

### 1. Create App

Create a new content retrieval application.

**Endpoint:** `POST /rag/apps/`

**Request:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | App name (1-30 characters) |
| `sources` | array | Yes | Array of `{"id": "upload_id"}` objects (max 50) |

**Response:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | App ID |
| `name` | string | App name |
| `documents` | array | Connected documents |

**Document Object:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Document ID |
| `name` | string | Document name |
| `status` | integer | Processing status |

**Status Codes:**

| HTTP Code | Error Code | Description |
|-----------|------------|-------------|
| 200 | - | Success |
| 400 | `unsupported_file_type` | Unsupported file type (only PDF, DOC, DOCX) |
| 400 | `contain_not_parsed_file` | Contains documents not yet parsed |
| 402 | - | Insufficient credits |

### 2. Get App

Retrieve app details.

**Endpoint:** `GET /rag/apps/{app_id}`

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `app_id` | string | App ID |

**Response:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | App ID |
| `name` | string | App name |
| `documents` | array | Connected documents |

**Status Codes:**

| HTTP Code | Error Code | Description |
|-----------|------------|-------------|
| 200 | - | Success |
| 404 | `not_found` | App not found |

### 3. Update App

Update app configuration and documents.

**Endpoint:** `PUT /rag/apps/{app_id}`

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `app_id` | string | App ID |

**Request:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | App name (1-30 characters) |
| `sources` | array | Yes | Array of `{"id": "upload_id"}` objects (max 50) |

**Response:** Updated app object

**Status Codes:**

| HTTP Code | Error Code | Description |
|-----------|------------|-------------|
| 200 | - | Success |
| 400 | `already_published` | Cannot update published app |
| 400 | `unsupported_file_type` | Unsupported file type |
| 400 | `contain_not_parsed_file` | Contains documents not yet parsed |
| 404 | `not_found` | App not found |

## Content Retrieval

### 4. Retrieval Query

Execute a semantic search query to retrieve relevant content.

**Endpoint:** `POST /rag/apps/{app_id}/retrieval`

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `app_id` | string | App ID |

**Request:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `query` | string | Yes | Search query (1-3000 characters) |
| `retrieval_token_length` | integer | No | Max response token length (1000-30000, default: 6000) |
| `retrieval_mode` | string | No | `basic`, `contextual` (default), or `expanded` |

**Retrieval Modes:**

| Mode | Description |
|------|-------------|
| `basic` | Fast and efficient. Combines Embedding and BM25 hybrid retrieval, followed by a non-contextual reranker to reorder the results |
| `contextual` | More precise. Combines Embedding and BM25 hybrid retrieval, followed by a contextual reranker to reorder the results for better accuracy |
| `expanded` | More comprehensive and highly accurate, with increased latency. After the initial contextual reranking, surrounding context from the top relevant segments is added to the candidate set for a second round of reranking |

**Response:** Array of retrieval results

| Field | Type | Description |
|-------|------|-------------|
| `upload_id` | string | Source document ID |
| `document_name` | string | Source document name |
| `elements` | array | Retrieved content elements |

**Element Object:**

| Field | Type | Description |
|-------|------|-------------|
| `type` | string | Element type: `paragraph`, `table`, etc. |
| `content` | object | Element content |
| `page` | array[int] | Page numbers |
| `index` | integer | Element index |
| `markdown` | string | Markdown representation |

**Status Codes:**

| HTTP Code | Error Code | Description |
|-----------|------------|-------------|
| 200 | - | Success |
| 400 | `invalid_retrieval_mode` | Invalid retrieval mode |
| 400 | `retrieval_failed` | Retrieval operation failed |
| 404 | `not_found` | App not found |
| 409 | `training` | App is still training |

## Important Notes

1. **Document Status**: All connected documents must have `status == 300` (ELEMENT_PARSED) before using retrieval
2. **Supported File Types**: Only PDF, DOC, and DOCX files are supported for content retrieval apps
3. **Training Time**: After creating/updating an app, there may be a brief training period before retrieval is available
4. **Query Length**: Maximum query length is 3000 characters
5. **Token Length**: Adjust `retrieval_token_length` based on your needs (default 6000, range 1000-30000)
6. **Retrieval Modes**: Choose based on your needs - `basic` for speed, `contextual` for accuracy, `expanded` for comprehensive results
