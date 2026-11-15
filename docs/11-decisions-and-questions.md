# 11 · Decision log, established requirements and unresolved questions

**Status:** living record. This document distinguishes what was actually asserted in the design conversation from assistant recommendations.

## Status vocabulary

- **Confirmed direction:** clearly requested/endorsed by the user.
- **Proposed design:** a concrete engineering recommendation, not yet approved.
- **Needs validation:** candidate approach requiring experiment/library/API verification.
- **Open question:** legitimate remaining choice.
- **Deferred:** intentionally outside the first usable slice.

## Confirmed direction

| ID | Topic | Decision/direction |
|---|---|---|
| C-01 | Initial scope | Build ChatFrame and MessageRenderer first, to start integrating the other application fragments |
| C-02 | App nature | Command Center is broader than chat: domains, files/Obsidian, webshop, repo dashboards and future functions |
| C-03 | MAIN | Middle navigation control opens a persistent continuous conversation oriented toward general interaction and backend capture |
| C-04 | Domain organization | Home-screen top tabs organize domain-specific general, agent and workflow conversations |
| C-05 | Files | Shared central storage should be accessible from the Files area; an Obsidian layer can define domain/agent/convention setup |
| C-06 | Device-first | Maximize locally executed work for responsiveness and token economy |
| C-07 | Components | Prefer prebuilt React components; agent can request/compose them rather than regenerate presentation details |
| C-08 | Component trees | Introduce a constrained tree protocol to reduce ambiguity |
| C-09 | Locality | Frame knows active domain/conversation/agent conventions and can resolve semantic UI differently by locality |
| C-10 | Projection | Full frame locality context does not have to be exposed to the agent in full |
| C-11 | Input | Agent can specify generative input via registered components, including overlay composer interactions |
| C-12 | Selection precision | Highlight exact content and accumulate non-contiguous references across messages |
| C-13 | Visual source | Use the existing Figma Command Center screens, not a newly invented generic design |
| C-14 | Modularity | Keep frame/message renderer separate from full app orchestration and external functionality |

## Proposed engineering decisions

| ID | Decision | Why |
|---|---|---|
| P-01 | Own canonical message/block/reference IDs | Selection and persistence should survive library changes |
| P-02 | Keep component interpreter usable outside chat | Same engine serves composer and future dashboard surfaces |
| P-03 | Compile/version Obsidian conventions into manifests | Avoid vault parsing during rendering |
| P-04 | Keep App Shell -> ChatFrame directional dependency | Prevent top-level navigation from infecting renderer |
| P-05 | Separate control, content and interaction planes | Cleaner state, security and cache invalidation |
| P-06 | Semantic request -> deterministic resolver -> normalized tree -> React registry | Minimize agent decision load while retaining flexibility |
| P-07 | Type all actions and authorize them in handlers | UI visibility alone is not security |
| P-08 | Reference store independent of DOM highlight state | Precise multi-selection and cross-device compatibility |
| P-09 | Preserve per-conversation drafts, scroll and form state | Natural return/navigation experience |
| P-10 | Prefer cached template and explicit UI tree before generated JSX | Latency, stability and safe execution |
| P-11 | Pin UI schema/template interpretation for historical content | Avoid silent semantic changes |
| P-12 | Create thin slice before integrating scattered repositories | Early UX feedback, reduced architectural coupling |

## Needs validation

| ID | Candidate | Test |
|---|---|---|
| V-01 | assistant-ui as headless/chat-runtime primitives | Integration spike with streaming, component slots and selection |
| V-02 | assistant-ui external-store integration | Check current API against chosen version |
| V-03 | Text anchor encoding | Cross-platform Unicode/Markdown selection fidelity |
| V-04 | Scroll virtualization | Test interaction with native text selection and dynamic heights |
| V-05 | React Native reuse | Share contracts, test separate native renderers where required |
| V-06 | Persisted UI pinning approach | Snapshot expanded tree versus semantic request + versioned template |
| V-07 | Domain manifest hot reload | Confirm safe cache invalidation and historical render stability |

## Open questions

### UI/runtime
- Is the initial target web responsive/PWA, native mobile, or both with a shared codebase?
- Which rich-text representation and parser gives precise stable range mapping?
- What is the exact overlay behavior when a composer input is agent-configured while the user has an unfinished draft?
- Which known UI primitives should be first-class versus semantic composites?
- When is a resolved component tree cached locally versus persisted remotely?

### Agent/context
- How does a capability projection get injected into prompts/tooling, and what refresh guarantees does the chosen agent orchestration system support?
- Does a frame ever proactively request additional agent information, or only return a needs-specificity result?
- How are agents allowed to discover extra component schemas without sending the whole registry?

### Files/configuration
- What machine/NAS/service will host central file storage?
- How will mobile/desktop clients authenticate and synchronize?
- What belongs in Obsidian Markdown/YAML files versus an application database?
- Which setup-agent edits require user review and schema validation before activation?
- How will sensitive personal/financial partitions be permissioned for agents?

### Persistence/actions
- Which interactions become canonical remote records versus device-local state?
- What is the cross-device conflict policy for drafts and selection trays?
- How are duplicate remote action submissions detected and confirmed?
- How is an old selection reanchored after edits, and when is it marked stale?

### Architecture integration
- Which existing repos have usable message/runtime code, and which contain only domain features?
- Should inrendering ship as an independent package, a workspace package, or a subproject until interfaces stabilize?
- What is the first real backend/provider to integrate?

## Deferred by design

- Full Command Center navigation and all dashboards
- Central file server implementation and Obsidian UI
- Full workflow orchestration/knowledge graph
- Large generative UI catalog
- Unrestricted executable JSX generation
- Cross-device synchronization beyond the essential initial proof
- Domain-specific business logic

## Decision discipline

When implementing, move an item from Proposed/Needs validation to an adopted ADR with:
- Context and constraints
- Options considered
- Decision and why
- Tradeoffs and consequences
- Test or evidence
- Date, affected contracts and compatibility/migration notes

This prevents exploratory statements from becoming invisible commitments.
