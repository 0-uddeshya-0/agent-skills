# skill: error-to-action

## purpose
Convert a raw error message into a plain-English explanation and a specific next action. Stops agents from returning cryptic error strings to users or getting stuck when a tool fails.

## when_to_use
- After a tool call fails or returns an unexpected error
- System response contains a status code or exception that must be handled
- Before presenting any error state to a user

## when_not_to_use
- Error is expected and already handled by your logic (404 that means "not found, skip")
- The "error" is a successful empty result (zero rows is not an error)
- Not enough context to determine action — ask the user instead

## input
```
error: string            # Raw error message, exception, or failure response
operation: string        # What was being attempted when this occurred
user_context: string     # Who the user is and what they were trying to do
retry_count: int         # How many times already retried
```

## output
```json
{
  "error_type": "string",
  "user_message": "string",
  "action": "retry|fix_input|escalate|skip|wait",
  "action_detail": "string",
  "retryable": true | false,
  "developer_note": "string"
}
```

## instructions

```
Classify the error and decide what to do.

Classification → action:
- auth/permission denied → escalate immediately (no retry, credentials invalid)
- not found (404) → fix_input or skip
- rate limit (429) → wait then retry
- validation error → fix_input (request was malformed)
- server error (500, 503) → retry once; escalate if persists
- timeout → retry once with simpler payload if possible
- network → retry once; escalate if persists

user_message rules: one sentence, plain English, say what happened and what happens next, no stack traces or error codes, no "an error occurred" — say specifically what failed.

retryable is false for: auth errors, permanent validation failures, after 2+ retries.

Return JSON only.
```

## constraints
- Max output tokens: 250
- user_message: one sentence, no technical jargon
- retryable false after 2 retries regardless of error type

## example

**Input:**
```
error: "429 Too Many Requests. Retry-After: 30"
operation: "Fetching billing data from billing_api"
user_context: "User viewing subscription page"
retry_count: 0
```

**Output:**
```json
{
  "error_type": "rate_limit",
  "user_message": "Fetching your billing info — it'll be ready in about 30 seconds.",
  "action": "wait",
  "action_detail": "Wait 30 seconds per Retry-After header, then retry same request unchanged.",
  "retryable": true,
  "developer_note": "billing_api 429. Retry-After: 30s. retry_count=0."
}
```

## feedback_log
<!-- date | situation | what went wrong | better behavior -->
