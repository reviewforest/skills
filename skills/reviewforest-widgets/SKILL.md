---
name: reviewforest-widgets
description: Display your ReviewForest reviews on your website. Embed an official ReviewForest widget with a small code snippet, or fetch review data from the ReviewForest API to render a fully custom design. Use when someone wants to show ReviewForest reviews, ratings, or tree counters on their site.
---

# ReviewForest Widget Integration

Integrate ReviewForest review widgets into any website. Two approaches:

1. **Widget embed (recommended)** — Create and customize a widget in the ReviewForest dashboard, then add a small code snippet to the site. 7 widget types available. Works with any site.
2. **Custom rendering via API** — Fetch review data from the ReviewForest API and render it in the user's own framework (React, Vue, HTML, etc.). No external JavaScript. Full styling control.

## Which Approach?

- **Default for most cases** → Use **Approach 1: Widget Embed**. The user creates and configures the widget in the dashboard (appearance, which reviews to show, sorting, etc.), then adds the embed snippet. Works with any site — custom (React, Vue, Next.js) or CMS (WordPress, Squarespace, Wix).
- **User needs a fully custom design** that the pre-built widgets can't achieve → Use **Approach 2: Custom Rendering via API**

## Getting Started

Confirm the approach (default: **widget embed**) and collect the matching prerequisite:

- **Widget embed** — the user creates a widget at https://app.reviewforest.org/website-widgets/add (or copies the snippet from an existing one at https://app.reviewforest.org/website-widgets/installed), then provides the embed snippet (or just the UUID). See **Approach 1**.
- **Custom rendering** — the user creates an API key with the **website display** scope at https://app.reviewforest.org/integrations/public-api. See **Approach 2**. Detailed API reference: [references/api.md](references/api.md)

## Approach 1: Widget Embed (Recommended)

Create a widget at https://app.reviewforest.org/website-widgets/add, customize it, then install the embed snippet on the site. Existing widgets can be managed at https://app.reviewforest.org/website-widgets/installed.

### Embed Snippet

The snippet consists of two parts — a script tag (included once) and a div for each widget:

```html
<script src="https://widgets.reviewforest.org/main.js" defer></script>
<div class="reviewforest-app-WIDGET_UUID_HERE"></div>
```

The user will usually provide the full snippet with their UUID. If they only provide the UUID, construct the snippet above replacing `WIDGET_UUID_HERE`.

### Widget Types and Placement

| Widget | Supports Floating | Placement Notes |
|--------|:-----------------:|-----------------|
| Testimonials carousel | No | Place where user wants. Needs a wide container — use a full-width or centered container (like Bootstrap/Tailwind `container`). Avoid narrow fixed-width parents. |
| Review forest page | No | Full-page widget. Give it maximum width — ideally no container constraints, or only a centered container. Do not place inside narrow columns. |
| Review score badge | Yes | See floating vs fixed position below |
| Tree counter badge | Yes | See floating vs fixed position below |
| 2-in-1 badge | Yes | See floating vs fixed position below |
| Mini review score | No | Compact — place inline where user wants |
| Mini tree counter | No | Compact — place inline where user wants |

All widgets use container queries and will adapt to their parent's width, but large widgets (carousel, forest page) will look poor in narrow containers.

### Floating vs Fixed Position

Badge widgets (Review score badge, Tree counter badge, 2-in-1 badge) support two behaviors, configured in the dashboard. Ask the user which behavior they chose:

- **Floating:** Place the `<div>` before the closing `</body>` tag. The widget will float over the page in a fixed position.
- **Fixed position:** Place the `<div>` exactly where the user wants it in the page layout.

### Multiple Widgets

Include the script tag only once. Add multiple `<div>` elements with different UUIDs:

```html
<script src="https://widgets.reviewforest.org/main.js" defer></script>
<div class="reviewforest-app-UUID_1"></div>
<div class="reviewforest-app-UUID_2"></div>
```

## Approach 2: Custom Rendering via API

Fetch data from the ReviewForest API and render reviews in the user's framework. Use this when the pre-built widgets don't fit the user's design needs. No external scripts needed. Requires an API key.

### Prerequisites

The user needs an API key. Create one at https://app.reviewforest.org/integrations/public-api

**Important:** When creating the API key, choose the **website display** scope (the dedicated read-only scope for public website widgets — **not** "Full access"). It is restricted server-side to exactly the four forest read endpoints below (any other request returns `403`), and sensitive fields (billing, email, platform OAuth tokens, tree invoice numbers) are stripped from its responses. Because of that, the key is safe to use in client-side JavaScript — it will be visible in the page source, which is expected and safe with this scope.

The scope grants read-only access to exactly these endpoints — use only these:

- `GET /v1/forests`
- `GET /v1/forests/{forestId}`
- `GET /v1/forests/{forestId}/reviews`
- `GET /v1/forests/{forestId}/trees`

All API requests require the `apikey` header:

```
apikey: YOUR_API_KEY
```

### What can be displayed

Clarify with the user what they want to show. Available data:

- **Aggregate score** — overall rating (e.g. "4.8")
- **Review count** — total number of reviews
- **Tree count** — total, from reviews, additional, or broken down by period
- **Platform info** — connected platforms with per-platform scores and review counts
- **Individual reviews** — reviewer name, score, text, date, platform
- **Individual trees** — a feed of planted trees: planter name, date, and either the source review (score/text/platform) or the occasion for manually planted ones
- **Forest link** — link to the public forest page

If showing individual reviews, ask about:
- **Sorting** — by date or name
- **Filtering** — show only reviews with text, or all

### Step 1: Get Forest ID

```
GET https://api.reviewforest.org/v1/forests?pageSize=100&page=1
```

Returns `{ query, count, data: [Forest, ...] }`. Each forest has an `id` (string) needed for subsequent requests.

Always request `pageSize=100` when listing forests. If `count` is greater than the number of items returned, fetch additional pages until you have the full list.

If the user has one forest overall, use it automatically. If multiple, show the complete list (use `name` to identify) and let the user choose.

Query params: `sortBy` (createdAt/name/score/reviewAmount/totalTreeAmount), `order` (asc/desc), `pageSize` (10/15/20/25/50/100), `page`.

### Step 2: Get Forest Data

```
GET https://api.reviewforest.org/v1/forests/{forestId}
```

Returns the forest object directly (not wrapped in an array).

Key fields:
- `name` — business/forest name
- `slug` — forest page slug (link to `https://reviewforest.org/{slug}`)
- `score` — aggregate rating (string, e.g. "4.8")
- `reviewAmount` — total review count
- `totalTreeAmount` — total trees planted
- `reviewTreeAmount` — trees from reviews
- `additionalTreeAmount` — manually planted trees
- `treeNumbers` — tree counts by period (`thisPeriod`, `thisWeek`, `thisMonth`, `thisYear`, `lastPeriod`, `lastWeek`, `lastMonth`, `lastYear`)
- `platforms[]` — connected platforms, each with:
  - `type` — platform identifier (e.g. "google")
  - `typeDisplayName` — human-readable platform name (e.g. "Google")
  - `name` — business listing name on that platform (NOT the platform name)
  - `score` — platform-specific rating
  - `reviewAmount` — reviews on that platform

**Brand forests:** if the forest's `type` is `"brand"` it has **no** `platforms`/`platformsOrder`. Instead it returns a `channels[]` array — each channel is itself a forest object (with its own `platforms[]`). Guard for this before reading `platforms` (e.g. `(forest.platforms ?? forest.channels?.flatMap(c => c.platforms) ?? [])`), or stick to single forests. Regular forests have `type: "single"`.

### Step 3: Get Reviews (optional)

```
GET https://api.reviewforest.org/v1/forests/{forestId}/reviews
```

Only needed if the user wants to display individual reviews. If they only need the aggregate score, tree count, or platform info — Step 2 is enough.

Returns `{ query, count, data: [Review, ...] }`.

Query params: `sortBy` (date/name), `order` (asc/desc), `pageSize` (10/15/20/25/50/100), `page`, `showReviewOnlyWithText` (true/false — filter to only show reviews that have text).

Ask the user how they want reviews sorted and whether to show only reviews with text.

Key fields from each review:
- `name` — reviewer display name
- `score` — 1-5 star rating (integer)
- `text` — review text (may be null)
- `date` — ISO date string
- `platformType` — source platform (see platform types below)
- `url` — link to the review on the ReviewForest forest page

This endpoint returns the plain `text` only — it does **not** return `title` or the structured `texts[]`/`ratings[]` breakdown. For structured-review platforms (Kununu, Glassdoor, G2) `text` is often null here; the topic-by-topic content is available instead on review-type **trees** (Step 4). See [Review Text Structure](#review-text-structure).

### Step 4: Get Trees (optional)

```
GET https://api.reviewforest.org/v1/forests/{forestId}/trees
```

Only needed if the user wants to display the individual planted trees (a "tree feed") rather than just the aggregate tree counts from Step 2.

Returns `{ query, count, data: [Tree, ...] }`.

Query params: `sortBy` (date/name), `order` (asc/desc), `pageSize` (10/15/20/25/50/100), `page`.

Each tree has a `type`:
- **`review`** — planted from a review. Has `name` (planter), `date`, `score`, `title`, `text` (and possibly `texts[]`/`ratings[]`), `platformType`, and `url` (link to the original review on the source platform).
- **`additionalTree`** — manually planted. Has `name`, `date`, and `occasion` instead of review data.

Both also include `pageTreeUrl` (link to the tree on the forest page) and `plantingProject` (project name). `text` is user-generated content — the same XSS rules in Step 5 apply.

See [references/api.md](references/api.md) for the full Tree object.

### Step 5: Render

Build UI components using the fetched data. What to render depends on the user's needs — a full review listing, a tree feed, just a score badge, a tree counter, or any combination.

Use the user's framework and match existing code patterns. For React — create components, for Vue — use templates, for static HTML — use semantic markup with minimal JavaScript. Do not default to building DOM entirely through JavaScript unless the project already does this.

**Security:** API data includes user-generated content (review text, names) from external platforms. Never render it via `innerHTML` or other methods that allow HTML injection. Use safe methods: `textContent` in vanilla JS, `{{ }}` in Vue, `{}` in JSX (React escapes by default). Structured review text may contain basic HTML formatting — if you don't need that formatting, render it as plain text. If you do want to keep it, sanitize first with a library like [DOMPurify](https://github.com/cure53/DOMPurify), then pass the sanitized output to `v-html` (Vue) or `dangerouslySetInnerHTML` (React). Never feed raw API text into those APIs.

### Review Text Structure

Review content comes in two formats. **Note:** only the plain `text` form is returned by the `/reviews` endpoint. The structured form is available on review-type **trees** (from the `/trees` endpoint) and on the widget-embed data — so to render a topic-by-topic breakdown, read it from the trees feed.

- **Simple:** `text` field contains the full review text as a string
- **Structured:** `texts[]` and/or `ratings[]` arrays — the review broken down by topics. Each element has `id` (topic key) and `text` (may contain basic HTML formatting — sanitize before rendering).

An item may have `texts` only, `ratings` only, or both — combine them when rendering. Check for `texts`/`ratings` arrays first; if present, render each topic with its heading and text. If neither array is present, fall back to the plain `text` field.

See [references/api.md](references/api.md) for the full list of topic keys and their display labels.

### Platform Types

| Type | Display Name | Category |
|------|-------------|----------|
| `google` | Google | Business |
| `facebook` | Facebook | Business |
| `trustpilot` | Trustpilot | Business |
| `amazon` | Amazon | Product |
| `kununu` | Kununu | Employee |
| `glassdoor` | Glassdoor | Employee |
| `applepodcasts` | Apple Podcasts | Business |
| `g2` | G2 | Business |
| `gartner` | Gartner | Business |
| `provenexpert` | ProvenExpert | Business |
| `appleappstore` | App Store | Application |
| `googleplaystore` | Google Play | Application |
| `trustedshops` | Trusted Shops | Business |
| `reviewforest` | ReviewForest | Business |
| `omr` | OMR | Business |

### Platform Logos

To show platform logos, map `platformType` to domain and use Google's favicon service:

```
https://www.google.com/s2/favicons?domain=DOMAIN&sz=SIZE
```

| Platform Type | Domain |
|--------------|--------|
| `google` | google.com |
| `facebook` | facebook.com |
| `trustpilot` | trustpilot.com |
| `amazon` | amazon.com |
| `kununu` | kununu.com |
| `glassdoor` | glassdoor.com |
| `applepodcasts` | podcasts.apple.com |
| `g2` | g2.com |
| `gartner` | gartner.com |
| `provenexpert` | provenexpert.com |
| `appleappstore` | apps.apple.com |
| `googleplaystore` | play.google.com |
| `trustedshops` | trustedshops.com |
| `reviewforest` | reviewforest.org |
| `omr` | omr.com |

### Example (Reference Only)

This is a minimal reference for correct API usage patterns. Do not copy-paste — adapt to the user's framework and project structure.

```javascript
// Fetch forest data and reviews
const API = 'https://api.reviewforest.org/v1';
const headers = { apikey: 'USER_API_KEY' };

// Independent requests — fetch in parallel. Check res.ok in real code; on 401/404
// the forest may be missing, so hide the section rather than rendering "undefined".
const [forest, { data: reviews }] = await Promise.all([
  fetch(`${API}/forests/${forestId}`, { headers }).then(r => r.json()),
  fetch(`${API}/forests/${forestId}/reviews?pageSize=10&showReviewOnlyWithText=true`, { headers }).then(r => r.json()),
]);

// forest.score, forest.reviewAmount, forest.totalTreeAmount, forest.slug
// forest.platforms[].type, forest.platforms[].typeDisplayName, forest.platforms[].score
// (brand forests have no forest.platforms — use forest.channels[] instead)

// reviews[].name, reviews[].score, reviews[].text, reviews[].date, reviews[].platformType, reviews[].url
// (structured texts[]/ratings[] come from the /trees endpoint, not /reviews)
```

### Design Guidance

- **CORS:** The API supports CORS — call endpoints directly from browser JavaScript, no backend proxy needed
- **Platform name:** Prefer the API's `platforms[].typeDisplayName` for the human-readable platform name, NOT `platforms[].name` (the business listing name). If you map platform types yourself, fall back to capitalizing the raw `platformType` for any type not in the table above — the platform list grows over time
- **Error handling:** Check `res.ok`. The forest endpoints return `401` (missing/invalid key), `404` (unknown forest), or `400` (malformed id). On error, hide the widget rather than rendering `undefined`
- **Caching:** Review data changes infrequently. On static/SSR frameworks (Next.js, Astro, Nuxt) fetch at build time or server-side and revalidate periodically instead of on every page view; for pure client-side sites, a short `sessionStorage` cache avoids refetching on each navigation
- **Forest link:** Always link to the forest page: `https://reviewforest.org/{slug}`
- **Star ratings:** 1-5 per review; aggregate `score` on the forest (e.g. "4.8")
