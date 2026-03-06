# Chat App API - Code Examples

This page contains code examples for the Chat App API in Python, TypeScript (server-side), Rust, and cURL.

## Environment Setup

Set up your environment variables:

```bash
export CHATDOC_STUDIO_BASE_URL="https://api.chatdoc.studio/v1"
export CHATDOC_STUDIO_API_KEY="your-api-key-here"
```

## Create App

### Python

```python
import os
import requests

BASE_URL = os.getenv("CHATDOC_STUDIO_BASE_URL", "https://api.chatdoc.studio/v1")
API_KEY = os.getenv("CHATDOC_STUDIO_API_KEY")

def create_chat_app(
    name: str,
    instruction: str,
    use_case: str = "customer_service",
    sources: list[dict] | None = None,
    **kwargs
) -> dict:
    """Create a new chat application."""
    url = f"{BASE_URL}/chat/apps/"
    headers = {"Authorization": f"Bearer {API_KEY}"}

    data = {
        "name": name,
        "instruction": instruction,
        "use_case": use_case,
        **kwargs
    }

    if sources is not None:
        data["sources"] = sources

    response = requests.post(url, headers=headers, json=data)
    response.raise_for_status()
    return response.json()["data"]

# Usage - Customer Service
app = create_chat_app(
    name="Customer Support Bot",
    instruction="You are a helpful customer support assistant.",
)
print(f"App ID: {app['id']}")

# Usage - Knowledge Base Q&A
app = create_chat_app(
    name="Document QA",
    instruction="Answer questions based on the provided documents.",
    use_case="knowledge_base_qa",
    sources=[{"id": "F1CMSW"}],  # upload_id from document
    temperature=0.3,
)
print(f"App ID: {app['id']}")
```

### TypeScript (Server-side)

```typescript
import axios from 'axios';

const BASE_URL = process.env.CHATDOC_STUDIO_BASE_URL || 'https://api.chatdoc.studio/v1';
const API_KEY = process.env.CHATDOC_STUDIO_API_KEY || '';

interface CreateAppRequest {
  name: string;
  instruction: string;
  use_case?: string;
  sources?: Array<{ id: string }>;
  temperature?: number;
  welcome_message?: string;
  show_history?: boolean;
  primary_color?: string;
  icon_primary_color?: string;
  position?: number;
  source_traceable?: boolean;
  support_new_conversation?: boolean;
  // ... other fields
}

interface AppResponse {
  id: string;
  name: string;
  published: boolean;
  published_at: number | null;
  team_id: string;
}

async function createChatApp(data: CreateAppRequest): Promise<AppResponse> {
  const response = await axios.post<{ data: AppResponse }>(
    `${BASE_URL}/chat/apps/`,
    data,
    {
      headers: {
        Authorization: `Bearer ${API_KEY}`,
      },
    }
  );
  return response.data.data;
}

// Usage - Customer Service
const app = await createChatApp({
  name: 'Customer Support Bot',
  instruction: 'You are a helpful customer support assistant.',
  use_case: 'customer_service',
  welcome_message: 'Hello! How can I help you today?',
});
console.log(`App ID: ${app.id}`);

// Usage - Knowledge Base Q&A
const kbApp = await createChatApp({
  name: 'Document QA',
  instruction: 'Answer questions based on the provided documents.',
  use_case: 'knowledge_base_qa',
  sources: [{ id: 'F1CMSW' }], // upload_id from document
  temperature: 0.3,
});
console.log(`App ID: ${kbApp.id}`);
```

### Rust

```rust
use reqwest::Client;
use serde::{Deserialize, Serialize};
use serde_json::json;

const BASE_URL: &str = "https://api.chatdoc.studio/v1";

#[derive(Debug, Deserialize)]
struct AppResponse {
    id: String,
    name: String,
    published: bool,
    published_at: Option<i64>,
    team_id: String,
}

#[derive(Debug, Deserialize)]
struct ApiResponse<T> {
    data: T,
}

#[derive(Debug, Serialize)]
struct Source {
    id: String,
}

#[derive(Debug, Serialize)]
struct CreateAppRequest<'a> {
    name: &'a str,
    #[serde(rename = "instruction")]
    instruction: &'a str,
    #[serde(rename = "use_case")]
    use_case: &'a str,
    #[serde(skip_serializing_if = "Option::is_none")]
    sources: Option<Vec<Source>>,
    #[serde(skip_serializing_if = "Option::is_none")]
    temperature: Option<f32>,
}

async fn create_chat_app(
    name: &str,
    instruction: &str,
    use_case: &str,
    sources: Option<Vec<String>>,
) -> Result<AppResponse, Box<dyn std::error::Error>> {
    let api_key = std::env::var("CHATDOC_STUDIO_API_KEY")?;
    let client = Client::new();

    let sources = sources.map(|ids| ids.into_iter()
        .map(|id| Source { id })
        .collect());

    let request = CreateAppRequest {
        name,
        instruction,
        use_case,
        sources,
        temperature: None,
    };

    let response = client
        .post(&format!("{}/chat/apps/", BASE_URL))
        .header("Authorization", format!("Bearer {}", api_key))
        .json(&request)
        .send()
        .await?;

    let api_response: ApiResponse<AppResponse> = response.json().await?;
    Ok(api_response.data)
}
```

### cURL

```bash
# Customer Service App
curl -X POST "${CHATDOC_STUDIO_BASE_URL}/chat/apps/" \
  -H "Authorization: Bearer ${CHATDOC_STUDIO_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Customer Support Bot",
    "instruction": "You are a helpful customer support assistant.",
    "use_case": "customer_service",
    "welcome_message": "Hello! How can I help you today?"
  }'

# Knowledge Base Q&A App
curl -X POST "${CHATDOC_STUDIO_BASE_URL}/chat/apps/" \
  -H "Authorization: Bearer ${CHATDOC_STUDIO_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Document QA",
    "instruction": "Answer questions based on the provided documents.",
    "use_case": "knowledge_base_qa",
    "sources": [{"id": "F1CMSW"}],
    "temperature": 0.3
  }'
```

## Get App

### Python

```python
def get_chat_app(app_id: str) -> list[dict]:
    """Get app details (returns draft and published versions)."""
    url = f"{BASE_URL}/chat/apps/{app_id}"
    headers = {"Authorization": f"Bearer {API_KEY}"}
    response = requests.get(url, headers=headers)
    response.raise_for_status()
    return response.json()["data"]

# Usage
apps = get_chat_app("abc123")
for app in apps:
    print(f"App: {app['name']}, Published: {app['published']}")
```

### TypeScript

```typescript
async function getChatApp(appId: string): Promise<AppResponse[]> {
  const response = await axios.get<{ data: AppResponse[] }>(
    `${BASE_URL}/chat/apps/${appId}`,
    {
      headers: {
        Authorization: `Bearer ${API_KEY}`,
      },
    }
  );
  return response.data.data;
}

// Usage
const apps = await getChatApp('abc123');
for (const app of apps) {
  console.log(`App: ${app.name}, Published: ${app.published}`);
}
```

### cURL

```bash
curl -X GET "${CHATDOC_STUDIO_BASE_URL}/chat/apps/abc123" \
  -H "Authorization: Bearer ${CHATDOC_STUDIO_API_KEY}"
```

## Update App

### Python

```python
def update_chat_app(app_id: str, **kwargs) -> dict:
    """Update the latest draft app."""
    url = f"{BASE_URL}/chat/apps/{app_id}"
    headers = {"Authorization": f"Bearer {API_KEY}"}
    response = requests.put(url, headers=headers, json=kwargs)
    response.raise_for_status()
    return response.json()["data"]

# Usage
app = update_chat_app(
    "abc123",
    name="Updated Name",
    instruction="Updated instruction",
    temperature=0.5,
)
```

### TypeScript

```typescript
async function updateChatApp(
  appId: string,
  updates: Partial<CreateAppRequest>
): Promise<AppResponse> {
  const response = await axios.put<{ data: AppResponse }>(
    `${BASE_URL}/chat/apps/${appId}`,
    updates,
    {
      headers: {
        Authorization: `Bearer ${API_KEY}`,
      },
    }
  );
  return response.data.data;
}

// Usage
const app = await updateChatApp('abc123', {
  name: 'Updated Name',
  instruction: 'Updated instruction',
  temperature: 0.5,
});
```

### cURL

```bash
curl -X PUT "${CHATDOC_STUDIO_BASE_URL}/chat/apps/abc123" \
  -H "Authorization: Bearer ${CHATDOC_STUDIO_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Updated Name",
    "instruction": "Updated instruction",
    "temperature": 0.5
  }'
```

## Publish App

**Important**: Publishing is an async operation when the app contains documents. You need to poll the publish endpoint until it returns 200.

**Status Codes:**

| HTTP Code | Error Code | Description |
|-----------|------------|-------------|
| 200 | - | Successfully published |
| 400 | `already_published` | App is already published (no-op) |
| 400 | `training` | App is still processing, continue polling |
| 404 | `not_found` | App not found |

### Python

```python
import time

def publish_chat_app(
    app_id: str,
    max_retries: int = 60,
    delay: int = 2
) -> None:
    """Publish the latest draft version with polling.

    Args:
        app_id: App ID
        max_retries: Maximum polling attempts
        delay: Delay between attempts (seconds)
    """
    url = f"{BASE_URL}/chat/apps/{app_id}/publish"
    headers = {"Authorization": f"Bearer {API_KEY}"}

    for attempt in range(max_retries):
        response = requests.post(url, headers=headers)

        if response.status_code == 200:
            print(f"✓ App published successfully!")
            return

        if response.status_code == 400:
            error_data = response.json()
            error_code = error_data.get("code")
            if error_code == "already_published":
                print(f"✓ App already published!")
                return
            if error_code == "training":
                print(f"Publishing... ({attempt + 1}/{max_retries})")
                time.sleep(delay)
                continue
            # Other errors - raise
            response.raise_for_status()

        if response.status_code >= 400:
            response.raise_for_status()

        # Still processing, wait and retry
        print(f"Publishing... ({attempt + 1}/{max_retries})")
        time.sleep(delay)

    raise TimeoutError(f"App did not publish within {max_retries * delay} seconds")

# Usage
publish_chat_app("abc123")
```

### TypeScript

```typescript
async function publishChatApp(
  appId: string,
  maxRetries: number = 60,
  delay: number = 2000
): Promise<void> {
  const url = `${BASE_URL}/chat/apps/${appId}/publish`;

  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      const response = await axios.post(url, {}, {
        headers: { Authorization: `Bearer ${API_KEY}` }
      });

      if (response.status === 200) {
        console.log('✓ App published successfully!');
        return;
      }

      // Still processing, wait and retry
      console.log(`Publishing... (${attempt + 1}/${maxRetries})`);
      await new Promise(resolve => setTimeout(resolve, delay));
    } catch (error: unknown) {
      if (axios.isAxiosError(error) && error.response?.status === 400) {
        const errorCode = (error.response.data as { code?: string } | undefined)?.code;
        if (errorCode === 'already_published') {
          console.log('✓ App already published!');
          return;
        }
        if (errorCode === 'training') {
          console.log(`Publishing... (${attempt + 1}/${maxRetries})`);
          await new Promise(resolve => setTimeout(resolve, delay));
          continue;
        }
        throw new Error(`Publish failed: ${errorCode ?? 'unknown_error'}`);
      }

      throw error;
    }
  }

  throw new Error('App did not publish in time');
}

// Usage
await publishChatApp('abc123');
```

### cURL

```bash
# Publish app
curl -X POST "${CHATDOC_STUDIO_BASE_URL}/chat/apps/abc123/publish" \
  -H "Authorization: Bearer ${CHATDOC_STUDIO_API_KEY}"

# Poll for completion (in a script)
for i in {1..30}; do
  RESPONSE=$(curl -s -X POST "${CHATDOC_STUDIO_BASE_URL}/chat/apps/abc123/publish" \
    -H "Authorization: Bearer ${CHATDOC_STUDIO_API_KEY}")

  STATUS=$(echo "$RESPONSE" | jq -r '.code // empty')

  if [ "$STATUS" = "success" ] || [ "$STATUS" = "already_published" ]; then
    echo "✓ App published!"
    break
  fi

  echo "Publishing... ($i/30)"
  sleep 2
done
```

## Create Conversation

### Python

```python
def create_conversation(app_id: str) -> dict:
    """Create a new conversation thread."""
    url = f"{BASE_URL}/chat/apps/{app_id}/conversations"
    headers = {"Authorization": f"Bearer {API_KEY}"}
    response = requests.post(url, headers=headers)
    response.raise_for_status()
    return response.json()["data"]

# Usage
conv = create_conversation("abc123")
print(f"Conversation ID: {conv['id']}")
```

### TypeScript

```typescript
interface Conversation {
  id: string;
  app_id: string;
  name: string;
  created_at: number;
}

async function createConversation(appId: string): Promise<Conversation> {
  const response = await axios.post<{ data: Conversation }>(
    `${BASE_URL}/chat/apps/${appId}/conversations`,
    {},
    {
      headers: {
        Authorization: `Bearer ${API_KEY}`,
      },
    }
  );
  return response.data.data;
}

// Usage
const conv = await createConversation('abc123');
console.log(`Conversation ID: ${conv.id}`);
```

### cURL

```bash
curl -X POST "${CHATDOC_STUDIO_BASE_URL}/chat/apps/abc123/conversations" \
  -H "Authorization: Bearer ${CHATDOC_STUDIO_API_KEY}"
```

## Send Message

### Python

```python
import json
import time

def send_message(
    app_id: str,
    question: str,
    conversation_id: str | None = None,
    stream: bool = False
) -> dict | None:
    """Send a message to a conversation.

    Automatically handles app publishing if needed.
    """
    url = f"{BASE_URL}/chat/apps/{app_id}/messages"
    headers = {"Authorization": f"Bearer {API_KEY}"}

    data = {"question": question}
    if conversation_id:
        data["conversation_id"] = conversation_id

    # Try sending message first
    response = requests.post(url, headers=headers, json=data, params={"stream": stream})

    # Check if app needs to be published
    if response.status_code == 400:
        error_data = response.json()
        if error_data.get("code") == "no_published_version":
            print("App not published. Publishing...")
            publish_app_with_retry(app_id)
            # Retry sending message after publish
            response = requests.post(url, headers=headers, json=data, params={"stream": stream})

    response.raise_for_status()
    return response.json()["data"]

def publish_app_with_retry(app_id: str, max_retries: int = 60, delay: int = 2):
    """Publish app and poll for completion.

    The publish endpoint processes documents in the background.
    You need to poll until it returns 200 (published successfully).
    If you call it again after successful publication, you'll get
    error code `already_published` (400 error). During processing,
    it may return `training` (400 error), which means keep polling.
    """
    url = f"{BASE_URL}/chat/apps/{app_id}/publish"
    headers = {"Authorization": f"Bearer {API_KEY}"}

    for attempt in range(max_retries):
        response = requests.post(url, headers=headers)

        if response.status_code == 200:
            print(f"✓ App published successfully!")
            return

        if response.status_code == 400:
            error_data = response.json()
            error_code = error_data.get("code")

            if error_code == "already_published":
                print(f"✓ App already published!")
                return
            if error_code == "training":
                print(f"Publishing... ({attempt + 1}/{max_retries})")
                time.sleep(delay)
                continue
            # Other errors - raise
            response.raise_for_status()

        if response.status_code >= 400:
            response.raise_for_status()

        # Still processing (202 or other status), wait and retry
        print(f"Publishing... ({attempt + 1}/{max_retries})")
        time.sleep(delay)

    raise TimeoutError(f"App did not publish within {max_retries * delay} seconds")

# Usage - auto-publish flow
result = send_message(
    "abc123",
    "What is the document about?",
)
print(f"Conversation ID: {result['conversation_id']}")
print(f"Answer: {result['answer']}")
```

### TypeScript

```typescript
interface SendMessageRequest {
  conversation_id?: string;
  question: string;
}

interface MessageResponse {
  name: string;
  conversation_id: string;
  answer: string;
}

async function sendMessage(
  appId: string,
  question: string,
  conversationId?: string
): Promise<MessageResponse> {
  const response = await axios.post<{ data: MessageResponse }>(
    `${BASE_URL}/chat/apps/${appId}/messages`,
    {
      conversation_id: conversationId,
      question,
    },
    {
      headers: {
        Authorization: `Bearer ${API_KEY}`,
      },
    }
  );
  return response.data.data;
}

// Usage
const result = await sendMessage(
  'abc123',
  'What is the document about?',
  'conv-id'
);
console.log(`Conversation ID: ${result.conversation_id}`);
console.log(`Name: ${result.name}`);
console.log(`Answer: ${result.answer}`);
```

### cURL

```bash
# Send message with conversation_id
curl -X POST "${CHATDOC_STUDIO_BASE_URL}/chat/apps/abc123/messages" \
  -H "Authorization: Bearer ${CHATDOC_STUDIO_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "conversation_id": "conv-id",
    "question": "What is the document about?"
  }'

# Send message (creates new conversation)
curl -X POST "${CHATDOC_STUDIO_BASE_URL}/chat/apps/abc123/messages" \
  -H "Authorization: Bearer ${CHATDOC_STUDIO_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "question": "Hello!"
  }'

# Send message with streaming
curl -X POST "${CHATDOC_STUDIO_BASE_URL}/chat/apps/abc123/messages?stream=true" \
  -H "Authorization: Bearer ${CHATDOC_STUDIO_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "conversation_id": "conv-id",
    "question": "Tell me more"
  }'
```

### Streaming Response (TypeScript)

```typescript
interface StreamingMessage {
  msg_id: string;
  round_id: string;
  role: string;
  content: string;
  reasoning_content: string | null;
  conversation_id: string;
  name: string;
}

async function sendMessageStream(
  appId: string,
  question: string,
  onChunk: (chunk: StreamingMessage) => void,
  conversationId?: string
): Promise<void> {
  const url = new URL(`${BASE_URL}/chat/apps/${appId}/messages`);
  url.searchParams.append('stream', 'true');

  const response = await fetch(url.toString(), {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${API_KEY}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      conversation_id: conversationId,
      question,
    }),
  });

  const reader = response.body?.getReader();
  const decoder = new TextDecoder();

  if (!reader) {
    throw new Error('No response body');
  }

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    const chunk = decoder.decode(value);
    const lines = chunk.split('\n');

    for (const line of lines) {
      if (line.startsWith('data: ')) {
        const data = line.slice(6);
        if (data.trim()) {
          try {
            const parsed: StreamingMessage = JSON.parse(data);
            onChunk(parsed);
          } catch (e) {
            console.error('Failed to parse SSE data:', e);
          }
        }
      }
    }
  }
}

// Usage
await sendMessageStream(
  'abc123',
  'Tell me more',
  (chunk) => {
    console.log(`Role: ${chunk.role}`);
    console.log(`Content: ${chunk.content}`);
    console.log(`Conversation ID: ${chunk.conversation_id}`);
    console.log(`Name: ${chunk.name}`);
  },
  'conv-id'
);
```

## Get Conversations

### Python

```python
def get_conversations(
    app_id: str,
    start_at: int | None = None,
    end_at: int | None = None
) -> list[dict]:
    """List all conversations for an app."""
    url = f"{BASE_URL}/chat/apps/{app_id}/conversations"
    headers = {"Authorization": f"Bearer {API_KEY}"}

    params = {}
    if start_at:
        params["start_at"] = start_at
    if end_at:
        params["end_at"] = end_at

    response = requests.get(url, headers=headers, params=params)
    response.raise_for_status()
    return response.json()["data"]

# Usage
conversations = get_conversations("abc123")
for conv in conversations:
    print(f"ID: {conv['id']}, Created: {conv['created_at']}")
```

### TypeScript

```typescript
async function getConversations(
  appId: string,
  startAt?: number,
  endAt?: number
): Promise<Conversation[]> {
  const params: Record<string, number> = {};
  if (startAt) params.start_at = startAt;
  if (endAt) params.end_at = endAt;

  const response = await axios.get<{ data: Conversation[] }>(
    `${BASE_URL}/chat/apps/${appId}/conversations`,
    {
      params,
      headers: {
        Authorization: `Bearer ${API_KEY}`,
      },
    }
  );
  return response.data.data;
}

// Usage
const conversations = await getConversations('abc123');
for (const conv of conversations) {
  console.log(`ID: ${conv.id}, Created: ${conv.created_at}`);
}
```

### cURL

```bash
curl -X GET "${CHATDOC_STUDIO_BASE_URL}/chat/apps/abc123/conversations" \
  -H "Authorization: Bearer ${CHATDOC_STUDIO_API_KEY}"
```

## Get Messages

### Python

```python
def get_messages(app_id: str, conversation_id: str) -> dict:
    """Get all messages in a conversation."""
    url = f"{BASE_URL}/chat/apps/{app_id}/conversations/{conversation_id}"
    headers = {"Authorization": f"Bearer {API_KEY}"}
    response = requests.get(url, headers=headers)
    response.raise_for_status()
    return response.json()["data"]

# Usage
messages = get_messages("abc123", "conv-id")
for msg in messages["dialogues"]:
    print(f"{msg['role']}: {msg['content']}")
```

### TypeScript

```typescript
interface Dialogue {
  id: string;
  role: 'user' | 'assistant';
  content: string;
  sources?: any[];
}

interface ConversationDetail {
  id: string;
  dialogues: Dialogue[];
}

async function getMessages(
  appId: string,
  conversationId: string
): Promise<ConversationDetail> {
  const response = await axios.get<{ data: ConversationDetail }>(
    `${BASE_URL}/chat/apps/${appId}/conversations/${conversationId}`,
    {
      headers: {
        Authorization: `Bearer ${API_KEY}`,
      },
    }
  );
  return response.data.data;
}

// Usage
const messages = await getMessages('abc123', 'conv-id');
for (const msg of messages.dialogues) {
  console.log(`${msg.role}: ${msg.content}`);
}
```

### cURL

```bash
curl -X GET "${CHATDOC_STUDIO_BASE_URL}/chat/apps/abc123/conversations/conv-id" \
  -H "Authorization: Bearer ${CHATDOC_STUDIO_API_KEY}"
```
