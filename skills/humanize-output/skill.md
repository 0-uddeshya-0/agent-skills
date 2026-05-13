# skill: humanize-output

## purpose
Strip the hollow AI-isms from a response — the filler openers, over-hedging, passive padding, and robotic structure that make users feel like they're talking to a template. Makes output read like a person wrote it.

## when_to_use
- Any user-facing response in chat, email, or support
- Output reads like it was written by a committee
- After applying other skills that produce formal or structured drafts

## when_not_to_use
- Output is structured data (JSON, tables, code) — don't humanize data
- Formal register is correct (legal, compliance, official communications)
- User explicitly wants formal language

## input
```
draft: string       # Response to humanize
context: string     # Who's receiving this and what the conversation is about
```

## output
```json
{
  "humanized": "string",
  "patterns_removed": ["string"]
}
```

## instructions

```
Remove these patterns:

HOLLOW OPENERS — cut entirely:
"Great question!" / "Certainly!" / "Absolutely!" / "Of course!" / "Sure thing!" / "I'd be happy to help with that."

OVER-HEDGING — just say the thing:
"It's worth noting that..." / "It's important to understand that..." / "Please be aware that..."

PASSIVE PADDING — make direct:
"It can be seen that X" → "X"
"There are many ways to approach this" → pick one and say it

HOLLOW CLOSES — cut or make specific:
"Let me know if you have any other questions!" → cut unless specific
"I hope this helps!" → cut
"Feel free to reach out!" → cut

STRUCTURE BLOAT:
- Remove bullets for 2 or fewer items — prose is cleaner
- Remove headers when there's only one section
- Cut the sentence that introduces a list ("Here are the steps:") and just start the list

Keep:
- Contractions (don't, can't, you'll) — they sound human
- First-person direct statements ("I'd check X first")
- Specific details over general statements

Do not change facts. Do not shorten below what's needed. Just make it direct.

Return JSON only.
```

## constraints
- Max output tokens: 400
- Do not change facts — phrasing and structure only
- humanized must still be complete and polite

## example

**Input:**
```
draft: "Great question! It's important to understand that there are several factors contributing to slow API response times. It should be noted that network latency, server load, and inefficient queries are potential causes. I'd be happy to help you investigate. Let me know if you have questions!"
context: "Developer asking why their API is slow"
```

**Output:**
```json
{
  "humanized": "Slow API responses usually come down to three things: network latency, server load, or inefficient queries. Check your query execution times in DB logs first — that's the most common culprit. If those look fine, check server load metrics. Which do you want to dig into?",
  "patterns_removed": [
    "'Great question!' — hollow opener, cut",
    "'It's important to understand that' — padding before the actual point",
    "'It should be noted that' — passive framing that says nothing",
    "'I'd be happy to help' — implies I wouldn't otherwise",
    "'Let me know if you have questions' — replaced with specific next step"
  ]
}
```

## feedback_log
<!-- date | situation | what went wrong | better behavior -->
