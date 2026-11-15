# 05 · Locality, conventions and agent capability projection

**Status:** **Confirmed direction** for frame locality and constrained agent exposure; configuration details are proposed.

## The key architectural insight

The frame should have a **richer local understanding of its environment than the agent needs to see**. The agent does not need the whole frame manifest, all installed components, formatting rules, device characteristics or permission graph. It needs a compact view of relevant choices and actions.

In the discussion, the decisive observation was: the component interpreter should be able to resolve a generic request **differently according to local context**, without making the agent choose every concrete React component.

This is more than UI theming. Locality is an **input to deterministic semantic interpretation**.

## Two distinct forms of locality

### Environmental locality

Where the frame is in the broader Command Center:

- App surface (MAIN, domain chat, embedded workflow or future dashboard)
- Domain ID (e.g., LinkedIn, Gmail, CRM, Shop, repo)
- Conversation ID and type (general, agent, workflow, MAIN)
- Agent identity and effective capabilities
- Component availability and preferred semantic templates
- Config versions, device capabilities and current user permissions
- Applicable data-resource bindings and cached templates

### Content locality

Where a specific user interaction belongs:

- Message ID and block ID
- Text range or structured-object address
- UI node ID or data point ID
- Selected references already attached to the composer
- Stable content revision and semantic surrounding context

These localities intersect. The same text-range selection can expose different **allowed operations** in a research chat versus a webshop review, while its underlying reference remains the same.

## Proposed configuration flow

~~~text
Central storage / Obsidian vault (human-editable)
          |
          v
Configuration loader
  - parse and validate
  - compile manifests
  - sign/version if distributed
          |
          v
Cached effective conventions
  - global defaults
  - domain overrides
  - agent-specific setup
  - conversation type
  - capability/security restrictions
  - device presentation constraints
          |
          +-------> FrameContext (rich, local)
          |
          +-------> CapabilityProjection (small, agent-facing)
~~~

**Obsidian is a management surface for the files**, not a renderer dependency. A user may create a new domain through the Files page with preloaded configuration templates and a setup agent to help edit the settings. Runtime manifests are loaded from a compiled/cacheable version.

## Suggested directory organization (illustrative)

~~~text
vault/
  system/
    conventions/
      components.yaml
      actions.yaml
      rendering.yaml
  domains/
    linkedin/
      domain.yaml
      agents/
        recruiter.yaml
      workflows/
        profile-review.yaml
    crm/
      domain.yaml
    shop/
      domain.yaml
  templates/
    domain-starter/
      domain.yaml
      agent.yaml
      workflows.yaml
~~~

This is a *layout sketch*, not an Obsidian filesystem mandate. The eventual central file service may separate public/config, shared agent, personal and financial partitions.

## Example convention manifest

~~~yaml
schema: frame-conventions.v1
domain: shop
version: 3

components:
  allow:
    - Text
    - Stack
    - SingleChoice
    - OrderReview

semanticMappings:
  - intent: review
    target: pending_orders
    template: order-review.v2
    dataSource: shop.pending_orders

composer:
  allowStructuredInput: true

selectionActions:
  - reference
  - inspect_order
~~~

This defines *meaningful permitted capabilities*. It does not contain executable callbacks or privileged credentials. The resolver would bind the symbolic data source to an authorized data provider.

## Effective context resolution

Given:
- Global manifest G
- Domain manifest D
- Agent manifest A
- Conversation manifest C
- Device capability profile V
- Current user/security policy P

Compute an effective context **R = resolve(G, D, A, C, V, P)** and give it a stable resolution ID/version. Domain and conversation may override **presentation defaults**, but **cannot** elevate permissions beyond P or install unapproved components through a text file.

Suggested merge semantics:
- Theme/layout defaults: deepest declared override wins, with schema validation.
- Allowed component sets: intersection with installed registry and user/security permissions.
- Actions: explicit allowlist + handler availability; deny wins.
- Data access: service-level grants, never inferred from merely being in a domain.
- Semantic mappings: most specific exact match wins, but two equally specific matches cause a needs-specificity result.
- Historical UI: use its pinned compatible template/convention version rather than silently changing meaning.

All merge behavior should be testable and documented; do not use an opaque last-write-wins JSON merge for authority-sensitive fields.

## Local semantic example

Agent request:

~~~json
{
  "schema": "ui.request.v1",
  "kind": "semantic",
  "intent": "review",
  "target": "pending_orders"
}
~~~

The **shop context** may map it to OrderReview with current authorized data. The **CRM context** has no matching target and should return unsupported, not guess that pending orders means pending deals.

A similarly generic request with target **items** might have multiple valid domain mappings. The resolver must ask for a distinction or pick only when a documented default unambiguously applies.

## Machine-readable capability projection

The FrameContext can be large. The model-facing projection can be as small as:

~~~json
{
  "projectionVersion": "cp-19",
  "scope": "shop.orders",
  "intents": ["review", "compare", "filter", "select"],
  "targets": ["pending_orders", "inventory"],
  "preferredInput": ["SingleChoice"]
}
~~~

Do not put all registered React components, CSS tokens, workflow scripts or hidden resource locations into the agent context. Keep a projection hash and reuse an unchanged projection across turns where the model/runtime supports such caching. When a feature is not projected, a specific discovery step can surface its relevant contract.

## Privacy and trust boundary

Environmental context is not a blank check. Central storage may hold sensitive documents and personal/financial data; the frame can know a resource category exists without showing its contents or exposing it to an agent. Separate:
- Discoverability of a capability
- Authorization to read data
- Authorization to mutate data
- User confirmation required for consequential actions

An agent-visible projection should disclose only what that agent is entitled to use. The host service must enforce permissions even if a client or model spoofs an action ID.

## Invalidation and offline behavior

Cache compiled manifests by content hash and schema version. Update the effective FrameContext when relevant manifest versions, domain/conversation route, permission state or device capabilities change. Avoid rereading vault files on every message render.

Offline operation may use an already-approved cached manifest and preloaded, non-sensitive data. Remote operations that require live authority should fail closed or visibly queue where allowed. Do not treat cached permissions as permanent.

## Test cases

1. Same semantic request resolves to distinct registered templates in distinct domains where explicitly mapped.
2. A denied action remains denied even when an agent manifest lists it.
3. A manifest edit updates future UI without corrupting pinned old messages.
4. A missing/malformed domain file does not crash the frame; fallback is bounded and visible.
5. A capability projection excludes confidential data even though FrameContext knows the resource exists.
6. Cached manifest changes invalidate the correct resolver state but not unrelated message content.
