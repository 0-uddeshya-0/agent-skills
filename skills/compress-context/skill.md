# skill: compress-context

## purpose
Shrink a long conversation or document context 60–75% while keeping everything needed for the current task. Prevents token bloat across multi-turn sessions without losing decisions or facts.

## when_to_use
- Conversation exceeds ~3,000 tokens and will continue for more turns
- Passing context from one agent step to the next and most of it is irrelevant
- Long document needs repeated reference across multiple calls

## when_not_to_use
- Context is under 1,000 tokens — overhead not worth it
- Task requires verbatim quoting (legal, transcription)
- First turn — nothing to compress

## input
```
context: string        # Full context to compress
current_task: string   # What the agent needs to do next — determines what's relevant
```

## output
```json
{
  "compressed": "string",
  "original_tokens": "int (approx)",
  "compressed_tokens": "int (approx)",
  "dropped": ["category of removed content"]
}
```

## instructions

```
Compress context for a specific task. Keep only what is needed to complete that task.

Rules:
1. Convert verbose explanations to one-line facts
2. Remove pleasantries, filler, agent meta-commentary ("Sure, I'd be happy to...")
3. Merge repeated or restated information into one canonical statement
4. Remove content that does not affect the current task
5. Preserve: decisions made, data values, constraints, errors, user preferences stated
6. Use shorthand: "User: cancel sub" not "The user indicated they would like to cancel their subscription"

Never drop decisions, confirmed facts, active constraints, or error states.
Never hallucinate — if uncertain whether something is relevant, keep it.
Target 60–75% reduction. More than that risks losing signal.

Return JSON only.
```

## constraints
- Max output tokens: 600
- Never drop: decisions, confirmed facts, constraints, errors
- Never compress below 25% of original — over-compression loses signal

## example

**Input:**
```
context: "User: Hi, trouble with my account. Agent: Happy to help! What's the issue? User: API key stopped working. Been using it 6 months. Agent: Can you share first 6 chars? User: sk-abc1. Agent: Key was revoked Jan 14 due to billing issue — card 4242 declined. User: Card expired. Updated it, new card ends 9191. Agent: Payment should process in 2–15 min, key reactivated then. User: How long exactly? Agent: Usually 2–5 min, up to 15."
current_task: "Check if the API key has been reactivated"
```

**Output:**
```json
{
  "compressed": "API key sk-abc1*** revoked Jan 14 — failed payment (card 4242 expired). User updated to card 9191. Reactivation pending, expected 2–15 min. Task: check reactivation status.",
  "original_tokens": 120,
  "compressed_tokens": 38,
  "dropped": ["pleasantries", "back-and-forth confirmation dialogue", "agent offers"]
}
```

## feedback_log
<!-- date | situation | what went wrong | better behavior -->
