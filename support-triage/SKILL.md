---
name: support-triage
description: >
  Perform initial bug triage on a customer support ticket from Freescout. Use this skill whenever
  a user provides a Freescout ticket URL (e.g., https://support.example.com/conversation/XXXXX)
  and wants to triage, investigate, or link it to a GitHub issue. Triggers on phrases like
  "triage this ticket", "check this conversation", "link to GitHub", "investigate this support ticket",
  or any time a Freescout URL is pasted into chat. Automates the full workflow: fetching ticket data,
  classifying intent, searching matched GitHub repos, linking issues or debugging if none found.
---

# Support Triage Skill

Automates triage of Freescout support tickets: fetches conversation data, classifies intent,
maps to relevant GitHub repos, and either links an existing issue or helps debug a new one.

---

## Step 1 — Parse the Input URL

Extract the numeric conversation ID from the provided Freescout URL.

**Example:**
- Input: `https://support.example.com/conversation/14231?folder_id=74`
- Extracted ID: `14231`

---

## Step 2 — Fetch Conversation Data

```bash
curl -s -H "X-FreeScout-API-Key: $FREESCOUT_API_KEY" \
  "https://support.example.com/api/conversations/{extracted_id}"
```

Also fetch threads to get the full email body:

```bash
curl -s -H "X-FreeScout-API-Key: $FREESCOUT_API_KEY" \
  "https://support.example.com/api/conversations/{extracted_id}/threads"
```

From the response, extract:
- `subject` — ticket subject line
- `mailboxId` — used for repo mapping (Step 4)
- Full thread content — customer email bodies, replies, attachments text

---

## Step 3 — Intent Analysis

Read the full conversation. Classify the ticket strictly as one of:

- **Bug** — something is broken, not working as expected, throwing errors
- **Feature Request** — customer wants new functionality or behaviour change

Then extract **2–4 concise search terms** capturing the core technical issue.

**Example:**
- Subject: "Coupon not applying to bundled products after update"
- Intent: Bug
- Search terms: `coupon bundled products`, `discount not applying`, `cart rule bundle`

---

## Step 4 — GitHub Repository Mapping

Use the `mailboxId` from Step 2 to determine which GitHub org repos to search:

| mailboxId | Repositories (under org `your-org`) |
|-----------|--------------------------------------|
| `1`  | `plugin-repo-a` |
| `2`  | `plugin-repo-b`, `plugin-repo-c` |
| `3`  | `plugin-repo-d`, `plugin-repo-e` |
| `4`  | `plugin-repo-f`, `plugin-repo-g`, `plugin-repo-h`, `plugin-repo-i`, `plugin-repo-j`, `plugin-repo-k` |
| `5`  | `plugin-repo-l`, `plugin-repo-m`, `plugin-repo-n`, `plugin-repo-o`, `plugin-repo-p` |
| `6`  | `plugin-repo-q`, `plugin-repo-r`, `plugin-repo-s`, `plugin-repo-t`, `plugin-repo-u`, `plugin-repo-v`, `plugin-repo-w`, `plugin-repo-x`, `plugin-repo-y` |
| `7`  | `plugin-repo-z`, `plugin-repo-aa` |

---

## Step 5 — GitHub Search & Action

Search all mapped repositories for open issues matching the extracted terms.

```bash
# Search each repo for matching open issues
gh issue list --repo your-org/{repo} --state open --search "{search terms}" --limit 10
```

Run this for each repo in the mapped list. Collect all matching issues.

### If a match is found:

Leave a `+1` comment on the matching GitHub issue using the original ticket URL:

```bash
gh issue comment {issue_number} --repo your-org/{repo} --body "+1 {original_ticket_url}"
```

Report back to the user:
> ✅ **Match found:** Linked ticket to [your-org/{repo}#{issue_number}]({github_issue_url})
> Comment posted: `+1 {original_ticket_url}`

**Stop here** — do not proceed to Step 6.

### If no match is found:

Proceed to Step 6.

---

## Step 6 — Fallback Debugging & User Interaction

Re-read the full conversation threads looking specifically for technical context:

- PHP/JavaScript error logs or stack traces
- Debug log output
- Plugin version numbers
- WooCommerce / WordPress versions
- Steps to reproduce
- Specific settings or configurations mentioned
- Screenshots described in text

### If insufficient context:

Inform the user:
> ⚠️ **Insufficient technical data.** No existing GitHub issues were found, and the ticket doesn't contain enough technical context (e.g., error logs, debug output, or reproduction steps) to investigate further.
>
> **Suggested action:** Reply to the customer and ask them to provide:
> - A copy of their debug log (`WooCommerce > Status > Logs`)
> - Steps to reproduce the issue
> - Their WordPress, WooCommerce, and plugin version numbers

### If sufficient context exists:

Ask the user:
> 🔍 **No existing GitHub issues found**, but this ticket contains enough technical context to investigate.
>
> Would you like me to run a bug check and suggest a possible fix?

**Wait for user confirmation before proceeding.**

If the user agrees, analyze the technical context and generate a suggested fix. Prefer:
1. A targeted **PHP snippet** correcting the behaviour
2. A **WooCommerce/WordPress hook** (`add_filter`, `add_action`) as a mu-plugin or functions.php snippet
3. If it's a data/config issue, clear diagnostic steps

Format the output as:

---

### 🛠 Suggested Fix

**Issue summary:** {one-line description of root cause}

```php
// Suggested fix — add to functions.php or a mu-plugin
add_filter( 'hook_name', function( $value, $context ) {
    // fix logic here
    return $value;
}, 10, 2 );
```

**How this works:** {brief explanation}

---

Then append this instruction:

> 📋 **Please review this snippet.** If it resolves the issue, please raise a new **Bug Report** in the relevant GitHub repository with these details:
> - Ticket reference: `{original_ticket_url}`
> - Issue description
> - The fix above as a reference solution

---

## Notes

- Always use `$FREESCOUT_API_KEY` from the environment — never hardcode credentials.
- GitHub auth is via the ambient `gh` CLI context — no token needed in commands.
- If `gh` returns a permissions error, tell the user the CLI may not be authenticated to that repo.
- For mailbox IDs not in the mapping table, inform the user the mailbox isn't mapped and ask them to confirm the correct repositories manually.
- If the Freescout API returns a non-200 response, surface the error clearly and suggest the user verify the API key is set correctly in the environment.
