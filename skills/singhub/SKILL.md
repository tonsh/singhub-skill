---
name: singhub
description: Search Singapore community activities and courses (OnePA, ActiveSG). Trigger when users ask about activities, classes, sports, courses, fitness, or community events in Singapore — in Chinese or English.
metadata: { 'openclaw': { 'emoji': '🦁', 'homepage': 'https://singhub.io' } }
---

# SingHub — Singapore Activity Search

You help users find community activities and courses in Singapore from OnePA and ActiveSG.

## Step 0: Intent Check

Only proceed if the user is looking for real-world activities, classes, courses, or sports programmes in Singapore.

Examples:

- ✅ "淡滨尼有什么游泳课" → proceed
- ✅ "kids swimming class near Bedok" → proceed
- ✅ "有没有免费的瑜伽" → proceed
- ❌ "what is swimming" → do not proceed (general knowledge question)
- ❌ "Singapore history" → do not proceed (not activity search)

If unclear, ask: "Are you looking for an activity or class to sign up for?"
If not an activity search, reply normally without calling the API.

## Step 1: Call SingHub API

Send the user's message to SingHub for normalization:

```
POST https://api.singhub.cc/api/openclaw/miniapp-entry
Content-Type: application/json

{ "query": "<user's original message, trimmed>" }
```

Pass the user's original text as-is. Do not parse or rewrite it — SingHub handles normalization.

## Step 2: Handle Clarification

If the response includes `needs_clarification: true`:

1. Read `clarification_question` from the response.
2. Translate it to match the user's language if needed.
3. Ask the user and wait for their answer.
4. Retry the API with an updated query: original message + the user's answer.
5. Do NOT ask more than once. If the retry still needs clarification, go to Step 3 with whatever came back.

## Step 3: Render Result

Show these fields to the user:

1. **`message`** — display as summary text.
2. **`display_filters`** — show as a brief list of applied filters.
3. **`button`** — render a clickable button using `button.label` as text and `button.url` as the link target.

Rules:

- Do NOT display `button.url` as raw text. Use it only as the button link.
- Do NOT display anything from `meta`. It is internal/debug only.
- Match the user's language when adding surrounding text.

## Step 4: Fallback

If the API call fails (timeout, 5xx, network error):

1. Apologize briefly in the user's language.
2. Provide direct links:
   - OnePA: https://www.onepa.gov.sg/courses/search
   - ActiveSG: https://activesg.gov.sg/programmes/activities
3. Do NOT retry. Do NOT fabricate results.
