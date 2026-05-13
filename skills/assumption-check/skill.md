# skill: assumption-check

## purpose
List the assumptions baked into a plan before executing it. Surfaces hidden guesses that could make the whole thing wrong. Blocks high-risk ones before they cascade.

## when_to_use
- Before executing a multi-step plan where one wrong assumption cascades
- Acting on behalf of a user without full context (delegated tasks, automated jobs)
- Task involves a domain you haven't verified facts about

## when_not_to_use
- Task fully specified with verified facts — no inference needed
- Single-step tasks with no dependencies
- You've already confirmed assumptions with the user this session

## input
```
plan: string      # The plan or action you're about to execute
context: string   # What was explicitly stated vs what you inferred
```

## output
```json
{
  "assumptions": [
    {
      "statement": "string",
      "risk": "low" | "medium" | "high",
      "source": "inferred" | "stated",
      "verify_by": "string | null"
    }
  ],
  "safe_to_proceed": true | false,
  "blocker": "string | null"
}
```

## instructions

```
Read the plan. Find every place you are acting on something assumed rather than stated.

State each assumption as a plain fact sentence: "The user wants X", "The file format is Y."

Risk levels:
- high: if wrong, output is useless or harmful
- medium: if wrong, requires significant rework  
- low: if wrong, minor correction needed

For high-risk assumptions: state exactly how to verify before proceeding.

safe_to_proceed is false if any high-risk unverified assumption exists.
blocker is the single most dangerous unverified assumption.

Return JSON only. Max 6 assumptions — if you have more, you need clarification first.
```

## constraints
- Max output tokens: 300
- Max 6 assumptions — more means the task itself needs clarification
- Only material assumptions — not obvious facts

## example

**Input:**
```
plan: "Refactor auth module to use JWT, update middleware, deploy to production."
context: "User said: 'We should modernize our auth.' No other details."
```

**Output:**
```json
{
  "assumptions": [
    {
      "statement": "JWT is the desired replacement (not OAuth, API keys, or another approach)",
      "risk": "high",
      "source": "inferred",
      "verify_by": "Ask: which auth method do you want to migrate to?"
    },
    {
      "statement": "Production deployment is in scope for this task",
      "risk": "high",
      "source": "inferred",
      "verify_by": "Confirm: refactor only, or also deploy?"
    },
    {
      "statement": "Session-based auth is the only current auth system",
      "risk": "medium",
      "source": "inferred",
      "verify_by": "Check codebase for other auth paths before modifying middleware"
    }
  ],
  "safe_to_proceed": false,
  "blocker": "JWT is the desired replacement — wrong choice invalidates the entire plan"
}
```

## feedback_log
<!-- date | situation | what went wrong | better behavior -->
