# Agent Prompt: Publish agent-skills to GitHub

You are an expert open-source maintainer. Your job is to take the `agent-skills/` directory provided to you and publish it as a production-ready GitHub repository. Follow every step exactly. Do not skip steps. Do not hallucinate commands.

---

## Context: what this repo is

`agent-skills` is a library of 18 slash-command skills for Claude Code, Cursor, and Codex. Each skill is a markdown file (`skills/<name>/skill.md`) that fixes one specific agent failure mode. The skills are plug-and-play — developers copy instructions into system prompts or invoke them as slash commands.

Inspired by: github.com/obra/superpowers, github.com/mattpocock/skills

---

## Step 1: Verify the repo structure

Before doing anything, confirm this exact structure exists:

```
agent-skills/
├── README.md
├── CLAUDE.md
├── CONTRIBUTING.md
├── LICENSE
├── .claude-plugin/
│   └── plugin.json
├── .agent-context-template/
│   ├── context.md
│   └── graph.md
├── examples/
│   ├── support-agent.md
│   ├── coding-agent.md
│   └── research-agent.md
└── skills/
    ├── setup/skill.md
    ├── route-or-answer/skill.md
    ├── clarify-intent/skill.md
    ├── simplify-scope/skill.md
    ├── assumption-check/skill.md
    ├── compress-context/skill.md
    ├── cache-hint/skill.md
    ├── caveman-summary/skill.md
    ├── graphify/skill.md
    ├── detect-contradiction/skill.md
    ├── validate-output/skill.md
    ├── prevent-redundancy/skill.md
    ├── format-response/skill.md
    ├── tool-call-guard/skill.md
    ├── error-to-action/skill.md
    ├── tone-match/skill.md
    ├── explain-depth/skill.md
    └── humanize-output/skill.md
```

Run: `find agent-skills -type f | sort`
If any file is missing, stop and report which files are missing before continuing.

---

## Step 2: Initialize git

```bash
cd agent-skills
git init
git add .
git commit -m "Initial commit: 18 agent skills for Claude Code, Cursor, and Codex"
```

---

## Step 3: Create the GitHub repository

Using the GitHub CLI (gh):

```bash
gh repo create agent-skills \
  --public \
  --description "18 plug-and-play skills that fix specific AI agent failure modes. Works with Claude Code, Cursor, and Codex." \
  --homepage "" \
  --push \
  --source .
```

If `gh` is not available, use the GitHub API:
```bash
curl -X POST https://api.github.com/user/repos \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "agent-skills",
    "description": "18 plug-and-play skills that fix specific AI agent failure modes. Works with Claude Code, Cursor, and Codex.",
    "private": false,
    "auto_init": false
  }'
```

Then push:
```bash
git remote add origin https://github.com/YOUR_USERNAME/agent-skills.git
git push -u origin main
```

Replace YOUR_USERNAME with the actual GitHub username.

---

## Step 4: Update plugin.json with real repo URL

After the repo is created, open `.claude-plugin/plugin.json` and update this field:

```json
"repository": "https://github.com/YOUR_USERNAME/agent-skills"
```

Commit the update:
```bash
git add .claude-plugin/plugin.json
git commit -m "chore: add repository URL to plugin.json"
git push
```

---

## Step 5: Update README.md install command

In `README.md`, find this line:
```
claude mcp add https://github.com/YOUR_USERNAME/agent-skills
```

Replace `YOUR_USERNAME` with the actual GitHub username. Commit and push.

---

## Step 6: Add GitHub repository topics

```bash
gh repo edit agent-skills \
  --add-topic "ai-agents" \
  --add-topic "prompt-engineering" \
  --add-topic "claude" \
  --add-topic "cursor" \
  --add-topic "llm" \
  --add-topic "agent-skills" \
  --add-topic "developer-tools"
```

---

## Step 7: Verify the published repo

After pushing, confirm:

1. `https://github.com/YOUR_USERNAME/agent-skills` loads
2. README renders correctly (tables, code blocks, links)
3. All 18 skill files are visible under `skills/`
4. `.claude-plugin/plugin.json` is present
5. `CLAUDE.md` is present at root (Claude Code reads this automatically)

Run a final check:
```bash
gh repo view YOUR_USERNAME/agent-skills
```

---

## Step 8: What NOT to do

- Do not rename any files or directories — the paths in plugin.json must match
- Do not add a `.gitignore` that hides `.claude-plugin/` or `.agent-context-template/` — these must be public
- Do not change the skill.md structure — it must maintain all sections
- Do not push a private repo — this library only works if skills are publicly accessible
- Do not modify the feedback_log sections — they are intentionally empty for users to fill

---

## Step 9: Optional — add a GitHub Action for skill validation

Create `.github/workflows/validate-skills.yml`:

```yaml
name: Validate Skills
on: [pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Check all skills have required sections
        run: |
          failed=0
          for skill_file in skills/*/skill.md; do
            for section in "## purpose" "## when_to_use" "## when_not_to_use" "## input" "## output" "## instructions" "## constraints" "## example" "## feedback_log"; do
              if ! grep -q "$section" "$skill_file"; then
                echo "MISSING: $section in $skill_file"
                failed=1
              fi
            done
          done
          if [ $failed -eq 1 ]; then exit 1; fi
          echo "All skills valid."
      - name: Check plugin.json matches skill directories
        run: |
          for skill_dir in skills/*/; do
            skill_name=$(basename "$skill_dir")
            if ! grep -q "\"name\": \"$skill_name\"" .claude-plugin/plugin.json; then
              echo "MISSING from plugin.json: $skill_name"
              exit 1
            fi
          done
          echo "plugin.json is in sync."
```

Commit and push this file.

---

## Done

The repo is live. Share the URL:
`https://github.com/YOUR_USERNAME/agent-skills`

Developers install with:
```bash
git clone https://github.com/YOUR_USERNAME/agent-skills
```

And use skills immediately by copying instructions from any `skill.md` into their system prompt or CLAUDE.md.
