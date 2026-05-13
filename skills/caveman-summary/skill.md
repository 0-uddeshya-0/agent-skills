# skill: caveman-summary

## purpose
Rewrite any text into the shortest version that keeps all meaning. Forces the agent to stop writing when it's done. Named for the rule: explain it like a caveman explaining to another caveman — short words, direct sentences, nothing extra.

## when_to_use
- Output is longer than the user's need requires
- Summarizing a doc or data dump for a downstream step
- Creating a context snippet to pass to the next step
- User asked for "quick summary" or "key points"

## when_not_to_use
- User explicitly wants comprehensive or full-detail output
- Precision would be sacrificed (legal, medical, technical specs)
- The long version is the deliverable (report, blog post, proposal)

## input
```
text: string            # Content to compress
max_sentences: int      # Hard cap (default: 5)
preserve: list[string]  # Values/facts that must survive verbatim
```

## output
```json
{
  "summary": "string",
  "sentence_count": "int",
  "dropped_detail": ["string"]
}
```

## instructions

```
Rewrite in fewest words possible. Keep all facts. Remove everything else.

Rules:
- Short words over long: "use" not "utilize", "show" not "demonstrate"
- One idea per sentence
- No "It is important to note that..." — just say it
- No hedging unless it is the core meaning
- Numbers and specific values survive verbatim
- Items in preserve must appear in output exactly as given
- Stop when you have said everything that matters

Return JSON only. Output must not exceed max_sentences.
```

## constraints
- Max output tokens: 200
- Output must not exceed max_sentences
- Values in preserve are non-negotiable

## example

**Input:**
```
text: "After a thorough investigation our engineering team identified the root cause of the performance degradation users have been experiencing. A database migration deployed last Tuesday introduced an N+1 query pattern in the user feed endpoint causing elevated database load during peak hours. We have a fix and plan to deploy Saturday during the maintenance window."
max_sentences: 3
preserve: ["N+1 query", "Saturday"]
```

**Output:**
```json
{
  "summary": "Tuesday's DB migration introduced an N+1 query bug in the user feed. It's spiking DB load at peak hours. Fix deploys Saturday.",
  "sentence_count": 3,
  "dropped_detail": ["'thorough investigation' framing", "'maintenance window' label", "attribution to engineering team"]
}
```

## feedback_log
<!-- date | situation | what went wrong | better behavior -->
