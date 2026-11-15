# 03 · Message model and MessageRenderer

**Status:** proposed canonical contract that captures confirmed product behavior.

## A message is not just a text string

The MessageRenderer should consume typed, ordered content blocks. A message has identity and provenance, and blocks have identities and revisions. A human-visible turn may contain prose, a table, an interactive UI request, a file attachment, a warning/status segment, or data produced by tools and workflows.

**AgentMessage** and **UserMessage** are different presentation surfaces. They can share the same block renderer and reference model.

## Suggested envelope

~~~ts
type Role = "user" | "assistant" | "tool" | "system";

type MessageEnvelope = {
  id: string;
  conversationId: string;
  role: Role;
  createdAt: string;
  revision: number;
  status: "streaming" | "complete" | "failed" | "cancelled";
  blocks: MessageBlock[];
  metadata?: {
    agentId?: string;
    domainId?: string;
    source?: string;
  };
};

type MessageBlock =
  | { id: string; kind: "text"; revision: number; text: string; format: "plain" | "markdown" }
  | { id: string; kind: "ui"; revision: number; spec: UIRequest; conventionVersion: string }
  | { id: string; kind: "attachment"; revision: number; resourceId: string; mediaType: string }
  | { id: string; kind: "status"; revision: number; label: string; level: "info" | "warning" | "error" };
~~~

These are **design sketches**. Role values and the precise discriminated union must be reconciled with whichever runtime adapter we adopt. In particular, do not assume tool responses should appear as standalone human conversation turns.

## Rendering pipeline

1. Accept a canonical message envelope from the store or runtime adapter.
2. Render its role-specific container without changing its content semantics.
3. Map each ordered block to a known part renderer.
4. Defer UI specifications to the component interpreter rather than letting the message renderer implement domain components.
5. Ensure existing block instances preserve stable React keys across updates.
6. Surface explicit placeholders/errors for unresolved or invalid content.

A failed or unsupported block should not take down the rest of the message tree.

## Stable identity and revisions

- **Conversation ID:** uniquely identifies the session/workspace conversation.
- **Message ID:** stable across streaming, rendering, navigation and sync.
- **Block ID:** identifies addressable content within a message. Changes to unrelated blocks should not invalidate it.
- **Block revision:** changes when the logical content of that block changes.
- **Node ID:** identifies a component within a UI tree.
- **Convention version:** binds the interpretation of a persisted UI specification.
- **Data revision:** identifies the referenced resource revision separately from the UI tree.

Never derive canonical IDs from DOM paths, row order or visual positions. A content hash is useful for deduplication but cannot replace stable identity when two identical text blocks occur.

## Streaming semantics

Streaming needs a protocol, not ad-hoc string concatenation in React. Normalize upstream events into operations such as:

~~~ts
type MessagePatch =
  | { op: "message.open"; messageId: string; role: Role; sequence: number }
  | { op: "block.add"; messageId: string; block: MessageBlock; sequence: number }
  | { op: "text.append"; messageId: string; blockId: string; text: string; sequence: number }
  | { op: "block.replace"; messageId: string; block: MessageBlock; sequence: number }
  | { op: "message.complete"; messageId: string; sequence: number };
~~~

A store must handle duplicate events and ordering; whether to buffer or reject out-of-order patches depends on the transport. On reconnection, prefer a canonical snapshot plus sequence cursor instead of blindly replaying ambiguous chunks.

**Selection rule:** until a text block is finalized, references into that block require revision-aware handling. Appending text after a selected range may be safe; replacing earlier text may not be. Do not silently attach a persisted quote to a different sentence.

## Rendering and caching

- Cache parsed rich text by block ID, content revision and parser version.
- Render only changed blocks; avoid remounting existing interactive UI when a sibling text block streams.
- Use controlled limits on huge tables, trees and attachment previews.
- Virtualize long timelines only if it is compatible with reliable text selection and scroll restoration.
- Preserve block order as part of the content contract, not inferred from viewport position.
- Sanitize user/agent-generated markdown and prevent arbitrary HTML/script execution.

## Content addressing

Exact user selections should attach to **canonical block content**, not to the rendered DOM string that may include decorations, hidden nodes or changed whitespace. The text pipeline should define a canonical selectable representation, with offset encoding and normalization specified by the reference engine.

For structured blocks, each component adapter can expose semantic addresses such as row ID + column ID or data series + point ID.

## Handling older messages after convention changes

A saved UI block should carry enough schema/manifest information to render the same intended interface or an explicit migrated equivalent. Do not blindly reinterpret historical messages under a new meaning of a generic semantic action.

Two possible strategies require validation:
- Pin a saved expanded component tree and convention version.
- Pin the semantic request plus its resolved template version, then use a versioned migration path.

## Integration rules

The MessageRenderer accepts plain data and context. It does not directly call an LLM, query Obsidian per block, select which domain is open, trigger remote actions or expose private data. The runtime adapter handles upstream mechanics and the interaction controller handles outbound events.

## Acceptance tests

- A streaming message adds blocks without changing previous IDs.
- An invalid UI block displays a local failure state; following text still renders.
- Two identical text blocks can be selected independently.
- An old message remains interpretable after domain settings update.
- The message tree and composer states survive a conversation switch.
- A full message snapshot produces the same visible result as equivalent ordered patches.
