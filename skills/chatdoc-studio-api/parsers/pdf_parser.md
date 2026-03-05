# PDF Parser API

The PDF Parser API converts PDF documents into structured data formats (JSON, Markdown, Excel).

## Base Path

```
/pdf/parser
```

## Endpoints

### 1. Upload PDF

Upload a PDF document for parsing.

**Endpoint:** `POST /pdf/parser/upload`

**Request:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `file` | File | Yes | PDF file to upload |
| `wait` | boolean | No | Wait for parsing to complete (default: `false`) |

**Response:**

| Field | Type | Description |
|-------|------|-------------|
| `upload_id` | string | Document upload ID |
| `name` | string | Document name |
| `status` | string | Document processing status: `chunking`, `chunked`, `indexed`, `failed` |
| `markdown` | string | (if `wait=true`) Parsed markdown content |

**Status Codes:**

| Code | Description |
|------|-------------|
| 200 | Success |
| 400 | Invalid file format (only PDF supported) |
| 413 | File too large |
| 415 | Unsupported media type |

### 2. Get JSON

Retrieve parsed document as structured JSON.

**Endpoint:** `GET /pdf/parser/{upload_id}/json`

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `upload_id` | string | Document ID from upload response |

**Response:**

| Field | Type | Description |
|-------|------|-------------|
| `document` | object | Document metadata (id, name, create_time) |
| `elements` | array | Parsed elements (paragraphs, tables, images, etc.) |

**Element Types:**

- `paragraph` - Text paragraphs
- `table` - Tables with grid data
- `figure` - Figures and charts
- `image` - Images
- `shape` - Shapes and drawings
- `page_header` - Page headers
- `page_footer` - Page footers

**Status Codes:**

| Code | Description |
|------|-------------|
| 200 | Success |
| 400 | Document not ready (status is `chunking`) |
| 404 | Document not found |

### 3. Get Markdown

Download parsed document as Markdown file.

**Endpoint:** `GET /pdf/parser/{upload_id}/markdown`

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `upload_id` | string | Document ID from upload response |

**Response:** Markdown file download

**Status Codes:**

| Code | Description |
|------|-------------|
| 200 | Success (file download) |
| 400 | Document not ready (status is `chunking`) |
| 404 | Document not found |

### 4. Get Excel

Download parsed document as Excel file (tables only).

**Endpoint:** `GET /pdf/parser/{upload_id}/excel`

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `upload_id` | string | Document ID from upload response |

**Response:** Excel file (.xlsx) with table data

**Status Codes:**

| Code | Description |
|------|-------------|
| 200 | Success (file download) |
| 400 | Document not ready (status is `chunking`) |
| 404 | Document not found |

## Usage Flow

1. Upload PDF via `POST /pdf/parser/upload`
2. Poll document status until `status == "chunked"` or `status == "indexed"`
3. Call desired endpoint (JSON/Markdown/Excel)

**Important**: Wait for `status` to be `"chunked"` or `"indexed"` before calling JSON/Markdown/Excel endpoints.

## Error Codes

| Error Code | Description |
|------------|-------------|
| `not_support_file_format` | File is not a PDF |
| `not_parsed` | Document parsing not completed |
| `file_too_large` | File exceeds size limit |
| `empty_file` | File is empty |
| `page_count_exceed_limit` | Page count exceeds plan limit |
