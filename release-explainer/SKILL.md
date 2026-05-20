---
name: release-explainer
description: >
  Use this skill whenever the user provides a GitHub release URL and wants to understand what the release is about in plain language — no developer jargon. Triggers on any message containing a GitHub release URL (e.g., https://github.com/your-org/your-plugin/releases/tag/v2.0.7), or when the user asks things like "explain this release", "what's in this release", "what did this version fix", "what's new in this version", "walk me through the release notes", or "what do I need to do to apply this update". For each item in the release (bug fix, feature, improvement), fetch the linked pull request or issue, read the actual changes and discussion, then explain it in plain everyday language that a non-developer store owner or site admin could understand — what was broken and how it's fixed, or what the new feature does and how to use it in the WordPress/WooCommerce admin UI.
---

# Release Explainer Skill

Explains a GitHub plugin release in plain, non-technical language — covering what was fixed, what's new, and what (if anything) the user needs to do.

---

## Step 0 — Check `gh` CLI Authentication

Before doing anything else, verify the GitHub CLI is available and authenticated:

```bash
gh auth status
```

**If not authenticated**, tell the user:

> "I need to connect to GitHub to read the release details. You have two options:
>
> **Option A — Log in interactively** (one-time setup):
> Run this in your terminal and follow the prompts:
> ```
> gh auth login
> ```
> Choose **GitHub.com → HTTPS → Login with a web browser**, then paste the code shown.
>
> **Option B — Use a Personal Access Token**:
> 1. Go to https://github.com/settings/tokens and create a token with `repo` scope.
> 2. Set it as an environment variable: `export GH_TOKEN=your_token_here`
>
> Once done, let me know and I'll continue."

If the user sets `GH_TOKEN`, prepend it to all `gh` commands: `GH_TOKEN=... gh release view ...`

Do NOT proceed until `gh auth status` confirms a logged-in account.

---

## Step 1 — Fetch the Release

Given a release URL like `https://github.com/OWNER/REPO/releases/tag/vX.Y.Z`, extract `OWNER`, `REPO`, and `TAG`, then fetch:

```bash
gh release view TAG --repo OWNER/REPO --json tagName,name,body,publishedAt,isDraft,isPrerelease
```

Parse the `body` field — this is the release notes markdown. Extract every line item that contains a GitHub issue or PR link. Each line typically looks like:

```
* Bug Fix: Some description [#1234](https://github.com/OWNER/REPO/issues/1234)
```

Collect:
- The **type** (Bug Fix / Feature / Improvement / Enhancement)
- The **short description** from the line
- The **issue/PR number** (e.g. `#1234`)
- The **URL** to the issue or PR

---

## Step 2 — Fetch Each Issue / PR

For each item, determine whether the link goes to an **issue** or a **pull request** (the URL path contains `/issues/` or `/pull/`).

**For pull requests:**
```bash
gh pr view NUMBER --repo OWNER/REPO --json number,title,body,closingIssuesReferences,files,commits
```

**For issues:**
```bash
gh issue view NUMBER --repo OWNER/REPO --json number,title,body,comments
```

Read the body and any comments to understand:
- What the actual problem or request was
- How it was resolved or implemented
- Any special configuration, migration steps, or settings involved

You may need to fetch both the PR *and* the linked issue it closes to get the full picture.

**For any feature or improvement that adds or requires a setting:** search the plugin's settings files in the local codebase to find the exact menu path, tab, and field label before writing the explanation. Do not guess or use vague directions like "check the settings". If you cannot find the exact path in the code, say so explicitly rather than writing a vague instruction.

---

## Step 3 — Explain Each Item in Plain Language

For each item, write a clear, friendly explanation using **zero developer jargon**. Follow these templates:

### Bug Fix Template

**🐛 [Short title in plain words]**

*What was the problem?*
[Describe the bug as a real-world scenario the user might have experienced. E.g., "When you opened the shipping settings for a product and saved, the rows would appear blank the next time you reopened them."]

*What was fixed?*
[Plain explanation of the resolution. E.g., "This has been corrected — your saved shipping rows will now display properly every time you open the product settings."]

*Do you need to do anything?*
[Either "No action needed — this fix applies automatically once you update the plugin." OR specific steps if data migration, resaving settings, or manual action is required.]

---

### Feature / Improvement Template

**✨ [Short title in plain words]**

*What is this?*
[Describe the feature as if talking to a store owner. E.g., "Vendors can now automatically receive a summary of their earnings and commissions each month, without you having to send anything manually."]

*How does it work?*
[Walk through the UI/UX: where to find the setting, what options exist, what the end result looks like for admins and/or vendors.]

*Where do I find it?*
[The exact, verified navigation path — e.g. "WC Vendors → Settings → Statements" or "WC Vendors → Settings → Capabilities → General → PDF Invoices checkbox". This must be confirmed by searching the plugin's settings registration code, not guessed. Never write a vague direction like "check the WC Vendors settings".]

*Do you need to do anything to turn it on?*
[If the feature requires enabling a setting: state the exact full path to that setting — tab, section, and field label — confirmed from the codebase. If no action is needed, say so clearly. Never leave this as a vague "yes, configure it in settings".]

---

## Step 4 — Summary Header

Before the individual items, write a short paragraph summarising the release overall:

> "This update (v2.0.7) includes X bug fixes, Y new features, and Z improvements. Here's a plain-English breakdown of what changed..."

Include whether it's a **pre-release** or **stable release**.

---

## Step 5 — "What Should I Do?" Checklist

At the end, provide a simple action checklist:

> **Before updating:**
> - [ ] Back up your site
>
> **After updating:**
> - [ ] [Any specific steps from the release items — re-save settings, re-enable toggles, etc.]
> - [ ] [If nothing needed: "No extra steps required — just update the plugin and you're good to go!"]

---

## Tone & Language Rules

- **No jargon**: Avoid words like "refactor", "regression", "sparse array", "inheritance", "hook", "migration", "deprecated", "null pointer", etc.
- **Use real-world framing**: Describe bugs as things the user would actually experience in their WooCommerce admin or storefront.
- **Be specific about UI**: When a feature involves settings, name the exact menu path and what the fields/toggles look like. Always verify the path from the plugin's settings registration code — never guess or approximate. A vague direction like "check the WC Vendors settings" is not acceptable; the exact path (e.g. "WC Vendors → Settings → Statements" or "WC Vendors → Settings → Capabilities → General → tick PDF Invoices") must be confirmed and written out in full.
- **Be reassuring**: Most updates require no action — say so clearly to reduce anxiety.
- **Use emojis sparingly** for section headers (🐛 for bugs, ✨ for features/improvements) to aid scannability.

---

## Error Handling

- If `gh` is not installed: guide the user to install it (`apt install gh` on Linux, `brew install gh` on Mac, or download from https://cli.github.com).
- If a PR/issue is private or returns a 404: note it in the output and skip with a message: "I wasn't able to read the details for this item (#XXXX) — it may be private or unavailable."
- If the release body is empty: say so and suggest the user check the release page directly.
