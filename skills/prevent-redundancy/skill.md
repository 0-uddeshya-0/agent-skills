# skill: prevent-redundancy

## purpose
Before sending a response, remove content that was already said earlier in the conversation. Eliminates the agent habit of re-explaining things the user already knows, restating conclusions, and padding with information already on screen.

## when_to_use
- Multi-turn conversations where context has accumulated
- Agent loop where the same output type is generated repeatedly
- Before summarizing or concluding — summaries often just repeat the body

## when_not_to_use
- First turn with no prior context
- User explicitly asks you to repeat or restate something
- Generating formal documents where repetition serves structure (exec summary, conclusion)

## input
```
draft: string            # What you're about to say
prior_context: string    # What has already been said this session
```

## output
```json
{
  "redundant_parts": [
    { "text": "string", "already_said_in": "string" }
  ],
  "clean_draft": "string",
  "reduction_pct": "int"
}
```

## instructions

```
For each claim or statement in the draft: has this already been established in prior_context?

Redundant if: restates a fact already established, re-explains something the user confirmed understanding of, summarizes points just made in the same response.

NOT redundant if: adds a new angle or detail, briefly mentioned before but now developed meaningfully, direct answer to a new question even if related to prior topics.

Remove redundant parts. Do not replace them — just cut.
clean_draft must still be a complete, coherent response.
If nothing is redundant, redundant_parts is empty and clean_draft equals the input draft.

Return JSON only.
```

## constraints
- Max output tokens: 400
- clean_draft must be coherent after cuts
- Only flag genuine repetition — not related content or elaboration

## example

**Input:**
```
draft: "To summarize: the issue is a memory leak from unclosed DB connections. As I mentioned, the connection pool isn't released after each request. The fix is adding connection closing in the finally block, which I described. This resolves the memory leak."
prior_context: "Agent turn 3: Memory leak caused by unclosed DB connections — pool not released after requests. Fix: connection.close() in a finally block."
```

**Output:**
```json
{
  "redundant_parts": [
    { "text": "the issue is a memory leak from unclosed DB connections", "already_said_in": "Turn 3 established root cause" },
    { "text": "the connection pool isn't released after each request", "already_said_in": "Turn 3 explained mechanism" },
    { "text": "adding connection closing in the finally block, which I described", "already_said_in": "Turn 3 gave the fix" },
    { "text": "This resolves the memory leak", "already_said_in": "Restates conclusion already made" }
  ],
  "clean_draft": "Fix is in — connection.close() in the finally block. Ready to test.",
  "reduction_pct": 76
}
```

## feedback_log
<!-- date | situation | what went wrong | better behavior -->
