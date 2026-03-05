# ChatDOC Studio Skills

This repository is used to publish and maintain reusable AI skills for ChatDOC Studio.

## Available Skills

### `chatdoc-studio-api`

- Path: `skills/chatdoc-studio-api`
- Description: A complete ChatDOC Studio API guide with multilingual examples (Python / TypeScript / Rust / cURL)
- Covered modules:
  - Uploads API
  - PDF Parser API
  - Chat App API
  - RAG App API
  - Extract App API
  - Apps API
  - Document Status reference

## Quick Start

Use this skill in any AI tool/agent framework that supports skill-style instructions or reusable prompt modules.

In codex, you can trigger it like this:

```text
Use $chatdoc-studio-api to help me build a ChatDOC Studio chat app.
```

You can also use natural language prompts (for example, "Show me a Python example for the Uploads API"), and your assistant can apply this skill's guidance.

## Repository Structure

```text
skills/
  chatdoc-studio-api/
    SKILL.md
    agents/openai.yaml
    uploads/
    parsers/
    chat/
    retrieval/
    extraction/
    apps/
    docs/
```

## Maintenance Notes

- Keep the skill entry instructions in `skills/chatdoc-studio-api/SKILL.md`
- Keep UI metadata in `skills/chatdoc-studio-api/agents/openai.yaml`
- Keep each `*_examples.md` file aligned with its corresponding API documentation
