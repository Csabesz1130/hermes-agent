---
name: targetgraph
description: "Ground every reasoning step in Targetgraph's evidence graph using the rank_targets, explain_target, pathfind, query_graph, recent_discoveries, and score_with_jepa MCP tools. Cite Targetgraph node ids in every claim."
version: 0.1.0
author: Targetgraph × Hermes Integration
license: MIT
metadata:
  hermes:
    tags: [Biotech, Drug-Discovery, Evidence-Graph, MCP, Targetgraph]
    related_skills: [mcp]
---

# Targetgraph Discovery Skill

This skill is the contract between the hermes-agent runtime and the
Targetgraph Research Suite. When loaded, the agent SHOULD discover the
Targetgraph MCP server at `$TARGETGRAPH_MCP_URL` (defaults set by the
sidecar's compose service) and use only its tools to ground biomedical
claims.

## Hard rule — citation discipline

**Never** state a finding, recommendation, or hypothesis without citing the
Targetgraph node id(s) that support it. If you cannot cite a node, say so
explicitly and propose what to query next. Hallucinating gene names,
mechanisms, or trials when these tools are available is a critical failure.

## Tools available

- `rank_targets(disease, top_k=10)` — fused-evidence target ranking
- `explain_target(target, disease)` — evidence chain with provenance hashes
- `pathfind(source, sink, max_hops=4)` — shortest evidence path between nodes
- `query_graph(cypher, limit=25)` — read-only Cypher pass-through
- `recent_discoveries(limit=10)` — cross-agent unified discovery feed
- `score_with_jepa(entities, text)` — DreamJEPA self-supervised score in [0,1]

## Default reasoning recipe

1. `rank_targets(disease, top_k=5)`.
2. For the top 1–2 candidates: `explain_target(target, disease)`.
3. If a mechanism is plausible: `pathfind(target, phenotype_node)`.
4. Before emitting a candidate: `score_with_jepa(entities, description)`.
   - JEPA < 0.3 → deepen the investigation or drop the candidate.
   - JEPA 0.3–0.7 → mark as `medium` confidence.
   - JEPA > 0.7 → emit as `high`.

## Output envelope

When you have graph-cited findings, emit a single JSON object — the
Targetgraph backend parses this directly into Helios `Discovery` records:

```json
{
  "candidates": [
    {
      "type": "target",
      "title": "EGFR for cutaneous SCC",
      "description": "EGFR overexpression drives proliferation in cSCC; supported by Targetgraph evidence path …",
      "entities": ["EGFR", "cSCC"],
      "confidence_score": 0.78,
      "graph_citations": ["tg:gene:1956", "tg:disease:1024", "tg:edge:447821"],
      "tool_trace": ["rank_targets", "explain_target", "score_with_jepa"]
    }
  ]
}
```

`type` MUST be one of: `target`, `hypothesis`, `pathway`, `evidence`, `insight`.

## When the user asks a question on a chat surface

If the user reached you over Slack / Telegram / MCP and just wants a quick
answer (no autonomous loop), still cite node ids inline (e.g. `[tg:gene:1956]`)
and POST the final answer back to the Targetgraph backend at
`POST $TARGETGRAPH_API_URL/api/v1/hermes-agent/discoveries` so it lands in
the unified discovery registry alongside autonomous-run findings.

## Failure handling

If a tool returns `{ok: false, ...}`, retry once with a more restrictive
query. If it still fails, surface the error verbatim — never fabricate node ids.
