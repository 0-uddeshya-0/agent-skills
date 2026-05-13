# Example: Customer Support Agent

**Skills:** `clarify-intent` → `route-or-answer` → `tool-call-guard` → `detect-contradiction` → `tone-match` → `humanize-output`

**Problem:** Agent makes 3 API calls when user just needed one answer. Returns corporate-speak to frustrated users. Occasionally tells users wrong facts about their own account.

---

## The pattern

```python
def support_agent(user_message, session_context, tools):

    # 1. Clarify before doing any work
    intent = call_skill("clarify-intent", user_message, session_context)
    if intent["needs_clarification"]:
        return intent["clarifying_question"]

    task = intent["clarified_task"]

    # 2. Do we even need a tool?
    route = call_skill("route-or-answer", task, tools, session_context)
    if route["decision"] == "answer_directly":
        raw = route["answer"]
    else:
        # 3. Guard the tool call
        guard = call_skill("tool-call-guard", route["tool_name"], build_params(task), is_destructive=True)
        if not guard["approved"]:
            return f"Need to check something first: {guard['issues'][0]}"

        tool_data = tools[route["tool_name"]].call(guard["corrected_params"] or build_params(task))
        raw = generate_answer(task, tool_data)

        # 4. Check for contradictions against source data
        check = call_skill("detect-contradiction", raw, tool_data)
        if not check["clean"]:
            raw = check["corrected_draft"]

    # 5. Match tone, then humanize
    toned = call_skill("tone-match", user_message, raw, brand_voice="friendly")
    final = call_skill("humanize-output", toned["adjusted_draft"], context="customer support")
    return final["humanized"]
```

---

## Before vs after

**User:** "Can you help with my account?"

Without skills:
> "Of course! I'd be happy to help with your account. I can assist with billing, profile settings, API keys, subscription management, usage statistics, team members, integrations, and more. What would you like help with today?"

With `/clarify-intent`:
> "Are you looking to update billing, change your profile, or manage API keys?"

---

**User:** "I got charged twice this month" (frustrated tone)

Without skills — 3 tool calls, 200 word response:
> "Thank you for bringing this to our attention! I can see your account has been charged $49 on March 1st and $49 on March 3rd. You are eligible for a refund for the duplicate charge. Our refund policy states refunds are processed within 5–7 business days..."

With skills — 1 tool call, 2 sentences:
> "That shouldn't have happened. I can see the duplicate $49 charge from March 3rd — refunding it now. Should be back on your card in 3–5 days."

| | Without | With |
|---|---|---|
| Tool calls | 3–4 avg | 1 avg |
| Response length | 180 words | 35 words |
| Factual errors | Occasional | Caught by detect-contradiction |
| Tone mismatch | Frequent | Eliminated by tone-match |
