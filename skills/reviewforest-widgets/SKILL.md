---
name: reviewforest-widgets
description: Display your ReviewForest reviews on your website. Embed an official ReviewForest widget with a small code snippet, or fetch review data from the ReviewForest API to render a fully custom design. Use when someone wants to show ReviewForest reviews, ratings, or tree counters on their site.
---

# ReviewForest Widget Integration

Integrate ReviewForest review widgets into any website. Two approaches:

1. **Widget embed (recommended)** — Create and customize a widget in the ReviewForest dashboard, then add a small code snippet to the site. Works with any site.
2. **Custom rendering via API** — Fetch review data from the ReviewForest API and render it in the user's own framework (React, Vue, HTML, etc.). No external JavaScript. Full styling control.

## Which Approach?

- **Default for most cases** → Use **Approach 1: Widget Embed**. The user creates and configures the widget in the dashboard (appearance, which reviews to show, sorting, etc.), then adds the embed snippet. Works with any site — custom (React, Vue, Next.js) or CMS (WordPress, Squarespace, Wix).
- **User needs a fully custom design** that the pre-built widgets can't achieve → Use **Approach 2: Custom Rendering via API**

## Getting Started

Confirm the approach (default: **widget embed**) and collect the matching prerequisite:

- **Widget embed** — the user creates a widget at https://app.reviewforest.org/website-widgets/add (or copies the snippet from an existing one at https://app.reviewforest.org/website-widgets/installed), then provides the embed snippet (or just the UUID). See **Approach 1**.
- **Custom rendering** — the user creates a widget in the dashboard (it decides which reviews show) and provides its UUID; its review feed and totals need no key. Only forest data the widget routes don't carry, such as per-platform scores, needs an API key with the access type **Website-Widgets** (https://app.reviewforest.org/integrations/public-api), used on the server or at build time. See **Approach 2**. Detailed API reference: [references/api.md](references/api.md)

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
| Review list | No | Large widget, like the carousel: give it a wide container. |

All widgets use container queries and will adapt to their parent's width, but large widgets (carousel, review list, forest page) will look poor in narrow containers.

The dashboard may offer widget types this table doesn't list yet. For an unlisted type, ask the user whether it is a large widget (give it a wide container) or a compact one (place it inline).

### Floating vs Fixed Position

Badge widgets (Review score badge, Tree counter badge, 2-in-1 badge) support two behaviors, configured in the dashboard. Ask the user which behavior they chose:

- **Floating:** Place the `<div>` right before the closing `</body>` tag. The badge then floats over the page in a bottom corner of the browser window (left or right, set in the dashboard). The `<div>` may also sit anywhere else inside `<body>`, as long as none of its parent elements has CSS such as `transform`, `filter`, `contain` or `container-type`, which would pin the badge to that element instead of the window.
- **Fixed position:** Place the `<div>` exactly where the user wants it in the page layout.

### Multiple Widgets

Include the script tag only once. Add multiple `<div>` elements with different UUIDs:

```html
<script src="https://widgets.reviewforest.org/main.js" defer></script>
<div class="reviewforest-app-UUID_1"></div>
<div class="reviewforest-app-UUID_2"></div>
```

### Content Security Policy

If the site sends a Content Security Policy (a `Content-Security-Policy` header or `<meta http-equiv>` tag), extend it, or the widgets stay empty. When you add a directive the policy doesn't have yet, the browser stops falling back to `default-src` for it, so copy those sources in too: a policy with only `default-src 'self'` becomes, for example, `script-src 'self' https://widgets.reviewforest.org`, or the site's own scripts, requests and images break. If the policy also sets `script-src-elem` or `style-src-elem`, add the same sources there too, because those take precedence for `<script>` and `<style>` elements. If it uses a nonce with `'strict-dynamic'`, host sources don't let the loader run: put the page's per-response nonce on the `<script src="https://widgets.reviewforest.org/main.js">` tag.

- `script-src`: `https://widgets.reviewforest.org`
- `connect-src`: `https://api.reviewforest.org` (widget data) and `https://cdn.comvi.io` (widget texts)
- `img-src`, if the policy restricts it: `https://widgets.reviewforest.org` (platform logos and icons) and the hosts of reviewers' profile photos. Today the photos load straight from Google and Facebook (`https://*.googleusercontent.com`, `https://*.ggpht.com`, `https://*.fbcdn.net`, `https://*.fbsbx.com`); after an upcoming ReviewForest release they load from `https://api.reviewforest.org` instead (photos already hosted on `reviewforest.org` or its subdomains keep their address). Allow both now so photos keep working across the switch, and keep the platform hosts until ReviewForest has announced the release and you have tested the narrower policy against real photo requests (a quiet console proves nothing while the old hosts are still allowed).
- Widgets still on version 1.0 also need `img-src data:` and `style-src 'unsafe-inline'`, because they inject a `<style>` tag and inline images. Updating them to version 2.0 in the dashboard avoids this. Where the style policy uses a nonce or hash, browsers ignore `'unsafe-inline'`, so version 1.0 widgets stay unstyled there: update them, or render the reviews yourself (Approach 2).
- The review forest page widget also loads a Mapbox map, more images and its own styles, so a strict policy needs more for it. Load the page and check the browser console for blocked requests.

## Approach 2: Custom Rendering via API

Fetch data from the ReviewForest API and render reviews in the user's framework. Use this when the pre-built widgets don't fit the user's design needs. No external scripts needed. Needs the UUID of a widget created in the dashboard; an API key only for forest data the widget routes don't return.

### Prerequisites

For a public page the user creates a widget at https://app.reviewforest.org/website-widgets/add (its settings decide which reviews show) and gives you its UUID. The widget's review feed and its totals need no key (see the example below). For forest data the widget routes don't carry, such as per-platform scores, the user needs an API key with the access type **Website-Widgets**. At https://app.reviewforest.org/integrations/public-api they click **Create new key**, set **Type of data access** to **Website-Widgets** (the default is Read-only) and click **Create key**. The account has to be activated first (plan chosen, payment details added); the dashboard asks for it otherwise.

**What this key can do:** it only reads, and only seven `GET` routes: the four forest routes below and three widget routes (`/v1/widgets`, `/v1/widgets/{uuid}`, `/v1/widgets/{uuid}/reviews`). Every other route that needs a key answers it with `401` or `403`, so it can't write anything or read invoices or user data. Even so, keep it out of the browser: call the forest routes on the server or at build time (see **Caching** below) and pass on only the fields the page shows. For a public page the keyless widget routes are enough (see the example below). (Calls to `/v1/widgets/{uuid}` and `/v1/widgets?uuids=…` count as impressions of those widgets, so don't poll them.)

**If the key ends up in a page anyway,** anyone who copies it can call those routes too, for **every** review forest in the account, not just the one on the page. They get forest names, scores and counts, display settings, connected platforms, reviews, the tree feed with reviewer names and photos, and the widget settings, plus the company ID, plan, subscription status and planting limit of the account (the widget routes return these even without a key) and its recurring extra-tree settings. The responses leave out email addresses, billing references and the access tokens of connected Google and Facebook accounts. Treat everything those routes return as public once the key is in a page.

Never put a key with the access type **Read-only** or **Full access** into frontend code: those also read invoices and account details, and Full access can change data. The dashboard's key list shows each key's type in the **Data access** column; if you're not sure which type a key the user gave you has, ask. Details: [references/api.md](references/api.md#the-website-widgets-key)

The custom rendering in this skill uses these four routes:

- `GET /v1/forests`
- `GET /v1/forests/{forestId}`
- `GET /v1/forests/{forestId}/reviews`
- `GET /v1/forests/{forestId}/trees`

Requests to these four routes need the key in the `apikey` header:

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

### Steps 1-4: the forest routes (server or build time only)

Steps 1-4 use the forest routes, which need the Website-Widgets key, so run them on the server or at build time and pass on only what the page shows. For reviews and totals on a public page, the keyless widget routes are enough; see the **Example** below and [Keyless widget routes](references/api.md#keyless-widget-routes).

### Step 1: Get Forest ID

```
GET https://api.reviewforest.org/v1/forests?pageSize=100&page=1
```

Returns `{ query, count, data: [Forest, ...] }`. Each forest has an `id` (string) needed for subsequent requests.

Always request `pageSize=100` when listing forests. If `count` is greater than the number of items returned, fetch additional pages until you have the full list.

If you start from a widget, use its `forestId` (from `GET /v1/widgets?uuids=`) instead of choosing a forest, so the data matches the widget's reviews. Otherwise: if the user has one forest overall, use it automatically. If multiple, show the complete list (use `name` to identify) and let the user choose.

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
- `reviewAmountOnPlatform` — total reviews on the source platforms (the count the badges show); `reviewAmount` counts only the reviews imported into ReviewForest
- `totalTreeAmount` — total trees planted
- `reviewTreeAmount` — trees from reviews
- `additionalTreeAmount` — manually planted trees
- `treeNumbers` — tree counts by period (`thisPeriod`, `thisWeek`, `thisMonth`, `thisYear`, `lastPeriod`, `lastWeek`, `lastMonth`, `lastYear`)
- `platforms[]` — connected platforms, each with:
  - `type` — platform identifier (e.g. "google")
  - `typeDisplayName` — human-readable platform name (e.g. "Google")
  - `name` — business listing name on that platform (NOT the platform name)
  - `score` — platform-specific rating
  - `reviewAmountOnPlatform` — reviews on that platform (`reviewAmount`: the ones imported into ReviewForest)

**Brand forests:** if the forest's `type` is `"brand"` it has **no** `platforms`/`platformsOrder`. Instead it returns a `channels[]` array — each channel is itself a forest object (with its own `platforms[]`). Guard for this before reading `platforms` (e.g. `(forest.platforms ?? forest.channels?.flatMap(c => c.platforms) ?? [])`), or stick to single forests. Regular forests have `type: "single"`.

### Step 3: Get Reviews (optional)

```
GET https://api.reviewforest.org/v1/forests/{forestId}/reviews
```

Only needed if the user wants to display individual reviews. If they only need the aggregate score, tree count, or platform info — Step 2 is enough.

Returns `{ query, count, data: [Review, ...] }`.

Query params: `sortBy` (date/name), `order` (asc/desc), `pageSize` (10/15/20/25/50/100), `page`, `showReviewOnlyWithText` (true/false — filter to only show reviews that have text).

Ask the user how they want reviews sorted and whether to show only reviews with text. `showReviewOnlyWithText=true` also counts structured text, so a few reviews (OMR reviews, for example) still arrive without `text`; skip those in your code as well.

Key fields from each review:
- `name` — reviewer display name
- `score` — 1-5 star rating; can have a decimal (OMR reviews, for example, come in halves like 4.5). Can be missing for a few imported reviews whose rating could not be read, so render stars only when `Number.isFinite(score)`
- `text` — review text (can be missing or null)
- `date` — ISO date string
- `platformType` — source platform (see platform types below)
- `url` — link to the review on the ReviewForest forest page

This endpoint returns the plain `text` only — it does **not** return `title` or the structured `texts[]`/`ratings[]` breakdown. For Kununu, Glassdoor and G2 that plain `text` is flattened, shortened or empty; see [Review Text Structure](#review-text-structure) for where the topic-by-topic content comes from. This is the forest's complete review list; the trees feed in Step 4 is not. For a review list on a public page, use the review feed of the forest's embed widget instead (`GET /v1/widgets/{uuid}/reviews`, see [references/api.md](references/api.md)): it is the list ReviewForest itself shows publicly, applies the widget's dashboard filters, leaves out reviews whose reviewer asked to be anonymized, and needs no key in the browser. Use `/reviews` only for views behind a login, such as an internal dashboard. Public pages, including pages rendered on the server or at build time, use the widget feed, with the widget's display settings applied before rendering names, text or stars.

### Step 4: Get Trees (optional)

```
GET https://api.reviewforest.org/v1/forests/{forestId}/trees
```

Only needed if the user wants to display individual tree records (a "tree feed") rather than just the aggregate tree counts from Step 2. The feed holds only the reviews that planted a tree (a forest can pause or cap planting) plus manually planted trees, so it is no substitute for the review list in Step 3.

Returns `{ query, count, data: [Tree, ...] }`.

Query params: `sortBy` (date/name), `order` (asc/desc), `pageSize` (10/15/20/25/50/100), `page`.

Each tree has a `type`:
- **`review`** — a tree for a review. Has `name` (planter), `date`, `score`, `title`, `text` (and possibly `texts[]`/`ratings[]`), `platformType`, and `url` (link to the original review on the source platform). Can have `profilePicture`, the reviewer's photo URL (see **Content Security Policy** under Design Guidance).
- **`additionalTree`** — manually planted. Has `name`, `date`, and `occasion` instead of review data.

Both also include `pageTreeUrl` (link to the tree on the forest page) and `plantingProject` (project name). `text` is user-generated content — the same XSS rules in Step 5 apply.

The feed can include trees that are not settled yet, and this key doesn't see that status, so take planted totals from the aggregate counts (`totalTreeAmount`) and don't label a single row as a confirmed planting.

See [references/api.md](references/api.md) for the full Tree object.

### Step 5: Render

Build UI components using the fetched data. What to render depends on the user's needs — a full review listing, a tree feed, just a score badge, a tree counter, or any combination.

Use the user's framework and match existing code patterns. For React — create components, for Vue — use templates, for static HTML — use semantic markup with minimal JavaScript. Do not default to building DOM entirely through JavaScript unless the project already does this.

**Security:** API data includes user-generated content (review text, names) from external platforms. Never render it via `innerHTML` or other methods that allow HTML injection. Use safe methods: `textContent` in vanilla JS, `{{ }}` in Vue, `{}` in JSX (React escapes by default). Structured review text may contain basic HTML formatting — if you don't need that formatting, render it as plain text. If you do want to keep it, sanitize first with a library like [DOMPurify](https://github.com/cure53/DOMPurify), then pass the sanitized output to `v-html` (Vue) or `dangerouslySetInnerHTML` (React). Never feed raw API text into those APIs.

### Review Text Structure

Review content comes in two formats. **Note:** only the plain `text` form is returned by the `/reviews` endpoint. The structured form comes with review-type **trees** (from the `/trees` endpoint), which cover only reviews that planted a tree, and with `GET /v1/widgets/{uuid}/reviews`, the review feed of an embed widget: every review that widget shows, after its dashboard filters, and public (no key needed). On a public page, take review names and texts only from the widget feed, also when you show them next to a tree: it leaves out reviews whose reviewer asked to be anonymized, while the tree feed can still carry another copy of the same review (a brand forest can hold one per channel). Read the tree feed's review content only in views behind a login.

- **Simple:** `text` field contains the full review text as a string
- **Structured:** `texts[]` and/or `ratings[]` arrays — the review broken down by topics. Each element usually has `id` (topic key) and `text` (may contain basic HTML formatting — sanitize before rendering); some Kununu entries are plain strings, so render a string entry as is, skip empty entries, and fall back to `text` when no topic content is left.

An item may have `texts` only, `ratings` only, or both — combine them when rendering. Check the `texts`/`ratings` arrays first; if they have entries, render each topic with its heading and text. If neither array has entries, fall back to the plain `text` field.

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
| `tripadvisor` | Tripadvisor | Business |

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
| `tripadvisor` | tripadvisor.com |

### Example (Reference Only)

This is a minimal reference for correct API usage patterns. Do not copy-paste — adapt to the user's framework and project structure.

```javascript
// Public page: the keyless widget routes, with the UUID of a widget the user created in
// the dashboard. No API key in the browser.
const API = 'https://api.reviewforest.org/v1';

async function loadWidgetReviews(widgetUuid) {
  // Check res.ok in real code; on 404 hide the section rather than rendering "undefined".
  const [{ data: [widget] }, { data: reviews }] = await Promise.all([
    // counts as one impression of the widget, like the embed's own request
    fetch(`${API}/widgets?uuids=${encodeURIComponent(widgetUuid)}`).then(r => r.json()),
    // the widget's dashboard filters applied, anonymized reviews left out; not counted as an impression
    fetch(`${API}/widgets/${encodeURIComponent(widgetUuid)}/reviews?pageSize=10`).then(r => r.json()),
  ]);
  if (!widget) return null; // unknown or deleted UUID (200 with an empty list): render nothing
  return { widget, reviews };
}

// widget.data.score, .reviewAmountOnPlatform (the review count the badges show; .reviewAmount
// counts imported reviews), .totalTreeAmount (for a brand forest whose campaign has ended, take the
// stored total from GET /v1/widgets/{slug}/counter instead), .slug, .displaySettings
// (widget.data.platforms ?? []) — single forests list their platforms there
// reviews[].name, .score (can be missing), .text, .texts[]/.ratings[] (structured reviews),
// .date, .platformType, .url, .profilePicture (use url/profilePicture only if they are http(s))

// Forest data the widget routes don't return (per-platform scores, typeDisplayName) needs the
// Website-Widgets key: fetch it on the server or at build time and pass on only what the page shows.
// Use the widget's own forest, so the extra data matches the reviews shown:
// const forest = await fetch(`${API}/forests/${widget.forestId}`, { headers: { apikey: process.env.RF_API_KEY } }).then(r => r.json());
// forest.platforms[].type, forest.platforms[].typeDisplayName, forest.platforms[].score
// (brand forests have no forest.platforms — use forest.channels[] instead)
```

### Design Guidance

- **CORS:** The API supports CORS, so the keyless widget routes can be called straight from browser JavaScript. Calls that need the key belong on the server or in the build.
- **Platform name:** Prefer the API's `platforms[].typeDisplayName` for the human-readable platform name, NOT `platforms[].name` (the business listing name). If you map platform types yourself, fall back to capitalizing the raw `platformType` for any type not in the table above — the platform list grows over time
- **Error handling:** Check `res.ok`. The forest endpoints return `401` (missing/invalid key), `404` (unknown forest, or one from another account), or `400` (malformed id). On error, hide the widget rather than rendering `undefined`
- **Display settings:** apply the forest's `displaySettings` the way the embed widgets do. If `showReviewerInitialOnly` is true, show only the reviewer's initials instead of `name`; if `showReviewTextAndScore` is false, leave out the review text and stars. The API returns full names either way, so this is up to your code
- **Content Security Policy:** if the site has one, `connect-src https://api.reviewforest.org` covers the API calls. Reviewer photos (`profilePicture` on review trees) also need `img-src`: today they load from Google's and Facebook's image servers (`https://*.googleusercontent.com`, `https://*.ggpht.com`, `https://*.fbcdn.net`, `https://*.fbsbx.com`), after an upcoming ReviewForest release from `https://api.reviewforest.org`. Allow both now, or leave the photos out. Platform logos from Google's favicon service need `img-src https://www.google.com https://*.gstatic.com` (it redirects there)
- **Caching:** Review data changes infrequently. On static/SSR frameworks (Next.js, Astro, Nuxt) fetch at build time or server-side and revalidate periodically instead of on every page view; for pure client-side sites, a short `sessionStorage` cache avoids refetching on each navigation
- **Links and photos:** before you put `url` or `profilePicture` into `href` or `src`, accept only `http:` and `https:` URLs and leave the link or photo out otherwise; the fields can also be missing. For example `const safeUrl = v => { try { const u = new URL(v); return ['http:', 'https:'].includes(u.protocol) ? u.href : null; } catch { return null; } };`. Escaping the text does not make a URL safe.
- **Forest link:** Always link to the forest page: `https://reviewforest.org/{slug}`
- **Star ratings:** 1-5 per review, sometimes with a decimal (render partial stars or round), and skip the stars when a review has no `score`; aggregate `score` on the forest (e.g. "4.8")
