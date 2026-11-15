# 07 · Agent-specified input, action routing and UI state

**Status:** bidirectional UI capability confirmed; exact lifecycle is proposed.

## Input is a shared UI surface

The user explicitly wants an agent to specify an interactive input experience, using **prebuilt local React components**. That input may appear in the composer overlay or inline in a message. It is not a request that the agent generate an entirely new implementation on every turn.

The same ComponentRenderer can mount either location. The composer itself remains a persistent conversation-level surface and should **not** be destroyed just because an agent proposes a new input.

## Distinct input situations

**Ordinary composer:** freeform text and optional attachments.

**Augmented composer:** text remains available; agent requests a known choice control, slider, table edit, checklist or domain-specific input component to collect typed values.

**Inline input:** a message contains an action or form whose lifecycle is tied to that message instance.

**Workflow input:** a recurring, domain-specific interaction can display locally from a cached template and submit structured values into a workflow action.

**Generated code input:** potential future fallback; not a version-one requirement and not executed in the privileged React tree.

## Sample agent request

~~~json
{
  "schema": "ui.input.v1",
  "requestId": "req-12",
  "targetSurface": "composer",
  "component": "SingleChoice",
  "props": {
    "title": "Choose execution mode",
    "options": [
      { "id": "manual", "label": "Manual" },
      { "id": "automatic", "label": "Automatic" }
    ]
  },
  "action": "mode.select"
}
~~~

This request does not include JavaScript callbacks, API URLs or CSS. The client knows the implementation, state rules, validation and visual conventions. Unrecognized or unauthorized requests receive a controlled fallback.

## Typed event bus

Every interactive component should emit an event into a shared action dispatcher, not directly into an agent orchestration implementation.

~~~ts
type ChatUIEvent = {
  id: string;
  kind:
    | "ui.value.changed"
    | "ui.action.requested"
    | "reference.added"
    | "reference.removed"
    | "composer.submitted";
  conversationId: string;
  messageId?: string;
  nodeId?: string;
  actionId?: string;
  payload?: unknown;
  occurredAt: string;
};
~~~

This sketch is intentionally generic. Production event schemas should use discriminated unions with typed payloads and strict validation.

## Routing policy

1. **Local UI action:** focus, expand/collapse, set slider, change tab, maintain selection, edit input. No LLM call.
2. **Local application action:** open cached details, filter a local dataset, update an authorized local preference. No LLM call unless explicitly required.
3. **Backend action:** save a file, schedule a workflow, update an order, mutate configuration; goes through an authorized service handler.
4. **Agent turn:** user explicitly asks for reasoning, transformation or new information. Compose the minimal context needed and send it to the chosen agent.
5. **Consequential action:** requires service-level permission checks and possibly explicit user confirmation. A component being visible is not authorization.

Action IDs are registered capabilities, not arbitrary executable method names. The service validates actor, scope, action and payload.

## Input and composer state

Separate:
- **Text draft**: per conversation.
- **Reference attachments**: per conversation, independent of text.
- **Inline form state**: per message and UI instance.
- **Composer-proposed form state**: per conversation + request instance, preserving active input during navigation.
- **Pending remote actions**: explicit status/idempotency, not just a loading spinner.
- **Transient state**: focus, hover, temporary menus.

Use stable UI instance IDs. Reusing a template does not mean reusing its values.

## State lifecycles

A single requested input should have visible states such as:
- Pending / offered
- Active
- Completed
- Cancelled / dismissed
- Expired or no longer valid

The application must define what happens if a new agent request arrives while an earlier form is unfinished. Suggested default: do not erase an active draft; keep multiple requests addressable or queue/show a conscious replacement choice.

A server update or domain switch should not unexpectedly destroy draft content. Persist sufficient state to restore on return.

## Payload on submission

A compose event can carry:
- Text body
- Selected exact references
- Structured component values
- Attachment IDs
- Conversation/agent routing metadata

The prompt builder decides what to expose to the model. The frame should not blindly dump its full cached state, Obsidian manifests, financial file metadata or UI tree into the prompt.

~~~json
{
  "conversationId": "crm.pipeline",
  "text": "Compare these two statements",
  "references": ["ref-7", "ref-11"],
  "inputValues": [
    { "requestId": "req-12", "component": "SingleChoice", "value": "automatic" }
  ]
}
~~~

This is an application-level event shape, not necessarily the final model message format.

## Accessibility and feedback

Controls must support keyboard navigation, touch targets, focus recovery, screen reader labels, validation errors and disabled/loading/success states. Avoid generating components whose controls look interactive but have no registered action. If a remote action times out, show an explicit retry/failure state and preserve user input.

## Tests

- Change a choice five times locally: zero agent round trips.
- Switch chats mid-form: return to the same values and references.
- Two form instances of the same template are independent.
- Unknown action ID cannot mutate application data.
- Submit with structured values and precise references; both are present in the typed event.
- Agent requests new input while user is typing: draft is preserved.
- Remote mutations are idempotent or guarded against duplicate sends.
