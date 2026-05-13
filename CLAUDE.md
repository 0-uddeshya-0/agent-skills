# CLAUDE.md

This is the agent-skills repository. Read this before doing anything.

## What this repo is
A library of slash-command skills for Claude Code, Cursor, and Codex.
Each skill lives in skills/<name>/skill.md and fixes one specific agent failure mode.

## Rules for contributing a skill
1. Every skill needs: skills/<name>/skill.md
2. Every skill needs an entry in .claude-plugin/plugin.json
3. Every skill needs a one-line entry in README.md under the correct category table
4. skill.md must have ALL sections: purpose, when_to_use, when_not_to_use, input, output, instructions, constraints, example, feedback_log
5. The example must use real values — no placeholders like <your_value_here>

## Rules for editing skills
- Do not change the instructions section without a documented failure case
- Do not remove the feedback_log section
- Do not add sections not in the template

## What gets rejected
- Skills with vague names (improve-quality, be-smart)
- Skills missing when_not_to_use
- Skills whose instructions require a human to run them manually
- Skills that duplicate an existing skill with different wording

## File ownership
- skills/*/skill.md — skill definitions (one per skill)
- .claude-plugin/plugin.json — skill registry for Claude Code
- .agent-context-template/ — template files copied by /setup into user repos
- examples/ — full working agent patterns using composed skills
- CONTRIBUTING.md — contributor rules
