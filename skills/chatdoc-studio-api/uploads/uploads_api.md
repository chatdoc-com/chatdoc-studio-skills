# Uploads API

The Uploads API enables you to upload documents to your team. These uploaded files can then be referenced when creating or updating Chat Apps, RAG Apps, and Extract Apps.

**Important**: This API is required for all apps except PDF Parser (which has its own upload endpoint). You must upload files first and use the returned `upload_id` when creating/updating apps.

## Base Path

```
/uploads
```

## Upload Document

### Upload File

Upload a document file to your team.

**Endpoint:** `POST /uploads/`

**Request:** (multipart/form-data)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `file` | File | Yes | Document file (PDF, DOC, DOCX, MD, TXT) |

**Supported File Types:**

| File Type | Extensions | MIME Types |
|-----------|------------|------------|
| PDF | `.pdf` | `application/pdf` |
| DOC | `.doc` | `application/msword` |
| DOCX | `.docx` | `application/vnd.openxmlformats-officedocument.wordprocessingml.document` |
| Markdown | `.md` | `application/markdown` |
| Text | `.txt` | `application/txt` |

**Response:**

| Field | Type | Description |
|-------|------|-------------|
| `upload_id` | string | Upload ID - save this for app creation |
| `name` | string | File name |
| `file_type` | string | File type (pdf, docx, md, txt, etc.) |
| `status` | string | Document processing status: `chunking`, `indexed`, `failed` |
| `created_at` | integer | Upload timestamp |

**Document Status After Upload:**

| Status | Description |
|--------|-------------|
| `chunking` | File uploaded, not yet parsed (this is the initial status) |

**Important**: This upload endpoint does **NOT** trigger document parsing. The file will remain at status `chunking` after upload.

**When does parsing happen?**

Parsing is automatically triggered when the `upload_id` is referenced in an app:
- When creating a Chat App with the document
- When creating/updating a RAG App with the document
- When uploading a file to an Extract App

If the document status is not `indexed` when referenced in an app, the system will automatically start parsing it.

**Status Codes:**

| HTTP Code | Error Code | Description |
|-----------|------------|-------------|
| 200 | - | Success |
| 400 | `not_support_file_format` | Unsupported file type |
| 400 | `file_too_large` | File size exceeds limit |
| 400 | `empty_file` | File is empty |
| 402 | - | Insufficient credits |
| 413 | - | Payload too large |

## Usage Flow

1. **Upload file**: Call `POST /uploads/` with your file
2. **Get upload_id**: Save the `upload_id` field from response (file is at status `chunking`)
3. **Use in app**: Reference the `upload_id` when creating/updating apps
4. **Automatic parsing**: The system will automatically trigger parsing when the document is referenced in an app (if not already parsed)
5. **Wait for app ready**: After creating the app, wait for all documents to reach status `indexed` before using the app

**Simplified workflow**: Since parsing is automatic when referenced in apps, you don't need to manually trigger parsing or wait before creating apps. Just upload, get the `upload_id`, and use it immediately in your app creation.

## Important Notes

1. **No parsing on upload**: This endpoint only uploads the file; it does NOT trigger parsing
2. **Automatic parsing**: Parsing starts automatically when the `upload_id` is referenced in an app
3. **Store upload_id**: The API doesn't provide a list-all-documents endpoint, so save the `upload_id` in your database
4. **App-ready status**: Documents must be fully processed (`status == "indexed"`) before using app features (chat, retrieval, extraction)
5. **File size limits**: Check your plan for maximum file size limits
6. **Credit consumption**: Each upload consumes credits based on file size and page count
7. **Multiple files**: Upload multiple files separately to get multiple `upload_id` values

## Integration with App APIs

### Chat App

```json
{
  "name": "My Chat App",
  "prompt": "Answer questions about the documents",
  "sources": [
    {"id": "UPLOAD_ID_1"},
    {"id": "UPLOAD_ID_2"}
  ]
}
```

### RAG App

```json
{
  "name": "My RAG App",
  "sources": [
    {"id": "UPLOAD_ID_1"},
    {"id": "UPLOAD_ID_2"}
  ]
}
```

## File Processing Timeline

1. **Upload (immediate)**: File uploaded, `upload_id` returned (status: `chunking`)
2. **Reference in app**: When you use the `upload_id` in an app, parsing is triggered
3. **Processing (seconds to minutes)**: File is parsed (status: `chunking`)
4. **Ready**: File ready for app features (status: `indexed`)

The processing time depends on file size and type:
- Small PDFs (< 10 pages): ~30 seconds
- Medium PDFs (10-50 pages): ~1-2 minutes
- Large PDFs (50+ pages): ~3+ minutes

**Key Point**: You can create your app immediately after upload. The system handles parsing in the background. Just check the app status or document status before using chat/retrieval/extraction features.

## Related Documentation

- [Document Status](../docs/document_status.md) - Understanding document processing states
- [Chat App API](../chat/chat_app.md) - Creating chat applications with documents
- [RAG App API](../retrieval/rag_app.md) - Creating retrieval applications with documents
- [Extract App API](../extraction/extract_app.md) - Creating extraction applications
