# Guidance for coding agents working on inrendering

This repo is the **chat frame and rendering foundation**, not the whole Command Center. Read README.md, docs/00-project-charter.md, docs/01-architecture.md and docs/11-decisions-and-questions.md before implementing substantial changes.

## Ground rules

- Do not silently turn a documented **proposal** or **open question** into a fixed product requirement.
- Prioritize the actual conversation surfaces in Figma (Main Chat node 5:541 and Shared Conversation node 5:260).
- Keep navigation, webshop, repo dashboards, Obsidian editing, central file service and agent orchestration outside the frame package.
- Preserve framework independence of canonical message/selection/UI contracts where practical.
- Never execute arbitrary model-produced JSX in the privileged app runtime.
- Keep domain conventions declarative and versioned; never read the vault on every render.
- Provide stable content IDs and exact references; do not use DOM coordinates as persistent anchors.
- Preserve drafts, scroll and selections when moving between conversations.
- Make local UI interaction local; only meaningful events cross the agent boundary.
- Cache per data type and version; never treat all caches as one blob.
- Enforce permissions in application/service handlers, not only in UI visibility.
- Do not import code from other repos without first evaluating ownership, licensing, version compatibility and adapter boundaries.
- Test a thin end-to-end slice before growing the component vocabulary.

## Expected engineering output

For a change affecting architecture, include: contracts changed, rendering/event flow, state and cache implications, permission implications, migration risks and an acceptance test. Prefer an isolated, testable increment over a broad speculative refactor.

## Design references

- [Figma Command Center](https://www.figma.com/design/v17vwmp74ZbPpQwnVLelfB/Untitled?node-id=0-1)
- [Design map](docs/13-figma-design-map.md)
- [Roadmap](docs/10-implementation-roadmap.md)
