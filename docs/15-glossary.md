# 15 · Glossary

Shared vocabulary is important because several similar-looking concepts have different owners.

| Term | Meaning |
|---|---|
| **Command Center** | The larger application: domain navigation, MAIN, Files, shop, repos, agents and workflows |
| **inrendering** | The independent chat frame and rendering infrastructure within that app |
| **MAIN** | Persistent, continuous general-purpose conversation entered through the center navigation control |
| **Domain** | Organizational context such as LinkedIn, CRM or Shop; usually has chats/agents/workflows |
| **Conversation kind** | MAIN, general, agent or workflow; affects defaults and routing |
| **ChatFrame** | Viewport, message surface, overlay anchors, composer host, continuity |
| **MessageRenderer** | Converts typed canonical message blocks into visible message content and UI slots |
| **AgentMessage / UserMessage** | Role-specific message presentation containers |
| **ComponentRenderer / interpreter** | Local engine resolving constrained UI requests and mounting registered React components |
| **Component registry** | Installed prebuilt components and their typed contracts |
| **Primitive** | Small layout/interaction component (e.g., Text, Stack, Action) |
| **Semantic component** | Predefined higher-level interaction (e.g., SingleChoice) expanded locally |
| **Domain component** | Specialized registered component (e.g., OrderReview) |
| **UI tree** | Typed, validated node hierarchy; not executable JSX |
| **Semantic UI request** | Compact request expressing purpose and target rather than explicit layout |
| **Normalization** | Deterministic expansion/default application/canonicalization before rendering |
| **Convention** | Rule governing component usage, presentation, mappings, input and interactions |
| **Obsidian layer** | Human-editable vault/configuration environment, not a per-render dependency |
| **Manifest** | Compiled, validated and versioned configuration consumed by the runtime |
| **Environmental locality** | Frame's position in domain/agent/chat/device/capability context |
| **Content locality** | Exact message/block/span/object addressed by an interaction |
| **FrameContext** | Rich effective local context after convention and permission resolution |
| **Capability projection** | Smaller machine-readable capabilities exposed to an agent |
| **Selection** | User gesture indicating exact content |
| **Reference** | Durable typed description of a selected source and range/object |
| **Highlight** | Visual manifestation of a selection/reference, not its source of truth |
| **Reference tray** | Composer UI for accumulated references |
| **Composer overlay** | Persistent input surface outside message flow |
| **Typed action event** | Standardized interaction report; may stay local, reach services or create an agent turn |
| **Runtime adapter** | Bridge from a chat backend/library into inrendering's message/event model |
| **Canonical message** | Backend-independent message envelope with stable identities |
| **Block revision** | Version of a particular addressable content block |
| **Structural template** | Reusable component structure independent of a particular UI instance's data/state |
| **UI instance** | Actual rendered occurrence with its own stable ID and input state |
| **Render cache** | Cached parsed content/templates/manifests, governed by versioned invalidation |
| **Historical pinning** | Retaining the schema/template interpretation of old UI messages |
| **Generated JSX** | Model-produced executable UI code; optional later fallback requiring isolation |
| **Vertical slice** | Small integrated working product path, used to test the whole architecture early |

## Three distinctions to retain

**Selection ≠ Reference ≠ Highlight.** Interaction produces an addressable object; styling displays it.

**Semantic request ≠ Component tree ≠ React code.** Intent is resolved into a constrained representation, then executed through prebuilt components.

**Full frame locality ≠ Agent capability projection.** The client maintains the comprehensive model; an agent receives only the minimum meaningful machine-readable capabilities.
