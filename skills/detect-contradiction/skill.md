# skill: detect-contradiction

## purpose
Compare a response draft against source data or prior established facts. Catches factual conflicts before they reach the user. The most common form of agent unreliability is saying something in turn 8 that conflicts with what was confirmed in turn 3.

## when_to_use
- Before returning any response that makes factual claims about user data
- Source data was provided (API response, document, database record) and agent will reference it
- Multi-turn conversation where facts have accumulated

## when_not_to_use
- First turn with no prior context
- Creative or subjective tasks — no ground truth to contradict
- Generating hypotheticals — internal consistency matters more than factual accuracy

## input
```
response_draft: string       # What the agent is about to say
established_facts: string    # Confirmed facts from context, source docs, or earlier turns
```

## output
```json
{
  "contradictions": [
    {
      "claim": "string",
      "conflicts_with": "string",
      "severity": "blocks|flags"
    }
  ],
  "clean": true | false,
  "corrected_draft": "string | null"
}
```

## instructions

```
Compare the draft against established facts.

A contradiction is when the draft states or implies something that conflicts with a confirmed fact.

NOT a contradiction:
- Elaborating on a fact with more detail
- Phrasing the same fact differently
- Not mentioning a fact (omission is not contradiction)

Severity:
- blocks: changes the correctness of the response (wrong number, wrong name, wrong status)
- flags: ambiguous phrasing, missing qualifier — response might still be usable

If contradictions exist: rewrite only the conflicting parts.
Do not rewrite the entire draft. Do not add new information.
corrected_draft is null if clean is true.

Return JSON only.
```

## constraints
- Max output tokens: 400
- Only flag genuine conflicts — not missing information
- corrected_draft: fix only what's wrong, nothing else

## example

**Input:**
```
response_draft: "Your Pro subscription renews on March 20th for $79/month."
established_facts: "billing_api: { plan: 'Pro', price: 49, renewal_date: '2025-03-15' }"
```

**Output:**
```json
{
  "contradictions": [
    {
      "claim": "renews on March 20th",
      "conflicts_with": "renewal_date is 2025-03-15",
      "severity": "blocks"
    },
    {
      "claim": "$79/month",
      "conflicts_with": "price is $49",
      "severity": "blocks"
    }
  ],
  "clean": false,
  "corrected_draft": "Your Pro subscription renews on March 15th for $49/month."
}
```

## feedback_log
<!-- date | situation | what went wrong | better behavior -->
