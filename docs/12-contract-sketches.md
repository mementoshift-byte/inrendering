# 12 · Candidate data contracts (TypeScript sketches)

**Status:** **illustrative designs, not final stable API**. Use these to facilitate implementation discussion and tests.

## 1. Context types

~~~ts
export type FrameContext = {
  contextId: string;
  domainId: string | null;
  conversationId: string;
  conversationKind: "main" | "general" | "agent" | "workflow";
  agentId?: string;
  conventionVersion: string;
  registryVersion: string;
  presentation: {
    density: "compact" | "comfortable";
    themeId: string;
    composerMode: "ordinary" | "augmented";
  };
  capabilities: {
    componentIds: string[];
    semanticIntents: string[];
    actionIds: string[];
  };
  dataBindings: Record<string, {
    resourceId: string;
    revision?: string;
    accessScope: string;
  }>;
};

export type CapabilityProjection = {
  projectionId: string;
  scope: string;
  intents: string[];
  targets: string[];
  inputKinds: string[];
};
~~~

Note: component IDs in a context are not sufficient for authorization. Application handlers must independently enforce permissions.

## 2. Canonical message and blocks

~~~ts
export type MessageEnvelope = {
  id: string;
  conversationId: string;
  role: "user" | "assistant" | "tool" | "system";
  revision: number;
  createdAt: string;
  status: "streaming" | "complete" | "failed" | "cancelled";
  blocks: MessageBlock[];
};

export type MessageBlock =
  | { kind: "text"; id: string; revision: number;
      value: string; format: "plain" | "markdown" }
  | { kind: "ui"; id: string; revision: number;
      request: UIRequest; conventionVersion: string }
  | { kind: "resource"; id: string; revision: number;
      resourceId: string; mediaType: string }
  | { kind: "status"; id: string; revision: number;
      label: string; severity: "info" | "warning" | "error" };
~~~

## 3. UI request and tree

~~~ts
export type SemanticUIRequest = {
  schema: "ui.request.v1";
  kind: "semantic";
  requestId: string;
  intent: string;
  target: string;
  params?: Record<string, unknown>;
};

export type TreeUIRequest = {
  schema: "ui.request.v1";
  kind: "tree";
  requestId: string;
  tree: UITree;
};

export type UIRequest = SemanticUIRequest | TreeUIRequest;

export type UITree = {
  schema: "ui.tree.v1";
  treeId: string;
  root: UINode;
};

export type UINode = {
  id: string;
  type: string;
  props?: Record<string, unknown>;
  actions?: Record<string, string>;
  children?: UINode[];
};
~~~

The registry provides the typed props/children grammar per component. The raw broad type above is an envelope; **runtime schema validation is mandatory**.

## 4. Resolver result

~~~ts
export type ResolveResult =
  | { kind: "resolved";
      tree: UITree; templateVersion: string;
      contextVersion: string }
  | { kind: "needs-specificity";
      reason: string; validOptions?: string[] }
  | { kind: "denied"; reason: string }
  | { kind: "unsupported"; reason: string };
~~~

No invisible fallback that changes a user's intended operation into a different one.

## 5. Content references

~~~ts
export type TextAnchor = {
  kind: "text";
  referenceId: string;
  conversationId: string;
  messageId: string;
  blockId: string;
  blockRevision: number;
  range: {
    start: number;
    end: number;
    unit: "unicode-codepoint";
    canonicalizer: "text-v1";
  };
  quote: string;
  contextBefore?: string;
  contextAfter?: string;
};

export type StructuredAnchor = {
  kind: "structured";
  referenceId: string;
  conversationId: string;
  messageId: string;
  blockId: string;
  nodeId: string;
  selector:
    | { kind: "cell"; rowId: string; columnId: string }
    | { kind: "point"; seriesId: string; pointId: string }
    | { kind: "field"; fieldId: string };
};

export type Reference = TextAnchor | StructuredAnchor;
~~~

**Warning:** Unicode codepoint offsets are only a *candidate*. DOM selection normally uses UTF-16 code units. Adopt the unit only after spike tests and implement a correct conversion layer.

## 6. Typed events and submissions

~~~ts
export type ChatEvent =
  | {
      kind: "reference.add";
      conversationId: string;
      reference: Reference;
    }
  | {
      kind: "reference.remove";
      conversationId: string;
      referenceId: string;
    }
  | {
      kind: "ui.action";
      conversationId: string;
      instanceId: string;
      actionId: string;
      value: unknown;
    }
  | {
      kind: "composer.submit";
      conversationId: string;
      text: string;
      references: Reference[];
      inputs: Array<{
        requestId: string;
        componentId: string;
        value: unknown;
      }>;
      attachmentIds: string[];
    };
~~~

A production action dispatcher should add event IDs/idempotency keys, actor identity, validation and authorization. The agent is not the direct destination for every ChatEvent.

## 7. Logical frame interface

~~~tsx
type ChatFrameProps = {
  conversationId: string;
  context: FrameContext;
  messages: readonly MessageEnvelope[];
  onEvent: (event: ChatEvent) => void;
  header?: React.ReactNode;
};

export function ChatFrame(props: ChatFrameProps) {
  // Header + independent scroll viewport + overlay root.
  // MessageRenderer is a dependency, not an agent backend.
  return null;
}
~~~

The code above is deliberately not an implementation. It illustrates **ownership and direction of dependency**. This interface might evolve to use an external store rather than pass all messages as an array.

## 8. Minimal end-to-end serialized example

Agent intent:

~~~json
{
  "schema": "ui.request.v1",
  "kind": "semantic",
  "requestId": "req-21",
  "intent": "review",
  "target": "pending_orders"
}
~~~

FrameContext:
- domainId = shop
- semantic mapping = review/pending_orders -> order-review.v2
- data binding = permitted orders dataset
- installed component registry = includes OrderReview

Local resolver returns a normalized UITree. User selects an order, application emits an authorized action event. No model call is needed for selecting the row. Only a request for further reasoning or missing data causes an agent turn.

## 9. Stability cautions

- Do not silently change schema meanings in place; use versions/migrations.
- Avoid serializing React nodes or callback functions across the wire.
- Keep UI instance identity independent of component template identity.
- Never infer user permissions merely from the FrameContext serialized by a client.
- Record canonical text/selection conventions before persisting real references.
