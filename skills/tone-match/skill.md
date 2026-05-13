# skill: tone-match

## purpose
Detect the emotional register of what the user wrote, then adjust the agent's response tone to fit. Stops agents from being formal when the user is casual, cheerful when the user is frustrated, or cold when the user needs acknowledgment.

## when_to_use
- User-facing agent in any domain
- User message carries emotional signal (frustration, urgency, confusion)
- Agent has been defaulting to one tone regardless of input

## when_not_to_use
- System-to-system communication
- Output is structured data — tone doesn't apply
- User has stated a preferred tone explicitly — use that instead

## input
```
user_message: string    # What the user wrote
agent_draft: string     # What the agent is about to say
brand_voice: string     # "professional"|"friendly"|"technical"|"casual" (optional)
```

## output
```json
{
  "detected_tone": {
    "register": "formal|casual|technical|emotional",
    "sentiment": "positive|neutral|frustrated|urgent|confused",
    "key_signals": ["string"]
  },
  "adjusted_draft": "string",
  "changes_made": ["string"]
}
```

## instructions

```
Read user message. Identify register and sentiment from actual signals in text: word choice, punctuation, sentence length, directness, capitalization.

Register signals:
- formal: full sentences, polite openers, professional vocabulary
- casual: contractions, short sentences, informal words
- technical: jargon, precision, specific terms, assumes shared knowledge
- emotional: expressive punctuation, strong adjectives, personal framing

Sentiment signals:
- frustrated: "still", "again", "why is this", "I've tried", multiple attempts mentioned
- urgent: "ASAP", "now", "immediately", short demanding sentences
- confused: "not sure", "I don't understand", "what does X mean"

Adjust rules:
- Match energy level, not mood. Don't mirror frustration — acknowledge then help.
- Match formality. If they wrote fragments, don't respond in full paragraphs.
- If frustrated: acknowledge first, then solve. Never start with "Great question!"
- If urgent: lead with the answer, not the explanation.
- If confused: short sentences, one concept at a time.
- Do not change facts — only tone, phrasing, structure.

Return JSON only.
```

## constraints
- Max output tokens: 350
- Do not change facts — only tone and phrasing
- No toxic positivity ("I totally understand your frustration! We love our customers!")

## example

**Input:**
```
user_message: "ive been trying to get this export working for 2 days. nothing works. support sent me here."
agent_draft: "Thank you for reaching out! To export, go to Settings > Data > Export and select your format. Let me know if you have questions!"
brand_voice: "friendly"
```

**Output:**
```json
{
  "detected_tone": {
    "register": "casual",
    "sentiment": "frustrated",
    "key_signals": ["2 days", "nothing works", "sent me here", "lowercase throughout"]
  },
  "adjusted_draft": "Two days on this is genuinely frustrating — let's fix it now.\n\nGo to Settings > Data > Export. If the page won't load or the button does nothing, tell me what you see and I'll walk you through it.",
  "changes_made": [
    "Removed 'Thank you for reaching out' — hollow when user is frustrated",
    "Acknowledged time spent before jumping to solution",
    "Replaced 'Let me know if you have questions' with specific offer to continue helping",
    "Removed exclamation marks — wrong register for this user's state"
  ]
}
```

## feedback_log
<!-- date | situation | what went wrong | better behavior -->
