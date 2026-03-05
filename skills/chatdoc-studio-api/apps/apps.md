# Apps API

The Apps API allows you to manage all types of applications (Chat, Extract, RAG) in your team.

## Base Path

```
/apps
```

## App Types

| Type | Name | Description |
|------|------|-------------|
| `1` | ChatApp | Document-based chat application |
| `2` | ExtractApp | Structured data extraction application |
| `5` | ContentRetrievalApp | RAG (Retrieval-Augmented Generation) application |

**Note**: PDF Parser (type 99) is not an app and is not managed by this API.

### 1. Get Apps

Retrieve a paginated list of all applications in your team.

**Endpoint:** `GET /apps/`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `currentPage` | integer | No | Page number (default: 1, min: 1) |
| `pageSize` | integer | No | Items per page (default: 20, min: 1) |

**Response:**

| Field | Type | Description |
|-------|------|-------------|
| `items` | array | Array of app objects |
| `page` | integer | Current page number |
| `size` | integer | Items per page |
| `total` | integer | Total number of apps |

**App Object:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | App ID (alias_id) |
| `name` | string | App name |
| `app_type` | integer | App type (1=Chat, 2=Extract, 5=RAG) |
| `created_at` | integer | Creation timestamp (Unix timestamp) |
| `updated_at` | integer | Last update timestamp (Unix timestamp) |

**Status Codes:**

| HTTP Code | Error Code | Description |
|-----------|------------|-------------|
| 200 | - | Success |

### 2. Delete App

Delete an application by app_id. This operation permanently removes the app and all its associated data.

**Endpoint:** `DELETE /apps/{app_id}`

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `app_id` | string | App ID (alias_id) |

**Response:** Empty data field on success

**Status Codes:**

| HTTP Code | Error Code | Description |
|-----------|------------|-------------|
| 200 | - | Success |
| 404 | `not_found` | App not found |

**Important Notes:**

1. **App ID**: Use the `id` field (alias_id) from the Get Apps response. This is the public app identifier.

2. **Deletion Behavior**:
   - For Chat/RAG apps: Deletes all app versions and associated conversations
   - For Extract apps: Soft deletes all app versions with the same alias_id

3. **Irreversible**: Deletion is permanent. Make sure you have backups if needed.

4. **App Types**: This API works for Chat (1), Extract (2), and RAG (5) apps. PDF Parser (99) is not supported.

## Important Notes

1. **Pagination**: Use `currentPage` and `pageSize` query parameters to navigate through large lists of apps.

2. **Timestamp Format**: All timestamps are Unix timestamps (seconds since epoch).

3. **App Identification**: The `id` field in the response is the alias_id - use this for all subsequent API calls.

4. **Type Filtering**: The `app_type` field helps you distinguish between different app types when processing the response.
