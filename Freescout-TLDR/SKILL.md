---
name: freescout-TLDR
description: >
  Use this skill whenever the user provides a FreeScout conversation URL and wants to analyze a support ticket.
  Triggers on any message containing a FreeScout URL (e.g., https://support.example.com/conversation/12345),
  or when the user says things like "analyze this ticket", "what's this ticket about", "summarize this conversation",
  "check this support thread", or "what should I reply to this". This skill fetches the full ticket context
  from the FreeScout API, reads all threads chronologically, and produces a clear summary of the customer's
  current issue along with an actionable fix or suggested reply. Always use this skill when a FreeScout URL is present.
---

# FreeScout Support Ticket Analyzer

You are a technical customer support assistant. Your job is to fetch full ticket context from the FreeScout API, analyze the conversation thread, and provide a clear summary and action plan.

## Step 1: Extract the Conversation ID

Parse the conversation ID from the URL the user provided.

- URL format: `https://<domain>/conversation/{conversationId}?folder_id=...`
- Example: `https://support.example.com/conversation/14187?folder_id=104` → ID is `14187`

Extract only the numeric segment after `/conversation/`.

## Step 2: Authenticate

All API requests must include the header:

```
X-FreeScout-API-Key: <value of FREESCOUT_API_KEY environment variable>
```

Read the key at runtime using:
```javascript
const apiKey = process.env.FREESCOUT_API_KEY;
```
Or in bash:
```bash
FREESCOUT_API_KEY env var
```

The base URL for the API is the same domain as the conversation URL (e.g., `https://support.example.com`).

## Step 3: Fetch Data

Make two GET requests using the extracted conversation ID and base domain:

1. **Conversation metadata:**
   ```
   GET {base_url}/api/conversations/{conversationId}
   Headers: X-FreeScout-API-Key: {apiKey}
   ```

2. **Conversation threads (the actual messages):**
   ```
   GET {base_url}/api/conversations/{conversationId}/threads
   Headers: X-FreeScout-API-Key: {apiKey}
   ```

If either request fails (non-2xx), report the error clearly to the user — do not guess at the content.

## Step 4: Analyze the Thread

Read all threads returned from `/threads` in chronological order (sort by `createdAt` ascending if not already ordered).

**Focus on the most recent active issue:**
- Identify whether the latest messages introduce a **new problem** distinct from earlier issues in the thread.
- If a new problem exists, focus your analysis on that — treat earlier resolved issues as background context only.
- If the conversation is a single continuous issue, summarize the full arc but emphasize the current unresolved state.

**Key signals to look for:**
- Customer's latest message tone and explicit complaints
- Any recent error messages, logs, or screenshots described
- Whether the latest staff reply resolved the issue or left it open
- How long since the last customer or staff reply (urgency indicator)

## Step 5: Output

Respond using exactly this structure:

---

**Current Customer Issue:**
[A brief, clear explanation of the latest active problem the customer is facing. 2–4 sentences. Ignore previously resolved issues unless they're directly causing the current one.]

**Suggested Fix / Next Steps:**
[Actionable steps or a suggested reply to resolve this specific current issue. Be concrete — include specific settings, steps, or copy-pasteable reply text where appropriate.]

---

## Notes

- Never expose the API key in your output.
- If the threads endpoint returns an empty array, state that the ticket has no message threads yet.
- If the ticket is already resolved (status: `closed` or `resolved` with no open follow-up), note that in your summary and flag it clearly.
- Keep your output focused and scannable — this is a support tool, not an essay.
