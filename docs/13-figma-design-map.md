# 13 · Figma design extraction map

**Status:** reference inventory from direct read of the supplied design file on 2026-11-15.

Design: [Command Center · Untitled](https://www.figma.com/design/v17vwmp74ZbPpQwnVLelfB/Untitled?node-id=0-1)

## What is actually present

The file contains seven prominent mobile artboards/screens:

| Screen | Figma node | Architectural relevance |
|---|---|---|
| **Command Center · Main Chat** | **5:541** | MAIN continuous conversation appearance |
| **Command Center · Shared conversation** | **5:260** | Agent/domain conversation appearance |
| **Command Center · Files** | **7:436** | Central directory UI, future Obsidian entry |
| **Command Center · Website** | **3:535** | Domain dashboard/example |
| **Command Center · CRM** | **3:447** | Domain dashboard/example |
| **Command Center · Gmail** | **3:349** | Domain dashboard/example |
| **Command Center - Linkedin** | **2:192** | Domain home tabs and grouped agent/workflow chats |

These designs are **references**, not evidence that the corresponding backend features already exist. Shop overview and repository dashboards were described as intended additional destinations; they should not be claimed to already appear in the current Figma file.

## Primary extraction targets

### Main Chat, node 5:541

[Open Main Chat in Figma](https://www.figma.com/design/v17vwmp74ZbPpQwnVLelfB/Untitled?node-id=5-541)

- Artboard: **402 × 884**.
- Conversation header **5:542**, approximately 118 px high.
- Header/toolbar **5:552**: back control, MAIN icon, conversation identity and options.
- Message region **5:562**, approximately 674 px high in the mockup.
- Assistant turns **5:566**, **5:581**, **5:593**: full-width, left-aligned prose.
- User turns **5:574**, **5:586**: right-aligned purple rounded message bubbles.
- Date dividers **5:563**, **5:571**.
- Composer **5:598**, approximately 92 px including home indicator; input **5:602**, attachment/camera/voice controls.

### Shared Conversation, node 5:260

[Open Shared Conversation in Figma](https://www.figma.com/design/v17vwmp74ZbPpQwnVLelfB/Untitled?node-id=5-260)

- Artboard: **402 × 884**.
- Conversation header **5:261**, approximately 118 px high.
- Conversation toolbar **5:271** with agent avatar **5:273** and identity **5:274**.
- Message region **5:281**, approximately 674 px in mockup.
- Assistant turn **5:285** has a **Needs attention** cue.
- Assistant turn **5:300** includes a structured, inline three-deal summary.
- User turns **5:294**, **5:315** use same message bubble pattern as MAIN.
- Composer **5:326**, same general layout family.

**Extraction conclusion:** one shared frame and message composition system should express both screens. MAIN/agent identity and the actual message content differ; the frame mechanics should not fork.

## Wider navigation relevant to boundaries

### LinkedIn home, node 2:192

This screen demonstrates **top domain tabs** (LinkedIn/Gmail/CRM/Website), and **General / Agents / Workflows** groupings within a domain. It also includes bottom navigation and a middle MAIN control. Those belong to the Command Center application shell.

### Files screen, node 7:436

The Files page shows a directory structure, folder/document rows and bottom navigation. In the proposed architecture, this is an application surface for central file storage and editing/managing Obsidian-backed setup. It is **not** part of the ChatFrame.

## Visual tokens observed in the design-context output

These are *reference values*, not final design-system decisions:

| Element | Observed value |
|---|---|
| Background | #252B2A |
| Header and composer surface | #2B3130 |
| Subtle borders | #3A4340 |
| Main text | #E7EBE8 |
| Secondary text | #AEB7B3 |
| User message bubble | #39336C |
| Microphone/primary purple | #4338CA |
| Body text | Inter ~15 px, line-height ~21 px |
| User message radius | ~18 px |
| Message-area horizontal padding | ~16 px |

Status bar/home-indicator elements in the 402×884 mockup are presentation references. On actual devices they may be supplied by the operating system or host shell. Do not duplicate OS chrome blindly.

## Design intent beyond the static mockup

The user subsequently specified behavior that is **not visually depicted fully in these artboards**:

- Composer and contextual selection controls are overlays relative to the frame.
- Exact, non-contiguous highlighted references can be accumulated before responding.
- Agent-specified structured input may be rendered by the composer.
- Domain conventions influence local component resolution.
- The frame must preserve scroll, drafts and references across navigation.
- The component engine can render complex, domain-specific UI inside messages.

Do not infer that these features are rejected simply because the initial Figma screens display mostly plain text and basic summary blocks.

## Engineering adaptation

- Reuse visual tokens; do not copy Figma's fixed-position dimensions as universal responsive CSS.
- Match structure and hierarchy first, then responsive behavior.
- Keep app-level navigation outside ChatFrame.
- Treat exported reference code as a visual baseline, not production source.
- Define a fixture for each main/shared conversation state and compare screenshots at the Figma viewport size.
- Create separate desktop breakpoints from system design, not from the mobile mockup alone.

## Source and future revisions

The Figma link and node IDs serve as durable references, but the file may change. Record design revision/screenshots when implementation is undertaken. If tokens/layouts change, update this map before silently changing contracts.
