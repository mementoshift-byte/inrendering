# 08 · Performance, caching and token economics

**Status:** confirmed priorities; metrics and concrete cache technologies are proposed.

## Optimize perceived response time, not merely token count

The system exists to support many domains, workflows and future services without making every ordinary UI interaction wait for an LLM. Performance is a joint problem:

- Agent reasoning/generation latency
- Tokens sent/returned
- Network and backend round trips
- Manifest/data retrieval
- Runtime compilation (if any)
- React rendering and layout
- Storage hydration and navigation
- Perceived touch/typing feedback

Smaller components do not automatically improve the system. A 40-node component tree can cost more tokens and parsing than a single semantic component backed by 40 compiled React components.

## Cache by *what changes*, not where it appears

| Cache | Example key | Invalidation | Notes |
|---|---|---|---|
| Installed component code | package version + component version | App release/update | Web may code-split; native bundles prepackage |
| Compiled convention manifests | source hash + schema version | File/config revision | No per-render Obsidian parsing |
| Resolved FrameContext | global/domain/agent/chat versions + device capability profile + permission revision | Relevant context/permission change | Not a globally shared public cache |
| Semantic mapping/template | intent + target + context resolution version + template version | Mapping/version change | Do not cache private data with template |
| Normalized UI structure | canonical tree hash + engine/schema version | Engine or tree version change | Structure reusable independently of instance values |
| Parsed message content | block ID + revision + parser version | Block revision | Cache only when beneficial |
| Resource data | resource ID + revision/ETag + access scope | New revision, expiration, permission change | Sensitive/private scoped cache |
| Composer/interaction state | conversation ID + instance ID | User action, completion, explicit reset | Persist rather than deduplicate |
| Reference anchors | reference ID + source revision | Source mutation/reanchor | Never reuse stale location silently |
| Scroll/viewport | conversation ID + anchor ID | Navigation/viewport changes | Not portable as raw pixels |

**Structural cache** is not **rendered-instance cache**. Reusing a SingleChoice template must not share the chosen value across independent instances.

## Hashing conventions

Canonicalize a validated tree by schema-driven prop ordering, normalized defaults and resolved node/template versions. Hash the **structural template** without identity-specific values when possible. Maintain separate:
- Template identity
- UI instance identity
- Data resource identity/revision
- Active user interaction state
- Context/policy version

A content hash is useful for template dedupe, not a replacement for stable message/block/reference IDs.

## Agent-facing economy

Prefer messages such as:

~~~json
{
  "kind": "semantic",
  "intent": "review",
  "target": "pending_orders"
}
~~~

over fully specifying the order table and action controls when the frame already knows the domain, registered template and authorized data reference.

Further savings:
- Pass a compact **capability projection** rather than the entire registry/manifests.
- Reuse the projection ID/hash when unchanged (subject to model/runtime support).
- Send resource IDs and bounded summaries instead of whole datasets where the device can resolve them.
- Request extra component schemas only when the agent needs details.
- Prefer local expand/filter/select interactions and send only the consequential final result.
- Do not repeatedly serialize Figma-style pixel details in model outputs.

These are architecture goals, not guaranteed token savings; measure prompts and cache-hit rates.

## Interaction latency

Every value change should update locally. Event dispatch to an agent is appropriate only when new reasoning or remote data is required. React state should live as close as possible to the interactive instance; avoid a global rerender of the entire message list for one slider movement.

The browser/client may cache and reuse parsed text and component descriptors, but avoid unnecessary memoization that introduces stale dependencies or complex invalidation.

## Streaming and incremental rendering

- Use stable React keys for message, block and UI-node instances.
- Apply delta updates to the canonical store and reconcile minimal subtrees.
- Coalesce very high-frequency token updates only if it does not increase perceived text lag.
- Avoid continuously rebuilding the entire markdown AST for the full conversation.
- Measure scroll anchoring under dynamic component heights.
- Virtualize only after confirming it does not break accurate selections or restoration.

## Storage tiers

**RAM/UI stores:** ephemeral interaction state and hot current conversation data.

**Device-persisted cache:** drafts, reference tray, last resolved conventions, recent message content, allowed resource caches; user-data controls and expiry apply.

**Remote authoritative storage:** canonical conversations/configuration versions and synchronized durable artifacts, according to the broader app architecture.

**Central physical file storage:** domain files, recordings, documents and vault content; accessed through services, not directly scraped by every UI renderer.

Specific browser storage technologies (IndexedDB, memory stores, service workers, etc.) are implementation choices, not settled requirements.

## Security-sensitive caching

- Partition by user/session/agent authorization scope.
- Cache permission revisions and revalidate consequential actions server-side.
- Never mix personal/financial data into a shared template cache.
- Avoid persistent plaintext secrets in client caches.
- Provide explicit cache reset/logout behavior.
- Do not assume a local cache is authoritative after access revocation.
- If a component uses remote data, expiration and offline behavior must be visible.

## Suggested instrumentation

Collect in development:
- First frame paint and time to first visible message
- Time from stream patch to visible text
- Time from input gesture to local UI update
- Frame/renderer commit count during streaming
- Scroll-anchor drift
- Semantic-resolution cache hit/miss
- Component template cache hit/miss
- Agent tokens spent on UI specification (when observable)
- Number of agent calls caused by ordinary UI actions (desired: zero)
- Selection anchor verification failure/staleness rate

Target numeric budgets should be established by profiling actual phone and desktop devices rather than asserted prematurely.

## Performance validation scenarios

1. Long conversation with many components while streaming near bottom.
2. Long conversation while reading far above newest message.
3. Rapid back-and-forth switching among MAIN and several domain chats.
4. Local multi-reference selection while a different message streams.
5. Reusing the same template in many chats with independent state.
6. Cold startup with cache miss, warm startup with cached manifest, offline startup with restricted permissions.
