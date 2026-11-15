# 09 · assistant-ui versus custom runtime

**Status:** **Proposed recommendation: hybrid**; exact library API/compatibility is **Needs validation**.

## Why this question matters

The subsystem requires a reliable chat runtime and a distinctly nonstandard interface: precise multi-selection references, rich device-local UI interpretation, overlays, persistent composer state and locality-aware semantic resolution.

Building every ordinary conversation mechanism from scratch would consume effort that belongs in the unique renderer. Conversely, adopting a library's complete UI and message model as our architecture could make precise references and local conventions difficult to implement.

## Working recommendation

**Use assistant-ui as a candidate headless runtime/primitives layer, but own the frame, message contract, component interpreter, convention resolver and reference system.**

Avoid a wholesale fork of a full chat app. Prefer an adapter that translates assistant-ui runtime output into our stable message/stream contracts. This prevents lock-in if the runtime becomes a blocker.

## What to evaluate for reuse

- Message/turn runtime and generation lifecycle
- Streaming and interruption/cancellation behavior
- Standard message-part infrastructure
- Optional accessibility/keyboard/attachment primitives
- Thread viewport and scroll primitives if their behavior fits our restoration needs
- External-store adapter possibilities for using our own canonical store

The names and current availability of library APIs are **version-sensitive** and should be checked at implementation time against the installed release. This document does not claim these are all guaranteed in a particular current version.

## What should be custom regardless

- ChatFrame composition and Figma visual surfaces
- Exact multi-message, non-contiguous selection/reference store
- Canonical message/block/reference identities
- Semantic UI request protocol and constrained tree grammar
- Locality and Obsidian convention resolver
- Agent-facing capability projection
- Domain-specific component bindings and actions
- Independent per-conversation draft/form state
- Cache policy and permission-sensitive data routing

## Adapter boundary

~~~text
assistant-ui / other chat backend runtime
                |
                v
        ChatRuntimeAdapter
        - snapshots + deltas
        - generation status
        - retry/cancel
        - attachments/events
                |
                v
      Our canonical MessageStore
                |
                v
       MessageRenderer / Frame

Outbound:
UIEvent / composer submission -> adapter/application router -> chosen agent/backend
~~~

Avoid letting assistant-ui-specific data structures travel through every component in the app. Keep them contained in one adapter and tests.

## Spike plan (the decisive proof)

Create a tiny implementation using mocked responses and one simple streamed source:

1. Display the two Figma conversation variants with custom AgentMessage and UserMessage.
2. Stream a multi-block assistant response while preserving block IDs.
3. Render a component via our interpreter inside a message.
4. Select a precise phrase, save it, select a second phrase in a different message, and show both as references in the composer.
5. Switch conversations and back; restore draft, references, UI state and scroll anchor.
6. Test a domain convention change without remounting unrelated content.
7. Verify mobile touch behavior and web keyboard behavior.

Record how much public API surface is used, whether internal library state fights our store, and whether virtualized/scroll behavior breaks selection.

## Go/no-go criteria

**Go hybrid** when it supplies reliable streaming/lifecycle and viewport functionality without forcing:
- message identity tied to ephemeral UI representation;
- one active quote only;
- remounting all blocks on every update;
- composer state owned by a component that is repeatedly replaced;
- opaque control of scroll/selection logic; or
- domain logic embedded into runtime internals.

**Choose custom minimal runtime** if the adapter needs invasive internal patches to work, or if the library prevents precise selection/reconciliation. Even in that case, reuse mature primitives independently where possible.

## Risks of starting from scratch

Custom streaming, retry/cancel, attachment and tool handling, keyboard semantics, accessibility and scroll continuity are real projects. Do not assume a blank React list will become a production-quality conversation system automatically.

## Risks of adopting too much

A UI library's default data model and quote/selection assumptions may be incompatible with ours. Avoid hacking persistent cross-message references into a temporary selection toolbar API unless it explicitly supports the necessary contract.

## Deliverable from spike

A short ADR with: installed version, observed APIs, code-size/adapter surface, selection behavior, performance measurements, limitations and chosen integration mode. Do not decide by marketing feature checklists alone.
