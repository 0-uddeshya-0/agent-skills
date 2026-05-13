# skill: explain-depth

## purpose
Calibrate how deeply to explain something based on who's asking. Stops agents from over-explaining to experts (condescending, wasteful) and under-explaining to beginners (useless). Same question from a senior engineer and a junior needs completely different answers.

## when_to_use
- User asked a technical or domain-specific question
- Expertise level materially affects how to answer
- Agent has been giving one-size-fits-all explanations

## when_not_to_use
- User's expertise level is already clear from prior turns
- Question has one reasonable explanation depth regardless of audience
- Task is procedural (do X, do Y) — depth calibration doesn't apply

## input
```
question: string             # What the user asked
user_signals: string         # Vocabulary, context, background clues from message
default_depth: string        # "beginner"|"intermediate"|"expert" fallback if signals weak
```

## output
```json
{
  "inferred_level": "beginner|intermediate|expert",
  "confidence": "high|low",
  "signals_used": ["string"],
  "response_strategy": {
    "lead_with": "string",
    "jargon_allowed": true | false,
    "analogies_needed": true | false,
    "show_code": true | false,
    "depth": "concept_only|concept_plus_example|full_detail"
  },
  "calibrated_answer": "string"
}
```

## instructions

```
Infer expertise from signals in the user message:

Expert signals: uses domain jargon correctly, asks about edge cases or internals, references specific tools/versions, asks "why" not "what"
Intermediate: understands basics, asks about practical application, one term slightly off
Beginner: asks for definitions, uses non-technical descriptions of technical things, asks "how do I", shows confusion about fundamentals

If signals weak or contradictory: use default_depth.

By level:
beginner: explain concept before solution, use analogy, no jargon without definition, one step at a time
intermediate: skip basics, one short example, jargon fine, mention trade-offs briefly
expert: lead with direct answer, include edge cases and failure modes, jargon and shorthand expected

calibrated_answer is the actual answer — not a plan for how to answer.

Return JSON only.
```

## constraints
- Max output tokens: 400
- calibrated_answer must be the real answer, not a meta-description
- Do not pad with caveats about the explanation style chosen

## example

**Input:**
```
question: "Why does my async function not wait for the API call?"
user_signals: "I'm new to JavaScript. I thought putting async makes it wait."
default_depth: "intermediate"
```

**Output:**
```json
{
  "inferred_level": "beginner",
  "confidence": "high",
  "signals_used": ["'I'm new to JavaScript'", "'I thought async makes it wait' — shows misunderstanding of async alone vs await"],
  "response_strategy": {
    "lead_with": "correct the misconception first",
    "jargon_allowed": false,
    "analogies_needed": true,
    "show_code": true,
    "depth": "concept_plus_example"
  },
  "calibrated_answer": "async alone doesn't make code wait — it just means the function is allowed to wait. You also need await in front of the API call.\n\nThink of it this way: async is a sign on a door that says 'waiting is allowed in here.' await is actually sitting down and waiting.\n\nChange this:\n```js\nasync function getData() {\n  fetch('/api/data'); // no await — doesn't wait\n}\n```\nTo this:\n```js\nasync function getData() {\n  const result = await fetch('/api/data'); // waits\n}\n```"
}
```

## feedback_log
<!-- date | situation | what went wrong | better behavior -->
