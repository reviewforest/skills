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

Ask the user which approach they want:

- **Widget embed** — the user creates a widget at https://app.reviewforest.org/website-widgets/add (or copies the snippet from an existing one at https://app.reviewforest.org/website-widgets/installed), then provides the embed snippet (or just the UUID). See **Approach 1**.
- **Custom rendering** — the user creates an API key (read-only mode) at https://app.reviewforest.org/integrations/public-api. See **Approach 2**. Detailed API reference: [references/api.md](references/api.md)

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

**Important:** When creating the API key, select **read-only** mode. A read-only key is safe to use in client-side JavaScript since it can only read data, not modify anything.

All API requests require the `apikey` header:

```
apikey: YOUR_API_KEY
```

### Step 1: Get Forest ID

```
GET https://api.reviewforest.org/v1/forests
```

Returns `{ query, count, data: [Forest, ...] }`. Each forest has an `id` (string) needed for subsequent requests.

If the user has one forest, use it automatically. If multiple, show the list (use `name` to identify) and let the user choose.

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
- `treeNumbers` — tree counts by period (`thisWeek`, `thisMonth`, `thisYear`, `lastWeek`, `lastMonth`, `lastYear`)
- `platforms[]` — connected platforms, each with:
  - `type` — platform identifier (e.g. "google")
  - `typeDisplayName` — human-readable platform name (e.g. "Google")
  - `name` — business listing name on that platform (NOT the platform name)
  - `score` — platform-specific rating
  - `reviewAmount` — reviews on that platform

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
- `title` — review title (may be null)
- `text` — review text (may be null)
- `texts[]` / `ratings[]` — structured review (see Review Text Structure below)
- `date` — ISO date string
- `platformType` — source platform (see platform types below)

### Step 4: Render

Build UI components using the fetched data. What to render depends on the user's needs — a full review listing, just a score badge, a tree counter, or any combination.

### Review Text Structure

Reviews can have two formats:

- **Simple:** `text` field contains the full review text as a string
- **Structured:** `texts[]` and/or `ratings[]` arrays — review broken down by topics. Each element has `id` (topic key) and `text` (may contain basic HTML formatting — sanitize before rendering).

A review may have `texts` only, `ratings` only, or both — combine them when rendering. When rendering, check for `texts`/`ratings` arrays first. If present, render each topic with its heading and text. If neither array is present, fall back to the plain `text` field.

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
| `applePodcasts` | Apple Podcasts | Business |
| `g2` | G2 | Business |
| `gartner` | Gartner | Business |
| `provenexpert` | ProvenExpert | Business |
| `appleappstore` | App Store | Application |
| `googleplaystore` | Google Play | Application |
| `trustedshops` | Trusted Shops | Business |
| `reviewforest` | ReviewForest Direct | Business |
| `omr` | OMR | Business |

### Design Guidance

- **Platform name:** Use `platforms[].typeDisplayName` for the human-readable platform name, NOT `platforms[].name` which is the business listing name
- **Forest link:** Always link to the forest page: `https://reviewforest.org/{slug}`
- **Star ratings:** 1-5 per review; aggregate `score` on the forest (e.g. "4.8")
