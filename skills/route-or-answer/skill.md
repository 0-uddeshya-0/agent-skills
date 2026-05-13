# skill: route-or-answer

## purpose
Decide whether a task needs a tool call or can be answered directly. Stops agents from calling APIs for questions they already know the answer to.

## when_to_use
- Before any tool call in an agent loop
- When the agent has 2+ tools available and could reach for any of them
- When tool calls have latency, cost, or rate-limit implications

## when_not_to_use
- Task explicitly requires live or user-specific data — just call the tool
- Task is ambiguous — run /clarify-intent first, then this
- You've already called the tool and have the data

## input
```
task: string           # Current objective or user request
available_tools: list  # Tool names + one-line descriptions
context: string        # What's already known in this session
```

## output
```json
{
  "decision": "answer_directly" | "use_tool",
  "tool_name": "string | null",
  "reason": "string",
  "answer": "string | null"
}
```

## instructions

```
Decide: can you answer this from general knowledge or context already provided, or does it need a tool?

Answer directly if:
- The answer is stable knowledge (geography, math, definitions, well-known facts)
- The answer is already present in the provided context

Use a tool if:
- The answer changes over time (prices, weather, status, user records)
- The task requires an action (create, update, send, delete)
- The answer depends on data specific to this user or session not already in context

Pick one. Do not hedge. If genuinely unsure, default to use_tool — safer error.

Return JSON only. No prose before or after.
```

## constraints
- Max output tokens: 150
- Must be valid JSON
- If decision is answer_directly, answer field must be filled
- If decision is use_tool, answer must be null

## example

**Input:**
```
task: "What timezone is Tokyo in?"
available_tools: ["web_search", "calendar_api", "user_profile"]
context: ""
```

**Output:**
```json
{
  "decision": "answer_directly",
  "tool_name": null,
  "reason": "Tokyo timezone is stable factual knowledge.",
  "answer": "Tokyo is JST (UTC+9)."
}
```

---

**Input:**
```
task: "What is the user's current plan?"
available_tools: ["billing_api"]
context: "user_id: 8821"
```

**Output:**
```json
{
  "decision": "use_tool",
  "tool_name": "billing_api",
  "reason": "Subscription tier is user-specific, not in context.",
  "answer": null
}
```

## feedback_log
<!-- date | situation | what went wrong | better behavior -->
