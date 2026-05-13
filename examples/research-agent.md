# Example: Research / Summarization Agent

**Skills:** `compress-context` → `caveman-summary` → `prevent-redundancy` → `format-response`

**Problem:** Agent accumulates context until hitting token limits. Summaries are 80% as long as originals. Repeats the same points across sections.

---

## The pattern

```python
def research_agent(documents, question, output_target):
    context = ""

    for i, doc in enumerate(documents):

        # Compress accumulated context before adding more
        if len(context.split()) > 1500:
            compressed = call_skill("compress-context", context, current_task=question)
            context = compressed["compressed"]

        doc_summary = summarize(doc, question, context)
        context += f"\nDoc {i+1}: {doc_summary}"

    # Generate answer
    raw = llm(f"Answer: {question}", context)

    # Shortest version that keeps all meaning
    short = call_skill("caveman-summary", raw, max_sentences=6, preserve=extract_key_claims(raw))

    # Remove repetition
    clean = call_skill("prevent-redundancy", short["summary"], prior_context=context)

    # Format for destination
    formatted = call_skill("format-response", clean["clean_draft"], format_target=output_target, audience="developer")

    return formatted["formatted"]
```

---

## Token savings from compress-context

Processing 5 documents. Without compression:
```
After doc 1: 800 tokens
After doc 2: 1,900 tokens
After doc 3: 3,200 tokens  ← repetition starts
After doc 4: 4,800 tokens  ← model attention degrades
After doc 5: 6,500 tokens  ← near limit, expensive
```

With compression between each doc:
```
After doc 1:  800 tokens → compressed to 280
After doc 2:  950 tokens → compressed to 320
After doc 3: 1,050 tokens → compressed to 360
Final context for synthesis: 420 tokens
```

Context tokens across all calls: ~4,200 (compressed) vs ~17,200 (uncompressed). ~75% reduction.

---

## Caveman in action

**Question:** "What are the main reasons Python async code is hard to debug?"

Without caveman-summary:
> Python's asynchronous programming model, while powerful and efficient for I/O-bound tasks, introduces several categories of complexity that make debugging particularly challenging for developers. First and foremost, the non-linear execution flow that arises from coroutines and event loops can make it difficult to trace exactly when and in what order operations are executing...
> *(continues for 350 words)*

With caveman-summary (max_sentences: 5):
> Python async is hard to debug because execution order isn't obvious — coroutines run when the event loop decides, not top-to-bottom. Stack traces show the event loop, not your code path. Exceptions get swallowed silently if you forget to await. `pdb` doesn't work cleanly inside async functions. Use `asyncio.run()` plus structured logging with task IDs.

350 words → 65 words. Same information.
