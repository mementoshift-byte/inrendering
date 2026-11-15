# 00 · Project charter and product context

**Status:** requirements capture and initial proposed boundaries.  
**Purpose:** preserve the intended system before writing production code.

## Thesis

The Command Center is a modular, conversation-first personal operating environment. inrendering exists to provide the **reusable chat interaction substrate** on which the rest of the system can converge. It is not intended to become the entire application, agent orchestrator, file service or knowledge-management system.

The initiating need is practical: the wider app already has Figma designs and partial functionality scattered across repositories. A working, well-isolated ChatFrame and MessageRenderer create an integration target without forcing premature amalgamation.

## User-established requirements

- The **MAIN** conversation is continuous, accessible from the center bottom navigation control, and especially useful for free-form thought and backend capture. It is not simply another domain tab.
- **Domains** are organized in home-screen top tabs. They group function-specific conversations, agents and workflows. The Figma LinkedIn example distinguishes a general conversation, specialized agent chats and workflow chats.
- Other application sections include **Files**, a webshop dashboard, repository dashboards and future tools. These are application surfaces, not responsibilities of the message renderer.
- Files should ultimately be accessible through centrally shared storage, potentially hosted on a physical device, with areas for agent, personal, financial and other material. The Obsidian layer for domain/agent/convention setup can live in this environment.
- The frame should be **aware of its logical locality**: active domain, agent, conversation type, capabilities, conventions, context and relevant user selections.
- The frame's full local context need **not** be reproduced in the agent prompt. A compact, relevant machine-readable projection is preferable.
- React UI components should generally be prebuilt and executed on the device. An agent may request components, semantic interactions or a constrained component tree; the system should locally resolve routine choices.
- Inputs can be requested by the agent and rendered in the interactive composer layer, not merely inside assistant messages.
- Text selections must be **precise** and may be non-contiguous, across multiple messages. A person should be able to accumulate references as they read, then respond without constantly scrolling back and forth.
- Low perceived latency, reduced token consumption, deterministic behavior and intentional caching matter.
- The layout should follow the actual Figma conversation screens rather than an arbitrary generic chat template.

## Boundaries for this repository

**In scope:**
1. Chat viewport/frame, message list, assistant and user presentation components, floating/anchored overlays and composer.
2. Typed message and content-block rendering, streaming stability and message identity.
3. Constrained component tree interpretation and local React component registry.
4. Locality/context resolution and integration with compiled convention manifests.
5. Precise selection/reference infrastructure and event dispatch.
6. State continuity, caching semantics, performance requirements and a runtime adapter.
7. Tests, Storybook-like test fixtures or minimal integration prototypes when implementation begins.

**Out of scope for version one:**
- Building the full Command Center shell, all domain dashboards or repository dashboards.
- Implementing the central file server, vault, knowledge graph or workflow scheduler.
- Building the webshop, LinkedIn, Gmail or financial domain logic.
- Designing an entire autonomous agent orchestration system.
- Embedding a full Obsidian editor into the frame.
- General arbitrary JSX compilation and execution as the default UI path.

These external capabilities connect through contracts and are not secretly absorbed into the frame.

## User experiences to optimize

**Explore-and-reference:** user reads a long response, highlights an exact phrase early, continues reading, highlights an unrelated passage and later submits one response with both citations.

**Agent-directed input:** assistant requests a known single-choice or structured input. The device resolves the interaction locally; the composer preserves ordinary draft text and selected references.

**Domain-contextual presentation:** the same semantic request, such as review a set of items, resolves to an allowed candidate review in LinkedIn, order review in shop or code review in a repository context according to explicit conventions.

**Navigate without losing work:** user switches from MAIN to a domain chat and back. Each conversation retains its draft, scroll position, selection references and UI state.

**Stream while interacting:** existing committed blocks retain identity during new assistant chunks. Highlights are not silently displaced by streaming updates.

## Quality characteristics

- **Responsiveness:** local UI state updates need no LLM round trip.
- **Determinism:** given a valid UI request, manifest versions, data and device capabilities, resolution should be reproducible.
- **Extensibility:** new domain-specific views are possible without modifying the message engine.
- **Accessibility:** text selection, keyboard navigation, focus management and touch interaction work across input modalities.
- **Security:** agents receive only authorized data and capabilities; generated code is never implicitly trusted.
- **Replaceability:** all agent and chat backend integrations happen behind a narrow adapter.

## Definition of the initial deliverable

Not a whole app. A running slice that reproduces the Main Chat and Shared Conversation patterns from Figma, renders streaming text and one interactive component, persists a draft and multiple exact references, and survives switching conversations. See 10-implementation-roadmap.md.

## Working method

Document choices using the explicit labels **Confirmed direction**, **Proposed design**, **Needs validation** and **Open question**. The discussion is exploratory; architecture recommendations should not be treated as approved implementation decisions without a deliberate checkpoint.
