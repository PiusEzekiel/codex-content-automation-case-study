# Reliability: engineering decisions and honest limitations

## Reliability work demonstrated by the gateway

| Failure or operational concern | Design response | Limit |
| --- | --- | --- |
| Downstream nodes expect a stable text/JSON shape | Compatibility-oriented response envelope, JSON/schema checks | Valid JSON can still be semantically wrong; n8n retains task-level validation |
| Excess simultaneous work | Bounded queue, worker/concurrency controls, explicit busy responses | In-process queue does not survive an application restart |
| Long or interrupted executions | Timeouts, failed job state, conservative session handling | A remote/CLI side effect may require inspection after interruption |
| Image request returns content unsuitable for downstream file nodes | Binary output, artifact handling and source/reference checks | Video assembly remains downstream; an image response is not a completed video |
| Repeated reference-image downloads | Optional validated cache, freshness window and eviction | Avoided downloads are not equivalent to saved model input tokens |
| Operators lack visibility | Health, job history, diagnostics, usage/performance and gallery | A dashboard's live transport can be connected while data snapshots are stale |
| Accidental retention of sensitive content | Opt-in prompt/output retention, configurable local cleanup controls | SQLite deletion is not secure erasure of prior backups, WAL files or external copies |

## What I would measure in a demo

- Complete an invented episode/scene metadata request and inspect schema validity.
- Execute a small invented image request and confirm that n8n receives real image bytes rather than a JSON wrapper.
- Force a recoverable API/validation error and show the failed-job record without exposing a prompt, token or private media.
- Confirm that a repeated reference download is reused when eligible while still validated.
- Record actual latency, success rate and token usage from a clean demo before publishing any numerical claims.

## Lessons

The most underestimated part of automation reliability is the **interface between systems**: filenames, binary/text expectations, inconsistent metadata, retries, duplicate work, mutable URLs, partial failures and operational handoffs. A successful HTTP response does not by itself prove the next node can use the output.

Conservative rollback and an explicit failure report often protect a production workflow better than silently trying to recover from an ambiguous state.
