# 04 · Constrained component engine and UI tree language

**Status:** key architecture direction, with proposed syntax.

## Objective

Build a **component rendering engine above/under the message presentation boundary**: conceptually the MessageRenderer delegates UI blocks to it; technically it is a separate reusable interpreter. It should resolve UI meaning using prebuilt React components and local conventions rather than requiring the agent to generate bespoke JSX.

**The component tree exists to constrain ambiguity, not just serialize JSX.**

The important distinction is between:
- **Semantic request**: what interaction/presentation is needed.
- **Component tree**: an explicit, constrained composition of known nodes.
- **React implementation**: locally compiled and maintained code.

## Execution levels

| Level | Payload | Client behavior |
|---|---|---|
| L0: direct component | Known component ID + props/data ref | Resolve and mount a registered component |
| L1: semantic intent | e.g. review pending orders | Resolve domain mapping to a registered template |
| L2: constrained tree | Typed nodes, props and action IDs | Validate and compose from registry |
| L3: cached custom definition | Template ID/version + params | Load cached, approved tree or prebuilt bundle |
| L4: generated code | New executable JSX/component | Exceptional **isolated** workflow, never directly in privileged app tree |

The ordering is a policy, not a universal cost ranking. A cached resolved semantic request is often cheaper than reserializing even a small tree.

## Resolution pipeline

~~~text
incoming UIRequest
      │
      ▼
Validate envelope + schema version
      │
      ▼
Resolve current / pinned FrameContext
      │
      ▼
Apply explicit semantic convention mappings
      │
      ▼
Resolve template or normalize explicit UI tree
      │
      ▼
Validate component types, props, children, actions, limits
      │
      ▼
Canonical IR + structural hash
      │
      ▼
Component registry -> React tree + state/action bindings
~~~

Do **not** allow free-form instructions in a component's props to override security policy, call arbitrary functions or alter application resources.

## Sketch of semantic request

~~~json
{
  "schema": "ui.request.v1",
  "kind": "semantic",
  "intent": "review",
  "target": "pending_orders",
  "requestId": "ui-072"
}
~~~

In the shop domain, an explicit mapping may bind this to a prebuilt OrderReview template using permitted order data. In a repository domain, a request for review with a different target can map to a code review. If the mapping is ambiguous, the resolver must **not** guess silently.

## Sketch of explicit UI tree

~~~json
{
  "schema": "ui.tree.v1",
  "treeId": "tree-13",
  "root": {
    "id": "root",
    "type": "Stack",
    "props": { "gap": "md" },
    "children": [
      {
        "id": "question",
        "type": "Text",
        "props": { "value": "Which mode?", "variant": "heading" }
      },
      {
        "id": "answer",
        "type": "SingleChoice",
        "props": {
          "options": [
            { "id": "manual", "label": "Manual" },
            { "id": "automatic", "label": "Automatic" }
          ]
        },
        "actions": { "submit": "execution_mode.choose" }
      }
    ]
  }
}
~~~

The agent specifies meaningful distinctions: question, options and requested action. The client supplies spacing, colors, input behavior, focus and layout rules.

## Grammar and registry

A component registration should declare:

- Stable name and version, plus typed prop schema.
- Allowed children/slot grammar and maximum recursion/collection constraints.
- Optional semantic capabilities: selectable, referenceable, editable, expandable.
- Known action contracts and state policy.
- Optional platform adapter (React web/React Native) and fallback.
- Accessibility requirements and supported data binding types.
- Whether the component needs local data, remote data or both.

**Primitive nodes** such as Text, Stack, Row, Action and Input give flexibility. **Semantic composites** such as SingleChoice, DataTable or ReviewPanel reduce description cost. **Domain components** such as CandidateReview/OrderReview are reusable prebuilt views with restricted capabilities.

Small building blocks do *not* imply asking the agent to emit hundreds of tiny nodes. A locally expanded composite is preferred for common patterns.

## Determinism and disambiguation

An intent resolver returns one of:
1. **Resolved**: one valid semantic mapping + versioned template + bindings.
2. **Needs details**: multiple valid mappings / missing necessary parameters.
3. **Denied**: capability or permission not available.
4. **Unsupported**: grammar or component not installed.

The resolver should not infer authority from location, and must not invent data or change workflow semantics simply to choose a visually pleasing component.

## Tree normalizer

A normalization stage applies:
- Stable default props and theme tokens.
- Explicit responsive constraints.
- Canonical ordering of serializable props for hashing.
- Expansion of registered semantic aliases.
- Known action binding syntax and typed data references.
- Schema migration and version pinning.

Normalization must not silently change the user-meaningful choices. Distinguish **structural template hash** from **instance identity** and **data resource revision**: identical forms may share a template but keep separate input state.

## Registry interpreter outline

~~~tsx
function RenderNode({ node, context, dispatch }: NodeProps) {
  const entry = registry.lookup(node.type, context.registryVersion);
  if (!entry) return <UnsupportedNode nodeId={node.id} />;

  const props = entry.schema.parse(node.props);
  const Component = entry.webComponent;

  return (
    <Component {...props} nodeId={node.id} dispatch={dispatch}>
      {node.children?.map(child => (
        <RenderNode
          key={child.id}
          node={child}
          context={context}
          dispatch={dispatch}
        />
      ))}
    </Component>
  );
}
~~~

Illustrative only. In production: validate the **entire tree** first, enforce depth/node count/allowed children, reject unsupported actions and prevent untrusted content from becoming executable props.

## Generated JSX policy

Generated executable JSX was considered as a possible flexibility tier, not a commitment to use it. If ever introduced:
- Treat it as untrusted code.
- Build and validate in a sandbox with a narrow capability bridge.
- Do not let it import privileged app modules, read the filesystem or execute arbitrary actions.
- Cache only reviewed, versioned and appropriately scoped artifacts.
- Prefer learning a new approved semantic template from repeated successful use.

## Cross-device principle

The **UI request and semantic contracts** can be shared across platforms. Physical rendering components may differ between web and React Native. Do not encode DOM-specific pixel coordinates as the meaning of an interaction.

## Tests

- Same request + same manifest/data versions resolves to the same normalized tree.
- Unknown component yields a local error component, not arbitrary rendering.
- Unsupported prop, nested child or unapproved action is rejected.
- Two instances of the same cached template retain separate state.
- Switching domain changes permitted presentation but not historical data identity.
- Generic review request with two equally valid mappings asks for specificity.
