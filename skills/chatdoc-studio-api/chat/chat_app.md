# Chat App API

The Chat App API allows you to create document-based Q&A chat applications with multi-turn conversations and source tracing.

## Base Path

```
/chat/apps
```

## App Management

### 1. Create App

Create a new chat application.

**Endpoint:** `POST /chat/apps/`

**Request:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | App name (1-30 characters) |
| `instruction` | string | Yes | System prompt/instruction for the AI |
| `use_case` | string | No | Use case: `customer_service` (default) or `knowledge_base_qa` |
| `show_history` | boolean | No | Show conversation history (default: true) |
| `temperature` | float | No | Temperature 0-1 (default: 0.7) |
| `welcome_message` | string | No | Welcome message shown to users |
| `input_placeholder` | string | No | Placeholder text for input field |
| `primary_color` | string | No | Primary color in hex format (default: `#5971ED`) |
| `logo` | string | No | App logo URL |
| `icon` | string | No | Floating icon URL (for customer_service) |
| `icon_type` | string | No | Icon type: `default`, `custom` |
| `icon_primary_color` | string | No | Icon color in hex format (default: `#5971ED`) |
| `position` | integer | No | Icon position: `1` (right) or `3` (left), default: `1` |
| `source_traceable` | boolean | No | Enable source tracing (default: true) |
| `support_new_conversation` | boolean | No | Allow new conversations (default: true) |
| `sources` | array | No* | Array of `{"id": "upload_id"}` objects |
| `embed_model` | string | No | Embedding model |
| `workflow_id` | string | No | Workflow ID to attach |

*Required when `use_case` is `knowledge_base_qa`

**Response:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | App ID |
| `name` | string | App name |
| `published` | boolean | Publication status |
| `published_at` | integer/null | Publication timestamp |
| `team_id` | string | Team ID |

**Status Codes:**

| HTTP Code | Error Code | Description |
|-----------|------------|-------------|
| 200 | - | Success |
| 400 | `upload_ids_not_found` | One or more upload IDs not found |
| 400 | `maximum_document_count_exceeded` | Too many documents connected |
| 400 | `bind_invalid_file_id` | Invalid file ID for binding |
| 402 | - | Insufficient credits |
| 404 | `not_found_knowledge` | Knowledge not found |

### 2. Get App

Retrieve app details by app_id. Returns the latest draft and published versions.

**Endpoint:** `GET /chat/apps/{app_id}`

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `app_id` | string | App ID |

**Response:** Array of app objects (draft and/or published versions)

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | App internal ID |
| `alias_id` | string | App ID (same as app_id in path) |
| `name` | string | App name |
| `published` | boolean | Publication status |
| `uploads` | array | Connected documents |

**Status Codes:**

| HTTP Code | Error Code | Description |
|-----------|------------|-------------|
| 200 | - | Success |
| 404 | `not_found` | App not found |

### 3. Update App

Update the latest draft app.

**Endpoint:** `PUT /chat/apps/{app_id}`

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `app_id` | string | App ID |

**Request:** Same fields as Create App (all optional except when updating `sources`)

**Response:** Updated app object

**Status Codes:**

| HTTP Code | Error Code | Description |
|-----------|------------|-------------|
| 200 | - | Success |
| 400 | `upload_ids_not_found` | One or more upload IDs not found |
| 400 | `maximum_document_count_exceeded` | Too many documents connected |
| 400 | `bind_invalid_file_id` | Invalid file ID for binding |
| 404 | `not_found` | App not found |

### 4. Publish App

Publish the latest draft version.

**Endpoint:** `POST /chat/apps/{app_id}/publish`

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `app_id` | string | App ID |

**Response:** Empty data field on success

**Status Codes:**

| HTTP Code | Error Code | Description |
|-----------|------------|-------------|
| 200 | - | Successfully published |
| 400 | `already_published` | App is already published (no-op) |
| 404 | `not_found` | App not found |

**Important Notes**:
- Publishing is an **async operation** when the app contains documents
- The endpoint processes documents in the background
- You need to **poll** this endpoint until it returns `200` (published successfully)
- If you call publish again after successful publication, you'll get `already_published` error
- An app must be published before you can send messages to it

## Conversation Management

### 5. Create Conversation

Create a new conversation thread for the app.

**Endpoint:** `POST /chat/apps/{app_id}/conversations`

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `app_id` | string | App ID |

**Response:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Conversation ID |
| `app_id` | string | App ID |
| `name` | string | Conversation name |

**Status Codes:**

| HTTP Code | Error Code | Description |
|-----------|------------|-------------|
| 200 | - | Success |
| 400 | `not_ready` | Documents not ready (status != 300) |
| 404 | `not_found` | App not found |

### 6. Get Conversations

List all conversation threads for the app.

**Endpoint:** `GET /chat/apps/{app_id}/conversations`

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `app_id` | string | App ID |

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `start_at` | integer | Start timestamp filter |
| `end_at` | integer | End timestamp filter |

**Response:** Array of conversation objects

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Conversation ID |
| `name` | string | Conversation name |
| `created_at` | integer | Creation timestamp |

**Status Codes:**

| HTTP Code | Error Code | Description |
|-----------|------------|-------------|
| 200 | - | Success |
| 404 | `not_found` | App not found |

### 7. Get Messages

Get all messages in a conversation.

**Endpoint:** `GET /chat/apps/{app_id}/conversations/{conversation_id}`

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `app_id` | string | App ID |
| `conversation_id` | string | Conversation ID |

**Response:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Conversation ID |
| `dialogues` | array | Array of message objects |

**Message Object:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Message ID |
| `role` | string | `user` or `assistant` |
| `content` | string | Message content |
| `sources` | array | Source documents (for assistant messages) |

**Status Codes:**

| HTTP Code | Error Code | Description |
|-----------|------------|-------------|
| 200 | - | Success |
| 400 | `conversation_not_match_app` | Conversation doesn't belong to app |
| 400 | `no_published_version` | No published version of app |
| 404 | `not_found` | App or conversation not found |

### 8. Send Message

Send a message to a conversation.

**Endpoint:** `POST /chat/apps/{app_id}/messages`

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `app_id` | string | App ID |

**Request:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `conversation_id` | string | No | Conversation ID (creates new if omitted) |
| `question` | string | Yes | User message content (max 3000 characters) |

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `stream` | boolean | Enable streaming response (default: false) |

**Response:** (non-streaming)

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Conversation name (summary of the chat) |
| `conversation_id` | string | Conversation ID |
| `answer` | string | AI response content |

**Streaming Response:** Server-Sent Events (SSE) format

Each SSE event contains a JSON object with the following fields:

| Field | Type | Description |
|-------|------|-------------|
| `msg_id` | string | Message ID |
| `round_id` | string | Round ID for this conversation |
| `role` | string | Message role: `system`, `user`, `assistant`, `tool`, `function` |
| `content` | string | Message content |
| `reasoning_content` | string/null | Reasoning content (if available) |
| `conversation_id` | string | Conversation ID |
| `name` | string | App name |

**SSE Format:**
```
data: {"msg_id": "...", "content": "...", ...}

data: {"msg_id": "...", "content": "...", ...}
```

**Status Codes:**

| HTTP Code | Error Code | Description |
|-----------|------------|-------------|
| 200 | - | Success |
| 400 | `no_published_version` | **App must be published before sending messages** |
| 400 | `conversation_not_match_app` | Conversation doesn't belong to app |
| 404 | `not_found` | App or conversation not found |

**Important**: You must publish the app before sending messages. If you try to send messages to an unpublished app, you will receive a `400` error with error code `no_published_version`. Call `POST /chat/apps/{app_id}/publish` first.

## Use Case Behavior

### Customer Service (`customer_service`)

- Floating chat widget on website
- UI customization: icon, color, position
- No document source required
- General purpose chatbot

### Knowledge Base Q&A (`knowledge_base_qa`)

- Document-based Q&A
- Requires at least one document source
- All documents must be parsed (status 300)
- Source tracing enabled by default

## Important Notes

1. **App Publishing Required**: The app MUST be published (`POST /chat/apps/{app_id}/publish`) before sending messages. Attempting to send messages to an unpublished app will return error code `no_published_version` (400 error).

2. **App ID**: Use the `id` field from Create App response for all subsequent API operations.

3. **Publishing Flow**: Create/Update operations affect the draft version. You must explicitly publish to make changes live for end users.

4. **Conversation Flow**:
   - You can send messages without creating a conversation first (one will be created automatically)
   - For multi-turn conversations, pass the `conversation_id` from the first response
   - Each conversation is independent and maintains its own context

5. **Streaming**: Set `stream=true` query parameter for real-time streaming responses via Server-Sent Events (SSE)
