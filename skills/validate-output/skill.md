# skill: validate-output

## purpose
Confirm that output actually answers the question asked, in the format requested, covering all required parts. Catches the two most common output failures: answering a different question, and ignoring format instructions.

## when_to_use
- Output has an explicit format requirement (JSON, numbered list, under N words)
- Task had multiple required parts and you want to verify all were covered
- Output will be parsed programmatically

## when_not_to_use
- Simple single-fact answers with no format requirement
- Creative tasks with no correct output
- You've already run this once on the same output — do not loop

## input
```
output: string              # The response to validate
original_task: string       # What was actually asked
format_required: string     # Format instructions ("JSON", "bullet list", "under 100 words")
required_parts: list        # Specific items that must be present (can be empty)
```

## output
```json
{
  "valid": true | false,
  "issues": [
    { "type": "wrong_answer|missing_part|wrong_format|too_long|too_short", "detail": "string" }
  ],
  "score": "int (0-100)",
  "fix_needed": true | false,
  "suggested_fix": "string | null"
}
```

## instructions

```
Check against four criteria:
1. ANSWERS THE QUESTION: Does it address what was asked? Not a related topic.
2. COVERS REQUIRED PARTS: Are all required_parts present?
3. CORRECT FORMAT: Is format_required followed exactly?
4. RIGHT LENGTH: Is length appropriate? Flag if obviously off.

Score: 100 = all met, 75-99 = minor issues, 50-74 = fix recommended, <50 = significant problems.

valid is true only if score >= 75 AND no wrong_answer issues.
suggested_fix is a single directive, not a rewrite. null if valid.

Return JSON only.
```

## constraints
- Max output tokens: 250
- suggested_fix: one directive only, not a full rewrite
- Do not rewrite the output — diagnose and instruct only

## example

**Input:**
```
output: "Tips for Python: use list comprehensions, avoid globals, write docstrings."
original_task: "List the 3 most common causes of memory leaks in Python web applications."
format_required: "numbered list"
required_parts: ["memory leaks", "web application context"]
```

**Output:**
```json
{
  "valid": false,
  "issues": [
    { "type": "wrong_answer", "detail": "Output gives general Python tips, not memory leak causes" },
    { "type": "missing_part", "detail": "No mention of memory leaks or web app context" },
    { "type": "wrong_format", "detail": "Requested numbered list, got comma-separated prose" }
  ],
  "score": 10,
  "fix_needed": true,
  "suggested_fix": "Rewrite: list exactly 3 causes of memory leaks in Python web apps (e.g. unclosed DB connections, circular references, unbounded caches) in a numbered list."
}
```

## feedback_log
<!-- date | situation | what went wrong | better behavior -->
