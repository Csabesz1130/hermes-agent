# Integrations

## quantaj (selective vendor)

[csabesz1130/quantaj](https://github.com/csabesz1130/quantaj) selectively
vendors hermes-agent v0.14.0 into its `lib/hermes/` subtree to power its
trading agent and self-improving training loop.

**What quantaj uses from this repo:**

| Upstream | Vendored as | Purpose |
| --- | --- | --- |
| `run_agent.py` (slim) | `lib/hermes/agent.py` | Tool-calling loop |
| `hermes_state.py` (slim) | `lib/hermes/state.py` | SessionDB + FTS5 |
| `tools/lazy_deps.py` | `lib/hermes/tools/lazy_deps.py` | Pinned allowlist |
| `providers/{base,anthropic,openai,openrouter}` | `lib/hermes/providers/` | Multi-LLM |
| `batch_runner.py` (slim) | `lib/hermes/trajectory/batch_runner.py` | Trajectory extraction |
| `trajectory_compressor.py` (slim) | `lib/hermes/trajectory/compressor.py` | Compression |
| `gateway/{__init__, telegram, discord, slack}` | `lib/hermes/gateway/` | Messaging |
| `tools/environments/modal*.py` | `lib/hermes/modal_env/` | Modal sandbox backend |

**What was dropped (intentionally):**

- `cli.py` (660KB monolith) - quantaj has its own thin CLI in `agent/runtime.py`
- Plugin marketplace, dashboards, ACP adapter
- Voice / STT / image / FAL tools
- 4 of 7 terminal backends (local/Docker/SSH/Singularity/Daytona/Vercel) — only Modal kept
- Mistral provider (upstream quarantined post Mini Shai-Hulud worm, May 2026)
- Cron scheduler (Modal cron used instead)
- Matrix / Feishu / DingTalk gateway adapters

**Why selective vendor vs. PyPI install?**

quantaj needs to adapt the agent to its market-making domain — registering
forecast/quote/Kelly tools, custom skills, a hot-swappable LoRA inference
path. The PyPI install path would also pull the 660KB CLI and unused
backends. Selective vendor keeps the dependency footprint tight and lets
quantaj evolve the agent independently when needed.

**Sync policy:** quantaj cherry-picks security and provider updates from
this repo manually; it does not auto-track upstream.
