---
name: rymera-kb-writer
description: >
  Creates or updates WordPress knowledge base articles for the Wholesale Suite plugin or Ad Tribes plugin via the REST API. Use this skill whenever the user wants to write, update, draft, or publish a KB article for Wholesale Suite or Ad Tribes — even if they only say things like "write a KB article about X", "update this KB post", "draft an article about [feature]", or paste a wp-admin edit URL. Handles article structure, Gutenberg block markup, and REST API communication automatically.
---

# Wholesale Suite / Ad Tribes KB Writer

Write or update knowledge base articles for the Wholesale Suite or Ad Tribes WordPress plugins, saving them as drafts via the WP REST API.

---

## Step 0: Identify the Target Site

Determine which site to use **before** any API calls:

| Signal | Site |
|--------|------|
| User mentions "Ad Tribes", "adtribes", "adtribes.io", or provides a URL containing `adtribes.io` | **Ad Tribes** → `https://adtribes.io` |
| User mentions "Wholesale Suite", "wholesalesuite", or provides a URL containing `wholesalesuiteplugin.com` | **Wholesale Suite** → `https://wholesalesuiteplugin.com` |
| Ambiguous — no clear signal | **Ask** the user which site they mean |

Site configuration:

### Wholesale Suite
- Base URL: `https://wholesalesuiteplugin.com`
- Username env var: `wholesalesuite_application_user`
- Password env var: `wholesalesuite_application_pass`
- Reference article ID: `YOUR_REFERENCE_ARTICLE_ID`
- Post type endpoint: `/wp-json/wp/v2/ht-kb`

### Ad Tribes
- Base URL: `https://adtribes.io`
- Username env var: `adtribes_application_username`
- Password env var: `adtribes_application_password`
- Reference article ID: `YOUR_REFERENCE_ARTICLE_ID`
- Post type endpoint: `/wp-json/wp/v2/ht-kb`

---

## Step 1: Resolve the Target Post

**If the user provides a wp-admin URL** (e.g. `/wp-admin/post.php?post=164823&action=edit`):
- Extract the post ID from the `post` query parameter
- Fetch the current article: `GET /wp-json/wp/v2/ht-kb/{id}`
- This is an **update** — PATCH the existing post

**If the user provides a post ID directly**: treat as update.

**If no URL or ID is given**: this is a **create** — POST to `/wp-json/wp/v2/ht-kb`.

---

## Step 2: Fetch Example Block Structure

Before writing any content, fetch a reference article to match its Gutenberg block structure.

Use the site config resolved in Step 0:

```
GET {base_url}{post_type_endpoint}/{reference_article_id}
Authorization: Basic {base64(username:password)}
```

Use the `content.raw` field of the response as your block format reference.

---

## Step 3: Write the Article

Follow the structure and style rules below exactly.

### Writing Style
- Professional but approachable — written for store owners, not developers
- Second person: "you", "your products", "your order form"
- Short, scannable sentences
- **Bold** all UI labels, setting names, navigation paths, and option names exactly as they appear in the UI
- Avoid jargon unless you define it immediately after

---

### Article Structure

#### Introduction (2–3 paragraphs as `wp:paragraph` blocks)
1. What the feature/setting does and why it matters to a wholesale store owner
2. What problem it solves or what flexibility it provides
3. If there's a newer version of the feature, briefly note it and mention a legacy section at the bottom

#### Requirements (`wp:heading` level 2 + `wp:list`)
Bulleted list covering:
- Which plugin(s) must be installed
- Any prior setup needed (link to related KB articles where applicable)
- Version requirements if relevant

#### Content (`wp:heading` level 2 + mixed blocks)
Main instructional body:
- Lead with the navigation path in bold using → arrows:
  `**Wholesale → Order Form → Edit form → Settings → Product Sorting By**`
- After each step, insert an image placeholder in brackets describing what to show:
  `["image of the Product Sorting By dropdown with Menu Order selected"]`
  Use `wp:image` blocks with a caption for these placeholders.
- For lists of options/settings, use numbered or bulleted lists with option name in bold + dash + explanation:
  ```
  1. **Menu Order** - use this when you need custom per-product ordering.
  ```
- For options needing extra explanation or sub-settings, add an H3 sub-section
- Call out setting dependencies explicitly (e.g., "The **Popularity Period** field only appears when **Popularity** is selected")
- If legacy versions differ, add a separate H2 section: **"Legacy [Feature Name] (version X.X.X and below)"**

#### Frequently Asked Questions (`wp:heading` level 2 + alternating `wp:heading`/`wp:paragraph` blocks)
Generate **6–10 FAQs** covering:
- Edge cases not in the main content (e.g., "What happens if a product has no SKU when sorting by SKU?")
- Common points of confusion
- Default behavior clarifications
- Role-specific or multi-form behavior
- Caching / "why isn't it working" questions

Format: bold H3 question, then paragraph answer.

#### Help & Support (`wp:heading` level 2 + `wp:paragraph` blocks)
Use the site-specific closing text below. Replace `[feature/feed name]` in the Ad Tribes version with the specific feature or feed name covered by the article.

**Wholesale Suite:**

> We have a dedicated support team for Wholesale Suite who knows our products, WooCommerce, and the industry very well. You're welcome to make use of their expertise at any time, worldwide.
>
> If you are an existing customer please go to the [support ticket request form](https://wholesalesuiteplugin.com/my-account/support/) and send us a message.
>
> If you are a free plugin user, please [send us a support request on the forum](https://wordpress.org/support/plugin/woocommerce-wholesale-prices/). We actively monitor the WordPress.org support forums for the free plugin and help our users there as best as we can.

**Ad Tribes:**

> If you need any further assistance with the [feature/feed name], feel free to [open a support ticket](https://adtribes.io/submit-ticket/) to reach out to our support team.
>
> Please note that the ticket support system is exclusive to Product Feed Elite users.
>
> If you're using only the Product Feed Pro plugin, feel free to [open a new topic in the WordPress.org forum](https://wordpress.org/support/plugin/woo-product-feed-pro/#new-topic-0).

---

## Step 4: Format as Gutenberg Block Markup

Convert your article to WordPress Gutenberg block HTML (the `content` field). Follow the block patterns from the reference article fetched in Step 2. Key block types:

```html
<!-- wp:paragraph -->
<p>Text here</p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":2} -->
<h2 class="wp-block-heading">Section Title</h2>
<!-- /wp:heading -->

<!-- wp:heading {"level":3} -->
<h3 class="wp-block-heading">Subsection</h3>
<!-- /wp:heading -->

<!-- wp:list -->
<ul class="wp-block-list"><!-- wp:list-item -->
<li><strong>Option Name</strong> - description</li>
<!-- /wp:list-item --></ul>
<!-- /wp:list -->

<!-- wp:list {"ordered":true} -->
<ol class="wp-block-list"><!-- wp:list-item -->
<li><strong>Step</strong> - do this</li>
<!-- /wp:list-item --></ol>
<!-- /wp:list -->

<!-- wp:image -->
<figure class="wp-block-image"><img src="" alt=""/><figcaption class="wp-element-caption">[image placeholder description]</figcaption></figure>
<!-- /wp:image -->
```

---

## Step 5: Save via REST API

Use the `{base_url}` and `{post_type_endpoint}` resolved in Step 0.

### Create (new article)
```
POST {base_url}{post_type_endpoint}
Authorization: Basic {base64(username:password)}
Content-Type: application/json

{
  "title": "Article Title",
  "content": "<!-- wp:paragraph -->...",
  "status": "draft"
}
```

### Update (existing article)
```
PATCH {base_url}{post_type_endpoint}/{id}
Authorization: Basic {base64(username:password)}
Content-Type: application/json

{
  "title": "Article Title",
  "content": "<!-- wp:paragraph -->...",
  "status": "draft"
}
```

> ⚠️ **Never set `status` to `"publish"`**. Always use `"draft"`.

---

## Step 6: Confirm and Report

After saving, confirm:
- ✅ Post ID
- ✅ Status is `draft`
- ✅ Link to the wp-admin edit URL: `/wp-admin/post.php?post={id}&action=edit`

---

## Authentication Notes

- Determine the base URL and credentials from the site resolved in Step 0.
- Encode credentials as `base64(username + ":" + password)` for the Basic Auth header.
- Use the Bash tool to read env vars and make `curl` requests.

### ⚠️ Credential Safety Rules

**Never** run `printenv`, `env`, or `set` without filtering to existence checks — they print raw values into the tool output which is visible in the UI.

To check whether a credential is set, use only:
```bash
[ -n "$(printenv adtribes_application_username)" ] && echo "set" || echo "not set"
```

**Never** echo, print, or log the value of any credential variable. Only ever pass credentials directly into `curl` via the `AUTH` variable — never output them separately.

## Bash Examples

### Wholesale Suite
```bash
WP_USER=$(printenv wholesalesuite_application_user)
WP_PASS=$(printenv wholesalesuite_application_pass)
BASE_URL="https://wholesalesuiteplugin.com"
ENDPOINT="/wp-json/wp/v2/ht-kb"
AUTH=$(echo -n "$WP_USER:$WP_PASS" | base64)

# Fetch reference article
curl -s -H "Authorization: Basic $AUTH" \
  "$BASE_URL/wp-json/wp/v2/ht-kb/$REFERENCE_ID"

# Create new draft
curl -s -X POST \
  -H "Authorization: Basic $AUTH" \
  -H "Content-Type: application/json" \
  -d '{"title":"My Article","content":"<!-- wp:paragraph --><p>Hello</p><!-- /wp:paragraph -->","status":"draft"}' \
  "$BASE_URL$ENDPOINT"

# Update existing draft
curl -s -X PATCH \
  -H "Authorization: Basic $AUTH" \
  -H "Content-Type: application/json" \
  -d '{"content":"<!-- wp:paragraph --><p>Updated</p><!-- /wp:paragraph -->","status":"draft"}' \
  "$BASE_URL$ENDPOINT/$POST_ID"
```

### Ad Tribes
```bash
WP_USER=$(printenv adtribes_application_username)
WP_PASS=$(printenv adtribes_application_password)
BASE_URL="https://adtribes.io"
ENDPOINT="/wp-json/wp/v2/ht-kb"
AUTH=$(echo -n "$WP_USER:$WP_PASS" | base64)

# Fetch reference article
curl -s -H "Authorization: Basic $AUTH" \
  "$BASE_URL/wp-json/wp/v2/ht-kb/$REFERENCE_ID?context=edit"

# Create new draft
curl -s -X POST \
  -H "Authorization: Basic $AUTH" \
  -H "Content-Type: application/json" \
  -d '{"title":"My Article","content":"<!-- wp:paragraph --><p>Hello</p><!-- /wp:paragraph -->","status":"draft"}' \
  "$BASE_URL$ENDPOINT"

# Update existing draft
curl -s -X PATCH \
  -H "Authorization: Basic $AUTH" \
  -H "Content-Type: application/json" \
  -d '{"content":"<!-- wp:paragraph --><p>Updated</p><!-- /wp:paragraph -->","status":"draft"}' \
  "$BASE_URL$ENDPOINT/{id}"
```
