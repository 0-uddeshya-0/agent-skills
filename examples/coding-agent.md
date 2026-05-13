# Example: Coding Agent

**Skills:** `code-map` → `assumption-check` → `simplify-scope` → `validate-output`

**Problem:** Agent reads wrong files first. Refactors more than requested. Ships code based on guessed function signatures. Explains things at wrong depth.

---

## The pattern

```python
def coding_agent(task, file_list, user_context):

    # 1. Map the codebase before reading anything
    graph = call_skill("code-map", file_list, detect_entry_points(file_list), task)
    relevant_files = read_files(graph["graph"]["task_relevant_files"])

    # 2. Check assumptions before writing any code
    plan = draft_initial_plan(task, relevant_files)
    assumptions = call_skill("assumption-check", plan, context=task)

    if not assumptions["safe_to_proceed"]:
        return f"Before I start: {assumptions['blocker']}\n\nCan you confirm?"

    # 3. Trim scope if it grew
    scope = call_skill("simplify-scope", plan, user_goal=task)
    final_plan = scope["simplified_task"]

    # 4. Generate the code
    code = generate_code(final_plan, relevant_files)

    # 5. Validate it actually solves the task
    validation = call_skill("validate-output", code, task, format_required="working code")
    if validation["score"] < 75:
        code = fix(code, validation["suggested_fix"])

    return code
```

---

## Graphify in action

**Task:** "Fix bug where subscription cancellation doesn't send confirmation email"

Without code-map — reads these files in order:
```
main.py          ← irrelevant
app/config.py    ← not the problem
app/routes/auth.py  ← irrelevant
app/utils/validators.py  ← irrelevant
... eventually finds billing.py and email_service.py
```
6 files read. Problem was in 2.

With code-map:
```
task_relevant_files: [
  "app/routes/billing.py",       ← cancellation logic
  "app/services/email_service.py", ← email sending
  "app/models/subscription.py",  ← shared model
  "app/config.py"                ← email config
]
safe_to_ignore: [auth.py, validators.py, formatters.py, all tests]
```
4 files read. Problem found in first pass.

---

## Assumption-check catching a bug before it ships

Plan: "Call `send_email()` after cancellation in `cancel_subscription()`"

Assumption-check output:
```json
{
  "assumptions": [{
    "statement": "email_service.send_email() exists and accepts a subscription object",
    "risk": "high",
    "source": "inferred",
    "verify_by": "Read email_service.py for actual function signatures"
  }],
  "safe_to_proceed": false,
  "blocker": "Wrong function signature would add a new bug while fixing the old one"
}
```

Agent reads email_service.py. Actual function: `send_cancellation_confirmation(user_id, plan_name)`.
Not `send_email`. Assumption-check prevented a new bug.
