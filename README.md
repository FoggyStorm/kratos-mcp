# Kratos‑MCP — Memory System for AI Coding Tools with Project Isolation

[![Releases](https://img.shields.io/github/v/release/FoggyStorm/kratos-mcp?label=Releases&color=blue)](https://github.com/FoggyStorm/kratos-mcp/releases)

![kratos-mcp-banner](https://raw.githubusercontent.com/github/explore/main/topics/ai/ai.png)

Fast, reliable memory for coding assistants. Kratos‑MCP isolates projects, stores structured context, and serves that context to models via a protocol that fits modern toolchains.

Topics: ai-development · ai-tools · claude · coding-assistant · context-management · cursor · developer-tools · kratos · llm · mcp · memory-management · model-context-protocol · prompt-engineering · sqlite · typescript

Releases: download the build from the Releases page and run the release binary or installer. Get the asset at https://github.com/FoggyStorm/kratos-mcp/releases and execute the downloaded file.

---

## Hero badges

[![Build Status](https://img.shields.io/github/actions/workflow/status/FoggyStorm/kratos-mcp/ci.yml?branch=main)](https://github.com/FoggyStorm/kratos-mcp/actions) [![License](https://img.shields.io/github/license/FoggyStorm/kratos-mcp)](https://github.com/FoggyStorm/kratos-mcp/blob/main/LICENSE) [![GitHub stars](https://img.shields.io/github/stars/FoggyStorm/kratos-mcp?style=social)](https://github.com/FoggyStorm/kratos-mcp/stargazers)

---

## What Kratos‑MCP does

- Store and retrieve codebase context and runtime signals.
- Keep project data isolated per workspace.
- Serve context to language models via a compact protocol.
- Let coding assistants maintain a long-lived, searchable memory.
- Track change history and map snippets back to files and line ranges.

Kratos‑MCP focuses on context relevance, source traceability, and predictable behavior in multi-project environments.

---

## Key concepts

- MCP (Model Context Protocol): A small JSON/HTTP protocol for context requests and responses. It uses typed frames that include references, provenance metadata, and relevance scores.
- Project isolation: Each project runs in its own namespace. Data never mixes across projects by default.
- Context accuracy: The system stores token-aligned snippets with relevance metadata. It ranks candidates by context score.
- Quad‑pillar framework: Four core services that form the memory pipeline:
  1. Ingest — capture code, comments, and runtime traces.
  2. Index — embed and index content for fast retrieval.
  3. Serve — resolve context frames for model queries.
  4. Audit — keep provenance, versions, and access logs.

---

## Architecture (high level)

![architecture](https://raw.githubusercontent.com/github/explore/main/topics/machine-learning/machine-learning.png)

- Frontend SDKs (TypeScript) instrument editors and agents.
- Local MCP server (TypeScript / Node) handles ingest, index, and serve.
- Storage backend uses SQLite for local deployments. Use a networked DB for scale.
- Vector index layer stores embeddings for similarity searches.
- Protocol layer exposes REST/gRPC endpoints that return MCP frames.

---

## Features

- Per-project namespaces and access controls.
- Context frames with provenance and file/line references.
- Embedding support for multiple LLM providers.
- Snapshot and rollback of memory state.
- Fielded queries: filter by file, tag, and time range.
- CLI for quick local operations.
- TypeScript SDK for IDE and agent integration.
- Pluggable index backends (SQLite, Redis, FAISS adapters).
- Audit trail: who asked, what was served, and why.

---

## Quickstart — local dev

1. Clone repo
   - git clone https://github.com/FoggyStorm/kratos-mcp.git
2. Install
   - cd kratos-mcp
   - npm install
3. Build
   - npm run build
4. Run server
   - npm start

Or download a release build from Releases, extract, and execute the binary. The Releases page contains ready-to-run builds for common platforms. Visit the releases page and run the downloaded asset:
https://github.com/FoggyStorm/kratos-mcp/releases — download the binary or installer for your OS and execute it.

Example run (default port 8088):
- ./kratos-mcp --data ./data --port 8088

The server will expose the MCP endpoints on the configured port.

---

## Install from Releases

Use the Releases page to grab a prebuilt artifact. Choose the file that matches your OS and architecture, then run the file.

Example:
- Linux: tar xzf kratos-mcp-linux-x64.tar.gz && ./kratos-mcp
- macOS: tar xzf kratos-mcp-darwin-x64.tar.gz && ./kratos-mcp
- Windows: unzip kratos-mcp-win-x64.zip && kratos-mcp.exe

Find builds at the project Releases page:
[Releases · FoggyStorm/kratos-mcp](https://github.com/FoggyStorm/kratos-mcp/releases)

---

## CLI examples

- Start
  - kratos-mcp start --port 8088 --data ./data
- Ingest files
  - kratos-mcp ingest --project my-app --path ./my-app
- Query context
  - kratos-mcp query --project my-app --prompt "How does auth work?"
- Export snapshot
  - kratos-mcp snapshot export --project my-app --out snapshot.json

---

## API (MCP) — Example

POST /mcp/v1/context

Request
{
  "project": "my-app",
  "query": "explain the login flow",
  "k": 8,
  "filters": { "path": ["src/auth/**"] }
}

Response
{
  "frames": [
    {
      "id": "frame-123",
      "content": "function login(user, pass) { ... }",
      "source": { "file": "src/auth/login.js", "start": 12, "end": 38 },
      "score": 0.92,
      "provenance": { "commit": "ae4f2a" }
    }
  ],
  "meta": { "took_ms": 23 }
}

Frames return both the content and the source pointer. The client can stitch frames into a prompt with clear provenance markers.

---

## SDK (TypeScript) usage

Install:
- npm install @foggystorm/kratos-mcp-client

Basic snippet:
import { KratosClient } from '@foggystorm/kratos-mcp-client'

const client = new KratosClient({ baseUrl: 'http://localhost:8088' })
const resp = await client.context({
  project: 'my-app',
  query: 'refactor the payment module',
  k: 6
})

resp.frames.forEach(f => console.log(f.source.file, f.score))

Use the SDK in editor extensions and automated agents. The SDK exposes typed requests and response models.

---

## Indexing and embeddings

- The server supports multiple embedding providers.
- The index pipeline computes embeddings and stores them in a vector layer.
- You can plug in your own vector store adapter.
- The default setup uses SQLite + a compact vector index for low friction.

Configuration example (config.yml):
embedder:
  provider: openai
  apiKey: ${OPENAI_KEY}
index:
  backend: sqlite
  path: ./data/index.db

---

## Provenance and audit

Every stored snippet includes:
- source path and range
- commit or version id
- ingestion timestamp
- origin (agent or user)
- confidence score

The audit log records:
- request id
- requester id
- frames served
- timestamps

Use the audit data to trace answers back to source code and to tune relevance.

---

## Integrations

- IDE plugins (VS Code, JetBrains)
- Chat assistant adapters (Claude, OpenAI, local LLMs)
- CI hooks to ingest commits during pipelines
- Cursor-style agents and custom shells
- Webhooks for external events

Example: configure VS Code extension to call the local MCP server on file save. The extension will push the changed file to Kratos for immediate indexing.

---

## Performance and scaling

- Local mode runs well for single developers.
- For teams, run a networked instance with a scaled index backend.
- Use a managed vector DB for large corpora.
- Use partitioning by project to keep queries fast.

---

## Security model

- Projects map to namespaces.
- Access tokens restrict endpoints per project.
- Audit trails record access events.
- You can encrypt the data store at rest.

---

## Contributing

- Fork the repo.
- Create a feature branch.
- Run tests and linters: npm test
- Open a pull request and describe the change.

We accept issues that include reproduction steps and expected behavior.

---

## Roadmap

- Multi-tenant cloud mode
- Additional vector backends (FAISS, Milvus)
- Notebook integration and runtime traces
- Native desktop agent for macOS and Windows
- More SDKs (Python, Go)

---

## FAQ

Q: How does Kratos match context?
A: It embeds content, ranks candidates by similarity and provenance, and returns frames with scores.

Q: Can I keep data local?
A: Yes. The default setup uses local storage and runs on a single host.

Q: How do I update a release?
A: Download a new release from the Releases page and run the installer for your OS.

Find builds and installers at:
https://github.com/FoggyStorm/kratos-mcp/releases

---

## Examples and recipes

- Create a CI job that ingests diffs on each merge and tags frames with commit ids.
- Add a VS Code command to fetch top 5 frames for the current selection.
- Add a pre-push hook to snapshot memory state.

---

## License

See LICENSE in the repo for full terms.

---

Images used
- AI topic icons from GitHub Explore
- Machine learning graphic from GitHub Explore

Changelog and builds live on the Releases page. Visit it to download assets and run the release file:
https://github.com/FoggyStorm/kratos-mcp/releases