---
name: chatdoc-studio-api
description: ChatDOC Studio API usage guide - complete documentation and examples for PDF parsing, chat applications, content retrieval, and data extraction APIs
---

## Overview

ChatDOC Studio is an AI-powered document processing and conversation platform providing multiple API capabilities:

- **PDF Parser** - Parse PDF documents into structured data (JSON, Markdown, Excel)
- **Chat App** - Create document-based Q&A chat applications
- **RAG App** - Content retrieval applications based on documents
- **Extract App** - Extract structured data from documents

## API Basics

### Base URL

```
https://api.chatdoc.studio/v1
```

### Authentication

All API requests require a JWT Token in the HTTP Header:

```
Authorization: Bearer YOUR_API_KEY
```

### Environment Variables

Manage API configuration through environment variables:

| Environment Variable | Description | Default Value |
|---------------------|-------------|---------------|
| `CHATDOC_STUDIO_BASE_URL` | API Base URL | `https://api.chatdoc.studio/v1` |
| `CHATDOC_STUDIO_API_KEY` | API authentication key | - |

### Supported File Types

| API | PDF | DOC | DOCX | MD | TXT |
|-----|-----|-----|------|----|-----|
| PDF Parser | ✓ | ✗ | ✗ | ✗ | ✗ |
| Chat App | ✓ | ✓ | ✓ | ✓ | ✓ |
| RAG App | ✓ | ✓ | ✓ | ✗ | ✗ |
| Extract App | ✓ | ✓ | ✓ | ✗ | ✗ |

## API Module Documentation

### Uploads API

**Required for all apps except PDF Parser**. Upload documents to your team before using them in Chat Apps, RAG Apps, or Extract Apps.

Documentation: [uploads/uploads_api.md](uploads/uploads_api.md)
Code Examples: [uploads/uploads_api_examples.md](uploads/uploads_api_examples.md)

### PDF Parser API

Parse PDF documents into structured data, supporting JSON, Markdown, and Excel exports.

Documentation: [parsers/pdf_parser.md](parsers/pdf_parser.md)
Code Examples: [parsers/pdf_parser_examples.md](parsers/pdf_parser_examples.md)

### Chat App API

Create document-based Q&A chat applications with multi-turn conversations and source tracing.

Documentation: [chat/chat_app.md](chat/chat_app.md)
Code Examples: [chat/chat_app_examples.md](chat/chat_app_examples.md)

### RAG App API

Perform semantic retrieval based on document content to retrieve relevant document fragments.

Documentation: [retrieval/rag_app.md](retrieval/rag_app.md)
Code Examples: [retrieval/rag_app_examples.md](retrieval/rag_app_examples.md)

### Extract App API

Extract structured data from documents based on JSON Schema definitions.

Documentation: [extraction/extract_app.md](extraction/extract_app.md)
Code Examples: [extraction/extract_app_examples.md](extraction/extract_app_examples.md)

### Apps API

Manage all types of applications (Chat, Extract, RAG) in your team - list and delete apps.

Documentation: [apps/apps.md](apps/apps.md)
Code Examples: [apps/apps_examples.md](apps/apps_examples.md)

## Document Status (DocumentStatus)

All uploaded documents go through a processing status flow. Understanding document status is crucial for proper API usage.

Documentation: [docs/document_status.md](docs/document_status.md)

## Common Response Format

All API responses follow a unified format:

Success Response:
```json
{
  "type": "System",
  "code": "success",
  "data": { ... },
  "detail": null
}
```

Error Response:
```json
{
  "type": "...",
  "code": "...",
  "data": ...,
  "detail": "...."
}
```

## Common Error Codes

In addition to API-specific error codes, the following error codes may be returned by any API endpoint:

### Plan Error Codes (PlanErrorEnum)

These errors are related to your subscription plan's credit and capacity limits. Your API usage consumes credits and counts against your plan's capacity.

| Error Code | Description |
|------------|-------------|
| `credit_not_enough` | Insufficient credits to perform the operation. Top up your credits or upgrade your plan. |
| `capacity_not_enough` | Storage capacity exceeded. Delete unused documents or upgrade your plan. |
| `app_count_not_enough` | Maximum number of apps allowed by your plan has been reached. |
| `member_count_not_enough` | Maximum number of team members allowed by your plan has been reached. |
| `upgrade_plan_error` | Error occurred during plan upgrade process. |
| `not_found` | Plan not found. Contact support. |

### System Error Codes (SystemErrorEnum)

These are general system-level errors that may occur during API operations.

| Error Code | Description |
|------------|-------------|
| `unknown_error` | An unexpected error occurred. Try again or contact support if it persists. |
| `validation_error` | Request validation failed. Check your request parameters. |
| `project_expired` | The project or subscription has expired. Renew your subscription to continue. |
| `handshake_error` | Authentication handshake failed. Check your API key. |

## Rate Limiting

API calls are subject to rate limits based on your subscription plan. HTTP 429 status code will be returned when limits are exceeded.

## Getting Started

### Basic Workflow

1. Obtain an API Key from the ChatDOC Studio console
2. Configure environment variables
3. **Upload documents** (for Chat/RAG/Extract Apps) using the Uploads API
4. **Immediately create your app** using the upload IDs (parsing is auto-triggered when referenced)
5. Wait for app to be ready before using app features (chat/retrieval/extraction)
6. Review the module's documentation and examples
7. Integrate into your application

### Quick Start Examples

**PDF Parser**: Upload and parse → Get JSON/Markdown/Excel

**Chat App**: Upload documents → Create Chat App → Send messages

**RAG App**: Upload documents → Create RAG App → Query content

**Extract App**: Create Extract App with schema → Upload document → Get extracted data

## Additional Resources

- See [docs/document_status.md](docs/document_status.md) for document processing status details
