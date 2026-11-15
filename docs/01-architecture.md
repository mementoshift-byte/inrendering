# 01 · System architecture

**Status:** proposed decomposition consistent with confirmed requirements.

## Architectural shape

The system is best viewed as a small **local UI runtime** with independent conversation and component interpretation.

~~~text
Command Center shell (elsewhere)
  ├─ Top-domain navigation / bottom destinations
  ├─ MAIN routing and domain chat routing
  └─ selects conversation + its resolved environment
                 │
                 ▼
     ┌──────────────────────────────┐
     │ ChatFrame                    │
     │ - viewport and scroll        │
     │ - message placement          │
     │ - overlay surfaces           │
     │ - composer host              │
     │ - continuity                 │
     ├──────────────────────────────┤
     │ MessageRenderer              │
     │ - user/agent turn variants   │
     │ - typed blocks, streaming    │
     │ - stable identity            │
     │ - UI slots                   │
     └─────────────┬────────────────┘
                   │ UI block
                   ▼
     ┌──────────────────────────────┐
     │ Component interpreter        │
     │ intent -> resolved UI tree   │
     │ normalize -> validate        │
     │ registry -> React components │
     └──────────────────────────────┘
                 ▲
                 │ resolved context
     ┌──────────────────────────────┐
     │ Locality/convention resolver │
     │ global/domain/agent/chat     │
     │ compiled & versioned config  │
     │ capability projection        │
     └──────────────────────────────┘
       ↕ selection/event/state services
       ↕ external runtime adapter (candidate: assistant-ui)
       ↕ file/config/data services (external)
~~~

This is a **conceptual dependency picture**, not a requirement to put everything in one React component or one package. The ComponentRenderer can also be mounted by a composer overlay or a dashboard outside messages.

## Ownership rules

### Application shell

Owns navigation structure, selected conversation, domain listing, app-wide permissions, workspace layout and routing to non-chat pages. The chat frame must not understand whether it was opened from the LinkedIn tab, a repository dashboard or the MAIN button.

### ChatFrame

Owns the visible conversation environment: header slot, bounded message viewport, scroll restoration, overlay placements, composer host, keyboard/safe-area handling and conversation-specific local UI continuity. It consumes a resolved context. It **does not** read the Obsidian vault directly.

### MessageRenderer

Consumes a canonical list/stream of message envelopes. Handles role-specific message presentation, ordered content blocks, updates by stable IDs, error/streaming states and handoff to the ComponentRenderer for UI blocks. It should not choose domain components by itself.

### Component interpreter

Consumes **semantic requests** or an explicit **constrained UI tree**. Resolves registered components, expands semantic nodes into concrete layout trees, normalizes defaults, validates types/children/actions and renders via local React. It must be independently usable outside the ChatFrame.

### Locality/convention resolver

Loads a compiled, versioned convention manifest, combines global/domain/agent/conversation settings, applies device and security constraints, and produces a deterministic **FrameContext**. It exposes a smaller capability projection to an agent when necessary.

### Interaction and reference service

Interprets typed UI events, exact selections and composer input. Can route locally, into application handlers, or to the agent through a deliberate bridge. It is not dependent on DOM coordinates for durable identity.

### Chat runtime adapter

Normalizes message snapshots, streams, generation state, cancellation and backend events into our contracts. assistant-ui is a strong *candidate*, not a forced foundation. The core should remain backend-agnostic.

## Control-plane vs content-plane separation

**Control/configuration plane:** convention manifests, domain policies, allowed component registry, route permissions, theme and behavior rules. Generally versioned and relatively stable.

**Content/data plane:** messages, streaming blocks, data references, values, attachments and live domain resources.

**Interaction plane:** drafts, selections, form state, focus, scroll anchors and typed events.

Avoid using one global store or cache invalidation rule for all three. This separation is critical for performance and privacy.

## Core runtime flows

### Message flow

1. An adapter emits a message envelope or incremental update with stable IDs.
2. The message store reconciles revision/sequence and maintains canonical content.
3. MessageRenderer maps block types to text, attachment, status or UI slots.
4. UI blocks are evaluated against the active versioned context and passed to the component interpreter.
5. The local React registry mounts prebuilt implementations; updates preserve identities where possible.

### Semantic component flow

1. Agent emits compact intent, for example **review pending orders**.
2. Resolver checks context, capabilities, available domain data and explicit mappings.
3. If exactly one allowed mapping exists, expand a semantic template into a typed component tree.
4. Normalize, validate and render locally.
5. If data, authority or intent is insufficient, show a safe unresolved state or ask for specific data. Do not silently guess.

### Selection flow

1. User makes an exact selection within a rendered text/structured block.
2. Selection controller translates it into a content anchor.
3. Reference store retains anchor + revision + quote/data address.
4. Composer displays reference chips independently of the underlying message position.
5. On submit, the application decides how much referenced content the agent needs to receive.

## Suggested package boundaries

~~~text
packages/
  chat-contracts/      # canonical types and schema versions
  chat-frame/          # visible frame, message surfaces and overlays
  message-renderer/    # typed parts and streaming reconciliation
  component-engine/    # semantic resolver, grammar, renderer, registry
  conventions/         # manifest validator, merge, locality resolver
  references/          # selection anchors and reference store
  runtime-adapters/    # assistant-ui and future bridges
~~~

Package names are provisional; logical boundaries are more important than creating seven npm packages on day one.

## Invariants

- App shell -> frame, not frame -> app shell.
- Renderer never executes arbitrary agent-provided code in its privileged React tree.
- Convention rules are declarative and validated before use.
- Permission policy is enforced by the app/services, **not** merely by hiding buttons.
- UI nodes, message blocks, data resources and references each have stable identity and explicit revisions where applicable.
- Local interactions do not require model calls unless a semantic decision or remote operation genuinely requires one.
- Device-local runtime state can be much richer than the machine-readable context projected to agents.

## Open architecture checks

- Is assistant-ui compatible with stable custom content-block identity and the desired cross-message selection behavior?
- Which state belongs in memory, persisted browser/native storage, Supabase or another backend?
- What text representation gives reliable exact selections across platforms and streamed edits?
- How will responsive React web and eventual native implementations share contracts without imposing one presentation tree on both?
