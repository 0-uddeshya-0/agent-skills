# skill: simplify-scope

## purpose
Cut a bloated task to the minimum version that still satisfies the user's actual goal. Prevents agents from doing 8 things when 2 were needed.

## when_to_use
- Task has accumulated scope through a long conversation ("also do X, and while you're at it...")
- Request contains nested sub-tasks that weren't explicitly asked for
- Agent output will clearly be excessive for the actual request

## when_not_to_use
- Task is already atomic and specific
- User explicitly asked for exhaustive or comprehensive output
- Cutting scope would change the core answer

## input
```
task: string          # Full task as currently understood
user_goal: string     # What success looks like — the actual objective
```

## output
```json
{
  "simplified_task": "string",
  "removed": ["item — reason"],
  "kept": ["item — reason"],
  "reduction": "none" | "minor" | "significant"
}
```

## instructions

```
Cut scope, not quality. Find the minimum task that makes the user satisfied.

For each part of the task ask:
- Did the user explicitly request this, or was it assumed?
- Can the user do this in 30 seconds after seeing the core output?
- Does this require a separate action that wasn't requested?

If any answer is yes: remove it.
Never remove anything the user explicitly stated.
One-line justification for each removal and each kept item.

Return JSON only.
```

## constraints
- Max output tokens: 250
- simplified_task must be completable in a single response
- Do not remove explicitly stated requirements

## example

**Input:**
```
task: "Write a blog post about Python async with intro to concurrency, history of threading, GIL explanation, Node.js comparison, asyncio examples, benchmarks, pitfalls, migration guide, and FAQ."
user_goal: "Help our team understand when to use async."
```

**Output:**
```json
{
  "simplified_task": "Write a practical post explaining Python async/await: what it is, when to use it vs threads, and 2 code examples. For engineers already comfortable with Python.",
  "removed": [
    "Threading history — background, not needed to understand async",
    "GIL deep-dive — tangential to the goal",
    "Node.js comparison — adds scope without serving stated goal",
    "Benchmarks — requires running code, separate deliverable",
    "Migration guide — separate article",
    "FAQ — add based on reader questions later"
  ],
  "kept": [
    "Async/await explanation — core to the goal",
    "When to use vs threads — directly answers the team need",
    "Code examples — essential for practical understanding"
  ],
  "reduction": "significant"
}
```

## feedback_log
<!-- date | situation | what went wrong | better behavior -->
