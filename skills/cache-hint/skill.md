# skill: cache-hint

## purpose
Tag which parts of agent output are safe to cache and reuse, and which are dynamic and must be recomputed. Eliminates redundant LLM calls in loops that repeat similar tasks.

## when_to_use
- Agent runs the same task type repeatedly (batch jobs, scheduled tasks)
- Multiple users ask structurally identical questions
- Output mixes stable information with dynamic data

## when_not_to_use
- Every task is unique — no repetition pattern exists
- Output is entirely user-specific or time-sensitive
- Single-turn interaction with no loop

## input
```
output: string       # Agent output to analyze
task_type: string    # What kind of task produced this
```

## output
```json
{
  "cacheable_parts": [
    { "content": "string", "reason": "string", "ttl": "permanent|24h|1h|session" }
  ],
  "dynamic_parts": [
    { "content": "string", "reason": "string" }
  ],
  "cache_key_fields": ["string"],
  "overall_cacheable": true | false
}
```

## instructions

```
Classify each meaningful piece of content as cacheable or dynamic.

Cacheable: same output for same input regardless of when or who asks.
Examples: query structure, code snippets, stable concept explanations, templates, error message mappings.

Dynamic: tied to a specific moment, user, or live data.
Examples: timestamps, user-specific values, prices, live API status, session IDs.

TTL guidance:
- permanent: facts that almost never change (language syntax, geography)
- 24h: pricing, plan features, documentation
- 1h: API responses for semi-live data
- session: valid only for this session's context

cache_key_fields: which input fields, if identical, guarantee the same cacheable output.

Return JSON only.
```

## constraints
- Max output tokens: 300
- ttl must be one of: permanent, 24h, 1h, session
- Only flag content actually worth caching — not every sentence

## example

**Input:**
```
output: "Your account (user_id: 8821) is on the Pro plan ($49/month). Pro plan includes unlimited API calls, priority support, and beta features. Next billing: March 15, 2025."
task_type: "account_info_lookup"
```

**Output:**
```json
{
  "cacheable_parts": [
    {
      "content": "Pro plan: unlimited API calls, priority support, beta features",
      "reason": "Plan features are same for all Pro users",
      "ttl": "24h"
    },
    {
      "content": "Pro plan price: $49/month",
      "reason": "Pricing changes infrequently",
      "ttl": "24h"
    }
  ],
  "dynamic_parts": [
    { "content": "user_id: 8821, next billing: March 15 2025", "reason": "User-specific and time-specific" }
  ],
  "cache_key_fields": ["plan_name"],
  "overall_cacheable": false
}
```

## feedback_log
<!-- date | situation | what went wrong | better behavior -->
