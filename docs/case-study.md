# Case study: automation infrastructure for media content production

## Context

This project sits behind n8n-powered content-production workflows for a distributed media operation. An episode brief and related production data can result in multiple AI and media-processing tasks rather than a single model completion. The technical challenge was integrating a locally authenticated Codex execution backend into those tasks without breaking existing n8n contracts or exposing unbounded execution.

## My engineering approach

1. **Identify the existing contracts.** Inventory which n8n nodes expect plain text, JSON-shaped chat content or image bytes. Preserve their downstream parser assumptions where reasonable.
2. **Introduce a gateway boundary.** Let n8n send authenticated HTTP requests while the Python service handles the local Codex CLI lifecycle.
3. **Validate inputs and outputs.** Provide predictable request models, schema-constrained output where appropriate and explicit errors.
4. **Observe production-style execution.** Track queue and job state, elapsed time, model usage, and generated image artifacts so a failure can be located.
5. **Reduce repeated I/O.** Validate reference images and cache eligible sources without making unverified claims about token savings.
6. **Keep optional features gated.** Treat episode-context reuse and its cost/interrupt behavior as an engineering trade-off rather than a default promise.

## A simplified fictional walkthrough

*This example describes the shape of an integration, not a raw export or a guarantee that every illustrated content step has been completed in a single live run.*

- A fictional brief for a 30-second coffee-shop video arrives in an n8n test workflow.
- n8n requests structured scene metadata from the gateway.
- The gateway runs Codex and returns a JSON-like response that n8n checks before continuing.
- Selected scene prompts can be sent through the gateway's image-generation route; n8n receives image bytes.
- Separate n8n nodes and providers manage other assets and FFmpeg-connected rendering.
- The operator can inspect job status and artifacts if any intermediate task fails.

## Boundaries

The public materials deliberately do not expose internal source, real workflow exports, client names, endpoints, media assets, private repo history or credentials. Any public screenshots should be made with fictional jobs in a disposable demo environment and reviewed before publication. This repository is primarily an architecture and problem-solving case study; a live demo requires separate validation and safe demo data.

## Relevance to automation engineering

The most transferable part of this work is designing the *glue*: reliable request/response contracts, failure handling, operational visibility, media/AI integration and iterative rollout. It complements separate practical experience with Teamwork, Zapier, Apps Script and Slack production handoffs.

## What the supplied captures actually demonstrate

The n8n screenshot shows a branching test-lab content automation canvas. The gateway screenshots show local health monitoring, recorded Codex-backed generation/image jobs, selected reference images, an artifact gallery and a seven-day activity view. This supports an **integrated working test environment**, not a claim that the gateway alone generates fully rendered videos or that every branch completed end-to-end in production.

The supplied seven-day report displays 24 completed tasks, 24 successes and zero failures, including 10 generated images. This is a bounded visual sample, not a longer-term reliability guarantee. The approximately 1.3M cached *model-input* tokens visible in the screenshot are not an estimate of savings from the optional reference-image *download* cache; those are separate systems.

Screenshots are selectively cropped and identifiers masked. The full live workflow export, secrets, private endpoints and source code are deliberately not part of this portfolio.
