# skill: format-response

## purpose
Apply the correct output format based on where the response is going. Stops agents from returning markdown in a terminal, a wall of prose in Slack, or plain text when the caller expects JSON.

## when_to_use
- Output goes to a specific interface with formatting conventions (chat, terminal, email, Slack)
- Caller is a system that parses the response (JSON required)
- User specified a structure explicitly

## when_not_to_use
- No format specified and no interface information — default to plain prose
- Conversational or exploratory responses — heavy formatting distracts
- Response is a single sentence

## input
```
content: string          # Raw content to format
format_target: string    # "chat_ui"|"terminal"|"email"|"slack"|"json_api"|"markdown_doc"
audience: string         # "developer"|"end_user"|"system"
key_info: list[string]   # Items that must be visually prominent
```

## output
```json
{
  "formatted": "string",
  "format_used": "string"
}
```

## instructions

```
Apply formatting for the target. Rules:

chat_ui: short paragraphs (2–3 sentences), bullets only for 3+ parallel items, bold for key terms if markdown supported.

terminal: plain text only, no markdown symbols, indentation for structure, preserve line breaks.

email: plain text, short paragraphs, no hollow openers for transactional email.

slack: conversational, *bold* and `code` sparingly, no walls of text, no emoji unless requested.

json_api: valid JSON only — nothing before or after, camelCase keys, no markdown in string values.

markdown_doc: headers (##, ###), fenced code blocks with language hints, tables for comparisons.

Apply only what's needed. Do not over-format simple responses.
Do not add content — only format what's given.

Return JSON only.
```

## constraints
- Max output tokens: 500
- For json_api: output must be parseable JSON
- Do not add content — only format

## example

**Input:**
```
content: "Deployment failed because DATABASE_URL env var is not set in production. Add it to .env and restart. Format: postgresql://user:password@host:5432/dbname"
format_target: "slack"
audience: "developer"
key_info: ["DATABASE_URL"]
```

**Output:**
```json
{
  "formatted": "Deployment failed — `DATABASE_URL` env var missing in production.\n\nFix:\n1. Add to `.env`: `postgresql://user:password@host:5432/dbname`\n2. Restart the server.",
  "format_used": "Slack: inline code for variable names, numbered steps for actions, short sentences"
}
```

## feedback_log
<!-- date | situation | what went wrong | better behavior -->
