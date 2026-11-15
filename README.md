# inrendering

**Locality-aware chat frame, message renderer, and constrained component rendering infrastructure for the Command Center.**

> Status: architecture and research baseline, not an implemented library.  
> Last documented: 2026-11-15.  
> Primary design reference: [Command Center Figma](https://www.figma.com/design/v17vwmp74ZbPpQwnVLelfB/Untitled?node-id=0-1).

## Why this repository exists

The Command Center is a modular, conversation-first personal operating environment. Its wider scope includes a persistent MAIN conversation, domain-organized agents and workflows, centralized file storage with an Obsidian-managed configuration layer, a webshop overview, and repository dashboards.

**inrendering is deliberately narrower.** It is the reusable interaction and rendering foundation that the other pieces can integrate with:

- A **ChatFrame** that owns the viewport, message surface, overlays, scrolling and continuity.
- A **MessageRenderer** that handles typed assistant/user message content, stable block identity, streaming and references.
- A **ComponentRenderer** that interprets constrained, typed UI requests and locally resolves them to prebuilt React components.
- A **Convention/locality resolver** that lets the frame know its domain and capabilities without repeatedly sending the entire context to an agent.
- A **selection/reference system** for precise, persistent, non-contiguous selections, plus a composer that can host agent-requested, prebuilt inputs.
- A **runtime adapter** (assistant-ui is the leading candidate, pending an integration spike) so conversation mechanics do not dictate the architecture.

The frame may maintain much more contextual information than it exposes to an agent. The agent should emit only the **meaningful distinctions** it needs; the client should resolve routine presentation and interaction decisions deterministically.

## Guiding principles

1. **Chat first.** Do not require the rest of the Command Center to be built before this subsystem becomes useful.
2. **Own our contracts.** The message model, selections, conventions and component-tree protocol should not be coupled to one vendor or framework runtime.
3. **Make the device do the work.** Prebuilt components, local state, cached templates, local interactions and domain defaults reduce latency and token use.
4. **Constrain ambiguity.** Prefer typed intent and semantic components over repeatedly constructing arbitrary UI.
5. **Progressive specificity.** Resolve simple intents locally; accept explicit constrained trees when necessary; treat generated executable JSX as an exceptional, isolated fallback.
6. **Preserve exact references.** Selection is an operation, a reference is its durable result, and highlighting is a visualization.
7. **Frame locality is not prompt bulk.** The frame knows domain, conversation, capabilities, device and content locality; agents receive a minimal capability projection.
8. **Separate content, behavior and environment.** Messages do not own domain orchestration; Obsidian does not become a per-render dependency; navigation does not define the chat engine.
9. **Version and cache deliberately.** Schema, convention, tree, content and state revisions have different lifecycles.
10. **Protect user boundaries.** Agents must not receive unrestricted access to centrally stored personal or financial material by virtue of shared storage.

## Documentation map

Read in this order for the architecture:

| Document | Subject |
|---|---|
| [00 · Project charter](docs/00-project-charter.md) | Scope, wider app context, user experience |
| [01 · Architecture](docs/01-architecture.md) | Subsystems, ownership, data flow, package boundaries |
| [02 · ChatFrame](docs/02-chat-frame.md) | Viewport, overlays, composer, continuity |
| [03 · Message model](docs/03-message-model.md) | Typed content, streaming, stable identity |
| [04 · Component engine](docs/04-component-engine.md) | Constrained trees, semantic resolution, React registry |
| [05 · Locality & conventions](docs/05-locality-and-conventions.md) | Obsidian manifests, precedence, agent projection |
| [06 · Selections & references](docs/06-selections-and-references.md) | Precise cross-message references |
| [07 · Input, actions & state](docs/07-input-actions-and-state.md) | Interactive inputs, event routing, persistence |
| [08 · Caching & performance](docs/08-caching-and-performance.md) | Cache keys, invalidation, device-first behavior |
| [09 · assistant-ui adapter](docs/09-assistant-ui-adapter.md) | Reuse vs custom, evaluation gates |
| [10 · Implementation roadmap](docs/10-implementation-roadmap.md) | Incremental milestones and acceptance criteria |
| [11 · Decisions & questions](docs/11-decisions-and-questions.md) | Explicit decisions, proposals and unresolved issues |
| [12 · Contract sketches](docs/12-contract-sketches.md) | Illustrative TypeScript interfaces and wire examples |
| [13 · Figma design map](docs/13-figma-design-map.md) | Existing screens and component extraction |
| [14 · Integration boundaries](docs/14-integration-boundaries.md) | How other repositories and modules attach |
| [15 · Glossary](docs/15-glossary.md) | Terminology |
| [Agent guidance](AGENTS.md) | Instructions for future coding agents in this repo |

## Status and next move

No production code is assumed to exist. Before importing existing repo fragments, build an isolated **vertical slice**:

1. Render the two Figma conversation variants from structured mock data.
2. Preserve independent conversation drafts and viewport state while switching.
3. Render a constrained semantic UI request with local components and one real action round trip.
4. Select multiple exact text ranges from different messages and attach those references to a draft.
5. Verify an assistant-ui adapter can stream without breaking identity/selection guarantees; replace it if not.

See [the roadmap](docs/10-implementation-roadmap.md) for testable exit criteria.

## Decision provenance

This repository distinguishes **Confirmed direction** (explicitly desired in discussion), **Proposed design** (recommended but not yet accepted), **Needs validation** (requires a spike), and **Open question** (not decided). Do not silently promote a proposal into a requirement.

The docs are living architecture notes, not a claim that all contracts, libraries or storage technologies have already been finalized.
