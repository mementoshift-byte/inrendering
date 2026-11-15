# 10 · Implementation roadmap, milestones and verification

**Status:** proposed execution sequence; do not start by integrating the whole Command Center.

## Overall strategy

Build the smallest *real* frame that is useful as an integration target. This means a thin vertical slice with functional messaging, navigation continuity and exact selections, not an enormous component library.

The broader application already has design work in Figma and code spread across repositories. Do not merge those pieces prematurely. Give them a stable rendering/interaction contract to connect to later.

## Milestone 0 · Repository and architecture baseline

**Work**
- Establish README, ownership boundaries, draft contracts, decision log and Figma mapping.
- Choose a TypeScript-based package/workspace setup after inspecting integration constraints.
- Create mock fixtures for MAIN and a shared agent conversation.
- Document how to run a local demo and tests before adding backend dependencies.

**Exit:** contributors can explain which subsystem owns layout, rendering, locality, input, references and agent execution. No hidden domain coupling.

## Milestone 1 · Visual ChatFrame

**Work**
- Implement responsive ChatFrame using Figma Main Chat **5:541** and Shared Conversation **5:260** as targets.
- Build ConversationHeader slot, message viewport, AgentMessage and UserMessage shells.
- Create composer overlay surface separate from message flow.
- Implement mobile safe-area and keyboard handling at the host boundary.
- Render mock messages and date dividers; preserve scroll when switching chats.

**Exit:** both Figma variants display via the **same frame implementation** with different identity/content props. The composer does not become another message item. User can switch MAIN/shared and return to the same viewport.

**Avoid:** importing full domain tabs, financial data, repo dashboards or running agent tools.

## Milestone 2 · Canonical messages and streaming

**Work**
- Define typed message envelopes, block IDs, block revisions and adapter events.
- Implement text/markdown/status/attachment/UI slot renderers with safe fallbacks.
- Add a mocked streaming source and patch reconciliation.
- Verify old blocks keep identity as new tokens arrive.
- Add cancellation/error state representation and recovery fixtures.

**Exit:** a stream can append and complete without remounting unrelated blocks; snapshot replay yields equivalent content.

## Milestone 3 · Precise selection and composer continuity

**Work**
- Determine canonical text representation and cross-platform range encoding.
- Implement selection-to-canonical-text mapping for simple text and markdown.
- Support adding/removing multiple references from different messages.
- Display a reference tray independent of draft text.
- Persist per-conversation draft, reference set and scroll anchor.
- Mark references stale if source edits cannot be safely reanchored.

**Exit:** choose individual words and non-contiguous passages, switch chats and return, then submit all references in one structured event. No wrong-text reanchoring.

**Risk:** this should be tested early, even if the overall order is rearranged, because it can influence which rich-text renderer/runtime is viable.

## Milestone 4 · Minimal constrained component interpreter

**Work**
- Define semantic-request and explicit-tree schemas.
- Build registry with a handful of nodes: Text, Stack, Action, SingleChoice and one data display.
- Add tree validation (schema, child grammar, node/depth limits and action IDs).
- Render the same component inside an agent message and inside the composer overlay.
- Route local value changes locally and send submission through the shared event bus.

**Exit:** one complete choice interaction works without an agent call for local changes, and returns typed values on submit. Invalid UI fails locally without breaking its containing message.

## Milestone 5 · Locality + conventions

**Work**
- Create a local mocked convention manifest (do not build Obsidian sync yet).
- Implement global/domain/agent/conversation resolution with deny/allow precedence.
- Add versioned semantic mappings, e.g., review a known collection.
- Generate a compact agent capability projection separate from full FrameContext.
- Verify changes invalidate correct caches only.

**Exit:** the same generic semantic intent resolves to two different known registered components when context makes that mapping unambiguous; unknown/unauthorized mappings fail safely.

## Milestone 6 · Runtime adapter decision

**Work**
- Spike assistant-ui in isolation, comparing adapter complexity and behavior to the custom contracts.
- Verify streaming, scroll behavior, component slots, stable block identity and selection compatibility.
- Keep a fallback baseline implementation that does not require assistant-ui internals.
- Record an ADR with observed behavior and selected library version.

**Exit:** conscious **hybrid vs minimal custom runtime** decision grounded in working tests. Neither option rewrites the frame's own protocols.

This spike can run in parallel with early milestones, particularly the selection proof.

## Milestone 7 · State persistence, caching and polish

**Work**
- Implement per-conversation local persistence and controlled hydration.
- Cache parsed blocks, compiled manifests and templates with explicit invalidation.
- Instrument streaming and interaction latency.
- Verify keyboard/touch accessibility, resize/scroll anchoring and long conversations.
- Apply permission-scoped resource cache rules.

**Exit:** warm navigation restores state reliably; ordinary UI interactions never invoke the model, and performance regressions are observable.

## Milestone 8 · First external integration

**Work**
- Attach one existing agent backend and one real data-backed UI component through adapters.
- Verify files/data fetch via authorized services, not arbitrary component reads.
- Attach domain selection/routing from the wider app shell.
- Connect a real convention source only after manifest contract stabilizes.

**Exit:** a real domain can supply messages, a semantic UI request and one authorized action without modifying the MessageRenderer's domain-independent logic.

## Build slices instead of layers in isolation

Each milestone should deliver a runnable page/scenario, not merely a package full of types. Prefer a small mock backend and deterministic fixtures so UX can be evaluated before infrastructure complexity arrives.

## End-to-end acceptance matrix

| Scenario | Must hold |
|---|---|
| Long streaming assistant reply | Previous block identities stable; readable scroll |
| Highlight word in message A and phrase in B | Both exact references coexist |
| Open composer with selected references | Draft and references remain separate |
| Switch to different domain, then back | Restored draft, references and scroll |
| Agent requests local SingleChoice | Local edits need no agent call |
| Same review intent in different domains | Deterministic permitted component mapping |
| Unsupported component/action | Safe explicit fallback, no execution |
| Change a domain manifest | Relevant conventions refresh, old message meaning stable |
| Two instances of same cached form | Separate state and values |
| Remote data access denied | UI cannot bypass service permissions |
| Mobile keyboard/touch selection | Can select and compose without viewport disruption |

## What not to optimize prematurely

Do not prebuild hundreds of components, invent a general JSX sandbox, migrate every scattered repository, or design a complete cross-device sync backend before the initial frame is usable.

**Recommended first concrete commit after documentation:** minimal project scaffold, mock conversation fixtures, ChatFrame skeleton and a visual comparison page for the two Figma references.
