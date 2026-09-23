# Architecture: n8n ↔ local Codex gateway

## Responsibility boundary

The project separates two systems:

- **n8n** controls the larger content-production workflow: metadata and scene-related steps, calls to other providers, stock/media processing, and FFmpeg-connected rendering.
- **The local gateway** provides Codex-backed text, structured JSON, research and image-generation capabilities, with authenticated access and job/artifact visibility.

The gateway does **not** independently perform final video assembly, ingest every content provider, or replace all n8n nodes.

## Data flow (illustrative)

```mermaid
sequenceDiagram
    participant N as n8n
    participant G as FastAPI Gateway
    participant Q as Worker / Queue
    participant C as Codex CLI
    participant O as n8n downstream nodes
    N->>G: Authenticated request + task input
    G->>G: Validate body and limits
    G->>Q: Accept / queue or return bounded error
    Q->>C: Execute task under configured local account
    C-->>G: Text/JSON or generated image
    G->>G: Validate result; record status/usage/artifact
    G-->>N: Structured response or image bytes
    N->>O: Parse, check, then continue media workflow
```

## Interface design

The private implementation exposes separate interfaces for ordinary text or JSON generation, research, chat-shaped compatibility and images. The compatibility route preserves familiar downstream parsing such as `choices[0].message.content` rather than requiring all workflow nodes to be rewritten at once. The image route provides file/binary output. Text and image outputs are not interchangeable: the n8n HTTP Request node must use the appropriate response mode.

The public illustrative request shapes are in [`../examples/illustrative-requests.json`](../examples/illustrative-requests.json). They omit endpoint hosts and credentials.

## Observability and state

Jobs are tracked through states such as queued, running, completed and failed. The local dashboard supports operational checks such as health, progress, usage, diagnostics and managed image artifacts. Persistent records are separate from the process's in-memory queue; a local dashboard is **not** a guarantee of durable execution after a machine restart.

## Reference images

An image request can provide selected HTTPS references. The gateway validates downloads and passes suitable local files to Codex. An opt-in cache limits duplicate network downloads for identical sources, subject to freshness, validation, leases and eviction policy. Model input cost must be measured separately; cache hits do not eliminate the model's image input.

## Optional episode sessions

An optional episode-title key can associate subsequent tasks with a resumable Codex session. Requests for one episode are serialized in a single gateway process; separate episode titles can remain isolated. A failed/interrupted resumed turn is treated as uncertain rather than blindly reused. This is a feature-gated implementation, not a claim of a live multi-worker or cross-machine distributed session system.

## Threat model and deployment limits

The gateway is a trusted-local-network tool. Bearer authentication is not a substitute for TLS or host isolation. A signed-in CLI can access resources available to its Windows user subject to the process's configured restrictions; a read-only sandbox is not full OS isolation. The portfolio excludes private network details, credentials and live endpoint instructions.
