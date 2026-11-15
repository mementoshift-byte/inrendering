# 06 · Precise selections, references and highlighting

**Status:** core user requirement; exact anchor encoding remains a design/spike decision.

## Why this is a first-class subsystem

The goal is not a “quote one message” feature. The user reads a long agent response and wants to react *at the point of encounter* rather than continually scrolling to the bottom and reconstructing thoughts from memory.

A user can:
1. Read paragraph 2, select an exact phrase.
2. Continue to paragraph 8, select a different fragment.
3. Notice a contradiction in paragraph 14 and select that statement.
4. Compose one response with all three independently addressable references.

A reference may also target a cell in a generated table or an individually addressable chart point. Selection needs **precision**: selecting a component alone is inadequate when the user intends to reference three words within it.

## Three operations that must not be conflated

- **Selection:** user interaction specifying exact content.
- **Reference:** durable, typed description of the selected content and its source.
- **Highlight:** a visual rendering of active/persisted references.

A highlight is not the source of truth. DOM Range and pixel coordinates are transient input primitives, not durable reference IDs.

## Reference model sketch

~~~ts
type TextReference = {
  kind: "text";
  id: string;                 // reference instance
  conversationId: string;
  messageId: string;
  blockId: string;
  blockRevision: number;
  selector: {
    start: number;
    end: number;              // exclusive
    offsetUnit: "unicode-codepoint"; // proposed; choose after cross-platform spike
    canonicalization: "text-v1";
  };
  quote: string;              // captured selected text for validation/display
  quoteHash?: string;
  surroundingText?: {
    prefix?: string;
    suffix?: string;
  };
};

type StructuredReference = {
  kind: "structured";
  id: string;
  conversationId: string;
  messageId: string;
  blockId: string;
  nodeId: string;
  resourceId?: string;
  address:
    | { kind: "table-cell"; rowId: string; columnId: string }
    | { kind: "chart-point"; seriesId: string; pointId: string }
    | { kind: "field"; fieldId: string };
  contentRevision?: string;
};
~~~

**Offset-unit choice is not settled.** JavaScript DOM offsets are typically measured in UTF-16 code units; Unicode code points and grapheme clusters behave differently. The selection adapter must translate correctly and store the chosen unit explicitly. Markdown rendering creates additional complexity because source offsets may differ from visible text offsets.

## Canonical selectable content

Choose a canonical text representation for each text block, then make rendering preserve an explicit mapping between that representation and visible text nodes.

Potential options:
- Plain-text canonical content with a mapping from rendered markdown spans.
- A rich-text AST with stable leaf IDs and offset ranges.
- Document-editor-style position anchors.

Do not assume directly taking DOM Range.startOffset/endOffset is sufficient: they refer to individual DOM text nodes, not necessarily the whole canonical block. Hidden decorations, inline components and markdown syntax can invalidate direct offsets.

A robust extraction flow:
1. User selects a visible range.
2. Identify the closest addressable block and leaf spans.
3. Map rendered text positions to canonical range(s).
4. Verify extracted canonical text equals the displayed quote under normalization rules.
5. Produce one or more typed references.
6. Store them independently of viewport/DOM lifecycle.

## Non-contiguous references

The normal browser selection API primarily exposes the active selection and does not automatically provide a persistent multi-message selection collection. The controller should maintain its own **reference set**, with add/remove/reorder/dedupe operations. A new selection **adds** a reference without erasing earlier references.

References can span different messages and even different content types; one text selection should not force all others into a single continuous range.

## Streaming and edits

Every persistent reference must carry enough revision evidence to detect change.

Possible outcomes after a text edit:
- **Exact:** selected range remains valid in the same revision.
- **Reanchored and verified:** offsets changed but quote/surrounding context uniquely identify the content.
- **Stale/unresolved:** reference cannot be uniquely relocated.

Never silently reanchor to a different identical phrase in the wrong context. Present stale state explicitly and retain the captured quote as historical context.

For a block still streaming, either mark references provisional or constrain selection to committed spans. This needs empirical testing with the chosen runtime and rich-text parser.

## Interaction lifecycle

~~~text
Native text or structured-object selection
   -> selection adapter
   -> canonical typed reference
   -> per-conversation ReferenceStore
   -> visual highlights + composer reference tray
   -> optional domain-specific actions
   -> submission payload when user sends
~~~

The reference store can retain multiple independent references while the user edits their draft. The composer should display enough quote/source context to disambiguate them, without forcing the full message text into the compose field.

## Domain-specific operations

A single reference protocol supports local actions such as:
- Add/remove reference
- Quote selected content
- Annotate
- Challenge/compare (agent round trip when necessary)
- Investigate selected structured data in an authorized domain workflow

Domain conventions determine which actions are offered, but never change the meaning of an existing text anchor. The action dispatcher decides whether an operation stays on-device or must reach a service/agent.

## Cross-device behavior

Use content identities, not screen coordinates. The same reference can be rendered at a different viewport width or on another device. Sync of durable references is separate from ephemeral selection-drag gestures.

## Failure scenarios and acceptance

- Select exactly one word including punctuation, not its whole paragraph.
- Select multiple non-adjacent spans from distinct messages.
- Navigate away and back: all saved references and draft remain.
- Resize text or reflow content: highlights target the same logical content.
- Select in Markdown with bold/links; text offset mapping stays exact.
- Select a table cell, then a text span; both attach to the same draft.
- Update a source block; a stale reference does **not** silently become a different quote.
- A mobile touch selection does not become unusable because a floating overlay blocks system handles.

## Main engineering risk

This is likely the hardest piece to retrofit. Design canonical content identity and range mapping **before** choosing a message renderer solely for its visual appearance.
