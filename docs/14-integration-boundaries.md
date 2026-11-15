# 14 · Integration with the wider Command Center

**Status:** integration map, not a scope expansion for inrendering.

## Why an explicit boundary matters

The wider application will combine interfaces and functionality from several existing repositories. inrendering is meant to be the **first usable foundation** for that convergence. It should not become a monolith that directly imports every domain's networking, agent logic or storage implementation.

The app is a command center with chat as its principal interactive control surface, not an isolated messaging product.

## External surfaces and ownership

| External area | Owns | Contract into inrendering |
|---|---|---|
| App shell / navigation | Top domain tabs, bottom destinations, MAIN entry, route selection | Conversation ID, type, domain context, header identity |
| Domain/agent registry | Available agents, domain workflows, defaults | Versioned convention manifests, capability projection |
| Agent orchestration | Model selection, prompts, routing, tools, automation | Message events/streams and agent-facing request submission |
| Central Files service | Shared files, vaults, uploads/downloads, permissions | Resource IDs, authorized content fetch and metadata |
| Obsidian layer | Human-editable domain/agent/convention documents | Compiled, validated configuration manifest |
| Webshop | Orders, inventory, KPI overview, domain actions | Data-provider bindings and registered components |
| Repository dashboards | Repo state, code/review workflows, visualization | Typed resources, semantic intent mappings, authorized actions |
| Conversation persistence | Durable messages, edits, sync | Canonical envelope snapshots/patches |
| UI data cache | Device-local allowed data, resource revision | Versioned resource lookup and invalidation |
| Runtime adapter | Bridge from underlying chat library/backend | Message patch stream and action transport |

## MAIN vs domains

**MAIN:** a persistent continuous conversation, accessible from the central navigation control. It may capture notes, questions, quick tasks and cross-domain ideas. Backend capture is an emphasized role, but it remains a usable conversation.

**Domains:** top-level organizational categories on the home chat area; inside each, general chat(s), agents and workflow chats can be grouped. Domain conventions influence component mappings, input options and allowed actions.

The same ChatFrame should display both. Differences are **configuration and data**, not a separate message engine.

## Proposed conversation descriptors

~~~ts
type ConversationDescriptor = {
  id: string;
  kind: "main" | "general" | "agent" | "workflow";
  domainId?: string;
  agentId?: string;
  title: string;
  subtitle?: string;
  avatarRef?: string;
  conventionRef?: string;
};
~~~

The app shell supplies a descriptor or selects one through a domain registry. The frame does not discover navigation by scanning Obsidian on every mount.

## Files and Obsidian integration

The Files UI is a separate product page, backed by central shared storage on a physical device or chosen service. The Obsidian vault can store setup files, templates and domain-level conventions. A dedicated setup agent can guide edits to configuration.

The runtime path should be:

~~~text
Files UI / Obsidian editor
   -> validated settings file
   -> manifest compiler / publication
   -> versioned manifest cache
   -> resolved FrameContext
   -> local UI renderer
~~~

Avoid:

~~~text
every React render -> read arbitrary vault Markdown -> ask LLM -> recompute UI
~~~

That defeats both latency and deterministic execution.

## Resource contract for sensitive areas

The central storage may include agent/shared, personal and financial partitions. A domain component should receive only an authorized **resource reference**, and a service must decide whether the current actor can read or update it. Merely having a route to Files or a matching manifest does not grant read access.

Examples of good external contracts:
- getResource(resourceId, capability, callerScope)
- subscribeResource(resourceId, revisionCursor, callerScope)
- dispatchAuthorizedAction(actionId, payload, callerScope)
- getResolvedManifest(contextDescriptor, installedRegistryVersion)

These are conceptual interfaces. The security model must be implemented server-side or within an appropriately trusted local service.

## Integrating scattered repositories

Do not paste modules blindly into the renderer. For each existing repo, perform an **integration inventory**:

1. What capability does it provide?
2. Is it UI, domain data, orchestration or infrastructure?
3. Does it already have reusable typed contracts?
4. Which runtime/framework/dependency versions does it assume?
5. Which APIs and permissions does it need?
6. Does it belong in the app shell, a domain adapter, or genuinely in the renderer?
7. Can it be represented through an existing semantic UI mapping?

Adapters should preserve boundaries and allow old functionality to be connected one at a time.

## Integration sequence

1. Render standalone Figma chat variants with fixtures.
2. Connect a mock runtime and one semantic component.
3. Confirm reference/composer behavior.
4. Integrate an actual chat runtime/agent backend.
5. Integrate one low-risk read-only domain resource.
6. Integrate one authorized write workflow with confirmation and robust feedback.
7. Connect compiled Obsidian manifests.
8. Extend navigation and the remaining dashboards independently.

## What should remain independent long term

- Core chat interaction model
- Visual component implementation
- Domain definition/configuration
- Workflow/orchestration engine
- Storage and synchronization
- App-level navigation

This allows changes to one part without cascading implementation details through the whole app.
