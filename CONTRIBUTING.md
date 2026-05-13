# Contributing

## The one test
Before opening a PR, answer this out loud:
> "Would I type /<name> in a real coding session today?"

If no — rework it or don't open the PR.

## Adding a skill

1. Fork the repo
2. Create `skills/<your-skill-name>/skill.md` using the template below
3. Add entry to `.claude-plugin/plugin.json`
4. Add one-line entry to README.md under the right category
5. PR with: what failure this fixes + a real example

## Template

```markdown
# skill: <name>

## purpose
One sentence. Specific problem. No buzzwords.

## when_to_use
2–4 concrete situations.

## when_not_to_use
2–4 situations where this costs more than it saves.

## input
field_name: type  # description

## output
field_name: type  # description

## instructions
The exact prompt. Copy-paste ready. No placeholders.

## constraints
- Max output tokens: N
- Hard limits on behavior

## example
Real input with actual values.
Exact output the skill produces.

## feedback_log
<!-- date | situation | what went wrong | better behavior -->
```

## Improving an existing skill
PR must include:
- The failure case that motivated the change (from feedback_log)
- Exact lines changed
- Before/after example

"Improved wording" PRs without a documented failure are closed without review.

## What we reject without review
- Vague skill names
- Missing when_not_to_use
- Placeholder examples
- Duplicate of existing skill
- Skills that need a human to run them
