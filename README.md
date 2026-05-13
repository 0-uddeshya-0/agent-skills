# agent-skills

**18 slash-command skills for Claude Code, Cursor, and Codex. Each one fixes a specific failure mode.**

Not a prompt library. Not a framework.  
Skills you invoke mid-session — `/graphify`, `/clarify-intent`, `/tool-call-guard` — that do real work inside your agent tool of choice.

---

## The failure modes these fix

- Agent reads 8 files, edits the wrong one → **`/graphify`**
- Agent makes 4 API calls for a question it could answer directly → **`/route-or-answer`**
- Agent ships code based on an assumption it never checked → **`/assumption-check`**
- Agent sends an email to the wrong address because you said "send it" → **`/tool-call-guard`**
- Agent explains async/await to a senior engineer like they're 12 → **`/explain-depth`**
- Agent returns "Your subscription renews March 20th" when it's actually March 15th → **`/detect-contradiction`**
- Agent writes 400 words when 40 were needed → **`/caveman-summary`**

---

## Install in 60 seconds

### Claude Code
```bash
# Add to your project's CLAUDE.md or run once:
claude mcp add https://github.com/0-uddeshya-0/agent-skills
```

### Cursor / Codex / any agent
Copy the contents of any `skill.md` into your system prompt or `AGENTS.md`. Done.

### Manual (always works)
```bash
git clone https://github.com/0-uddeshya-0/agent-skills
# Pick a skill. Open skills/<name>/skill.md. Copy the instructions section.
# Paste into your system prompt.
```

---

## Skills

### 🧠 Cognitive — think before acting
| Skill | What it fixes |
|---|---|
| [`/setup`](skills/setup/skill.md) | Run once per repo. Creates `.agent-context/` so every other skill has shared project knowledge. |
| [`/route-or-answer`](skills/route-or-answer/skill.md) | Stops tool calls for questions the model already knows how to answer. |
| [`/clarify-intent`](skills/clarify-intent/skill.md) | One targeted question when intent is ambiguous. Nothing when it's clear. |
| [`/simplify-scope`](skills/simplify-scope/skill.md) | Cuts task to the minimum that satisfies the goal. |
| [`/assumption-check`](skills/assumption-check/skill.md) | Lists hidden assumptions before executing a plan. Blocks dangerous ones. |

### ⚡ Efficiency — fewer tokens, less waste
| Skill | What it fixes |
|---|---|
| [`/compress-context`](skills/compress-context/skill.md) | Shrinks accumulated conversation context 60–75% without losing decisions. |
| [`/cache-hint`](skills/cache-hint/skill.md) | Tags which parts of output are reusable across calls and which must be recomputed. |
| [`/caveman-summary`](skills/caveman-summary/skill.md) | Shortest possible version of any text. Forces the agent to stop writing when it's done. |
| [`/graphify`](skills/graphify/skill.md) | Reads your codebase, maps dependencies, writes `.agent-context/graph.md`. Agent navigates like someone who's worked in the repo for a month. |

### 🛡️ Reliability — don't ship wrong things
| Skill | What it fixes |
|---|---|
| [`/detect-contradiction`](skills/detect-contradiction/skill.md) | Compares draft response against source data. Catches factual conflicts before they go out. |
| [`/validate-output`](skills/validate-output/skill.md) | Confirms output actually answers what was asked, in the format that was requested. |
| [`/prevent-redundancy`](skills/prevent-redundancy/skill.md) | Removes content the agent already said earlier in the conversation. |

### 🔌 Integration — work with tools correctly
| Skill | What it fixes |
|---|---|
| [`/format-response`](skills/format-response/skill.md) | Applies correct formatting for Slack / terminal / email / JSON API — doesn't guess. |
| [`/tool-call-guard`](skills/tool-call-guard/skill.md) | Validates tool call params before execution. Blocks bad calls, warns on destructive ones. |
| [`/error-to-action`](skills/error-to-action/skill.md) | Converts raw error strings into plain-English explanations and specific next actions. |

### 💬 UX — responses that feel human
| Skill | What it fixes |
|---|---|
| [`/tone-match`](skills/tone-match/skill.md) | Reads user's emotional register. Adjusts before replying. |
| [`/explain-depth`](skills/explain-depth/skill.md) | Calibrates explanation depth to actual expertise level. No more over-explaining to seniors. |
| [`/humanize-output`](skills/humanize-output/skill.md) | Strips "Great question!", "Certainly!", and all other hollow AI-isms. |

---

## Start here

```
/setup
```

Run this once in any new project. It creates `.agent-context/` — a shared knowledge folder that every other skill reads from. Without it, skills work fine but in isolation. With it, they share project context.

---

## Composing skills

Skills chain. Common patterns:

```
# Before any tool use
/clarify-intent → /route-or-answer → /tool-call-guard

# Before any code change  
/graphify → /assumption-check → /validate-output

# Before any user-facing response
/detect-contradiction → /tone-match → /humanize-output

# For long conversations
/compress-context → /prevent-redundancy → /caveman-summary
```

Full working examples in [`examples/`](examples/).

---

## Self-improvement

Each skill has a `## feedback_log` section.  
When a skill misfires, log it there in one line:
```
# 2025-01-15 | clarify-intent | asked for clarification when intent was obvious
```
After 5–10 entries, paste the log to a model and ask it to improve the instructions. No automation needed — just a habit of noticing failures.

---

## Contributing

See [`CONTRIBUTING.md`](CONTRIBUTING.md). New skills must pass one test: would you type `/<name>` in a real session today?

---

## License
MIT
