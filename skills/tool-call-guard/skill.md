# skill: tool-call-guard

## purpose
Validate a tool call before it executes. Catches missing parameters, malformed values, and scope overreach. Blocks bad calls. Warns on irreversible ones. Prevents wasted API calls and accidental side effects.

## when_to_use
- Before every external tool call in an agent loop
- Tool call involves writes, deletes, or sends
- Input to the tool comes from user text that might be incomplete or malformed

## when_not_to_use
- Simple safe read operations where calling is always correct
- You've already validated this exact call this turn

## input
```
tool_name: string          # Tool you're about to call
tool_params: object        # Parameters you're about to pass
tool_schema: object        # Expected types/requirements (if known)
task_context: string       # Why you're making this call
is_destructive: boolean    # Creates, modifies, sends, or deletes data?
```

## output
```json
{
  "approved": true | false,
  "issues": ["string"],
  "safe_to_call": true | false,
  "warning": "string | null",
  "corrected_params": "object | null"
}
```

## instructions

```
Check this tool call before it executes.

Validate:
1. NECESSITY: Is this call needed for the task, or can the task be done without it?
2. PARAMETERS: Required params present? Types correct? Values in expected range?
3. SCOPE: Does this call do exactly what the task requires, or could it do more?
4. SIDE EFFECTS: If destructive, is scope limited to what was authorized?

Reject if: required param missing or null, value clearly wrong (negative ID, empty required field, wrong format), call would affect more data than intended.

Warn if: call is approved but irreversible.

Correct if: param has fixable format issue (whitespace, wrong case for enum, missing prefix). Never change values that affect intent.

safe_to_call is false if: not approved, OR is_destructive is true and warning is present.

Return JSON only.
```

## constraints
- Max output tokens: 200
- corrected_params: fix format only, never change intent
- safe_to_call false if destructive + has warning

## example

**Input:**
```
tool_name: "delete_user"
tool_params: { "user_id": "", "confirm": true }
tool_schema: { "user_id": "required non-empty string", "confirm": "required boolean" }
task_context: "Delete account for user ID 8821"
is_destructive: true
```

**Output:**
```json
{
  "approved": false,
  "issues": ["user_id is empty — required param missing. Context shows it should be 8821."],
  "safe_to_call": false,
  "warning": null,
  "corrected_params": { "user_id": "8821", "confirm": true }
}
```

## feedback_log
<!-- date | situation | what went wrong | better behavior -->
