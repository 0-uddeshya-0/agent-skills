# skill: clarify-intent

## purpose
Ask one targeted question when intent is genuinely ambiguous. Nothing when it's already clear. Stops agents from acting on wrong assumptions and from interrogating users with lists of questions.

## when_to_use
- User message is short and could mean multiple different things
- Correct next action depends on an unstated detail
- You're about to take an irreversible action (send, delete, publish, charge)

## when_not_to_use
- Intent is specific enough to act on without guessing
- You've already asked a clarifying question this session — infer instead
- Task is mechanical: translate this, format that, summarize this
- The question you'd ask is already answered elsewhere in context

## input
```
user_input: string    # What the user said
context: string       # Conversation history or session state
```

## output
```json
{
  "needs_clarification": true | false,
  "clarifying_question": "string | null",
  "clarified_task": "string | null",
  "confidence": "high" | "low"
}
```

## instructions

```
Read the user input. Is the intent specific enough to act on, or could two different users saying this want two completely different things?

If you need to clarify:
- Ask exactly ONE question
- Prefer choice questions: "Are you looking to X, Y, or Z?"
- Ask only what you need for the immediate next step

If intent is clear:
- State the task as a concrete action sentence starting with a verb
- Do not repeat what the user said — state what you will do

Return JSON only. clarifying_question and clarified_task are mutually exclusive.
```

## constraints
- Max output tokens: 120
- Maximum one clarifying question — never a list
- If asked twice in same session: set needs_clarification false, make best inference

## example

**Input:**
```
user_input: "Can you help with my account?"
context: "User authenticated. Account has billing, profile, API key sections."
```

**Output:**
```json
{
  "needs_clarification": true,
  "clarifying_question": "Are you looking to update billing, change your profile, or manage API keys?",
  "clarified_task": null,
  "confidence": "high"
}
```

**Input:**
```
user_input: "Delete the draft I was working on"
context: "User has one draft: id=draft_442, title='Q3 Report', last edited 3 min ago."
```

**Output:**
```json
{
  "needs_clarification": false,
  "clarifying_question": null,
  "clarified_task": "Delete draft draft_442 ('Q3 Report').",
  "confidence": "high"
}
```

## feedback_log
<!-- date | situation | what went wrong | better behavior -->
