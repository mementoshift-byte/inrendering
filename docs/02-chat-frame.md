# 02 · ChatFrame and physical interaction surfaces

**Status:** core product requirement; specific implementation rules are proposed.

## The frame is an interaction environment

The chat frame is **one component embedded in the wider application**, not the entire app. It is responsible for everything required for a comfortable conversation surface: a clear viewport, stable messages, a composer overlay, contextual controls, selections and restoration when returning from other screens.

The frame does **not** own top-level domain tabs, shop navigation, file services or agent orchestration.

## Physical layout in the Figma reference

Two source frames:

- **Command Center · Main Chat**, node **5:541**.
- **Command Center · Shared conversation**, node **5:260**.

Both are mobile design frames of approximately **402 × 884** px. Both depict a **118 px conversation header**, a **674 px message viewport**, and a **92 px composer/home-indicator region** within the artboard. These are snapshot measurements, **not hard-coded production viewport dimensions**. Native operating-system status and safe-area behavior need appropriate handling.

The main conversation uses a symbolic MAIN avatar and general identity. The shared conversation uses an agent avatar, domain/agent/status subtitle and domain-specific message content. The outer frame pattern should be identical.

## Suggested component structure

~~~text
ChatFrame
  ├─ ConversationHeader (provided by host or frame slot)
  ├─ ConversationViewport
  │    ├─ VirtualizedMessageList (optional after measurement)
  │    ├─ AgentMessage / UserMessage
  │    │    └─ MessageRenderer -> TextPart | UIPart | ...
  │    └─ DateDivider / StatusPart
  ├─ OverlayRoot
  │    ├─ SelectionToolbar
  │    ├─ ReferenceTray
  │    ├─ ContextMenu
  │    └─ ComposerOverlay
  └─ Interaction / scroll / continuity controllers
~~~

The overlay root is structurally separate from the message list. The composer can float visually over the conversation without becoming a message item. Use normal flow for messages; only the necessary overlays are positioned.

## Responsibilities in detail

### Conversation viewport

- Own scroll container and scroll anchor state.
- Maintain orientation during image/component height changes.
- Support stick-to-bottom behavior only when the user is near the bottom; do not yank them down while they are reading earlier material.
- Support progressive message arrival without remounting all previous turns.
- Offer a stable host for text selections.
- Support responsive width and safe-area insets rather than relying on Figma coordinates.

### Distinct message presentations

Assistant content is generally left-aligned, with prose that can occupy the available width. User turns are right-aligned in rounded bubbles. A domain-specific assistant message can include attention cues, data summaries, cards and interactive blocks. The **role determines presentation defaults**, not a different chat engine.

### Composer overlay

- Anchored to the bottom frame region, independent of scroll content.
- Supports ordinary text input, attachments, voice/camera triggers as allowed.
- Supports **agent-requested input** using the same component interpreter as messages.
- Shows an independently managed tray of selection references and pending structured values.
- Can expand for large input without destroying the underlying conversation state.
- Keyboard opening should resize or adapt the usable viewport without losing the current scroll anchor.

### Selection UI

Selection menus can be anchored near visible selected text, but the underlying selection identity must not depend on the menu's screen position. The user may accumulate references across non-adjacent text and messages. Avoid trapping the native selection gesture merely to display a custom toolbar.

### Cross-conversation continuity

When switching MAIN <-> domain chat <-> files/repo/dashboard <-> back:

- Restore each conversation's scroll location (anchor ID + relative position preferred to raw pixel offset).
- Restore each conversation's draft.
- Restore its selected references, expanded UI regions and pending form state.
- Do not leak MAIN draft/selection state into another conversation.
- Do not let arrival of fresh messages automatically discard the remembered reading position.

## Proposed public interface

~~~tsx
<ChatFrame
  conversationId={conversationId}
  context={resolvedFrameContext}
  messageSource={messageStore}
  onEvent={dispatchChatEvent}
  header={headerElement}
/>
~~~

This is an illustrative composition API. It should remain free of assumptions about the global navigation tabs or where conversations are persisted.

## Frame state hierarchy

**Ephemeral:** hover, currently focused control, transient popovers, in-progress native DOM selection.

**Per-conversation persisted locally:** draft, selected references, scroll anchor, component expansion, unfinished form values.

**Canonical remote or synced:** messages, attachments, durable references and agreed workflow effects when backed by central storage.

Not every ephemeral UI detail needs to be synchronized across devices.

## Test cases before backend hookup

1. Mount MAIN and shared conversation simultaneously or toggle repeatedly; each retains its own state.
2. Render very long agent prose and several interactive blocks; composer never jumps into message flow.
3. Start reading at turn 20 while new tokens arrive at turn 50; viewport remains stable.
4. Select two distinct text spans, scroll away, and display both in reference tray.
5. Resize from narrow mobile to desktop width; text reflows and content references remain valid.
6. Open a domain-specific input in the composer and return to ordinary typing without losing draft text.
7. Keyboard, screen reader and touch controls remain usable.

## What this document does not decide

Whether to use assistant-ui's viewport primitives or a custom scroll container is a **spike decision**. The boundary and behavior above are the requirements; the internal library is interchangeable.
