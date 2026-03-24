---
name: reviewforest-widgets
description: Add ReviewForest review widgets and tree counters to any website. Use when a user wants to display ReviewForest reviews, ratings, tree counters, or badges on their site — either by fetching data from the ReviewForest API and rendering natively, or by embedding the official widget JavaScript. Also use when the user wants to manage ReviewForest widgets via API (create, update, list). Triggers on mentions of ReviewForest, review widgets, tree counter widgets, review badges, or forest page embeds.
---

# ReviewForest Widget Integration

Integrate ReviewForest review widgets into any website. Two approaches:

1. **Native rendering** — Fetch review data from the public API and render it in the user's own framework (React, Vue, HTML, etc.). No external JavaScript. Full styling control.
2. **JS embed** — Paste the official widget embed code. Zero code, styled by ReviewForest, customizable with CSS.

## Which Approach?

- **User is building a custom site** (React, Vue, Next.js, HTML) → Use **Approach 1: Native Rendering**
- **User wants zero code / quick embed** (WordPress, Squarespace, Wix, any CMS) → Use **Approach 2: JS Embed**
- **User wants to create or manage widgets programmatically** → Use **Authenticated API** section

## Prerequisites

Ask the user for their **widget UUID**. They get this from https://app.reviewforest.org → Widgets → select a widget → copy the UUID.

If they want to use the authenticated API (create widgets, list forests, plant trees), they also need an **API key** from Settings → API.

## Approach 1: Native Rendering (Recommended for Vibe Coders)

Fetch data from the public API and render reviews/badges in the user's framework. No external scripts needed.

### Step 1: Fetch Widget Data

```
GET https://api.reviewforest.org/v1/widgets/{uuid}
```

No auth required. Returns widget config + forest data including score, review count, tree count, platform info.

### Step 2: Fetch Reviews

```
GET https://api.reviewforest.org/v1/widgets/{uuid}/reviews?pageSize=10&sortBy=date&order=desc
```

No auth required. Returns paginated reviews with `name`, `score` (1-5), `text`, `date`, `platformType`.

Optional query params: `sortBy` (date/name/score), `order` (asc/desc), `pageSize` (10/15/20/25/50/100), `page`, `showReviewOnlyWithText` (true/false).

### Step 3: Render

Build UI components using the fetched data. Key data fields:

**From widget data** (`response.data[0]`):
- `data.score` — aggregate rating (string, e.g. "4.8")
- `data.reviewAmount` — total review count
- `data.totalTreeAmount` — trees planted
- `data.name` — business name
- `data.slug` — forest page slug (link to `https://reviewforest.org/{slug}`)
- `data.platforms` — array of connected platforms. Use `type` for the platform label (e.g. "google"). Note: `name` is NOT the platform name — it's the name of the business listing
- `config.appearance` — "light" or "dark"

**From reviews** (`response.data[]`):
- `name` — reviewer display name
- `score` — 1-5 star rating
- `text` — review text (may be empty)
- `date` — ISO date string
- `platformType` — source platform (google, facebook, trustpilot, etc.)

### Step 4: Track Impressions (Optional but Appreciated)

```
POST https://api.reviewforest.org/v1/widgets/{uuid}/events/impression
```

Call once when the widget becomes visible. Helps the customer see widget performance analytics.

### Example: Minimal Vanilla JS

```javascript
const UUID = 'WIDGET_UUID_HERE';
const API = 'https://api.reviewforest.org/v1/widgets';

async function loadReviewForest(containerId) {
  const [widgetRes, reviewsRes] = await Promise.all([
    fetch(`${API}/${UUID}`),
    fetch(`${API}/${UUID}/reviews?pageSize=5&showReviewOnlyWithText=true`)
  ]);

  const widget = (await widgetRes.json()).data[0];
  const reviews = (await reviewsRes.json()).data;
  const container = document.getElementById(containerId);

  // Build header
  const header = document.createElement('div');
  header.textContent = `${widget.data.score} ★ · ${widget.data.reviewAmount} reviews · 🌳 ${widget.data.totalTreeAmount} trees planted`;
  container.appendChild(header);

  // Build review cards
  reviews.forEach(r => {
    const card = document.createElement('div');
    const stars = '★'.repeat(r.score) + '☆'.repeat(5 - r.score);
    const name = document.createElement('strong');
    name.textContent = r.name;
    card.appendChild(name);
    card.appendChild(document.createTextNode(` ${stars} `));
    if (r.text) {
      const text = document.createElement('p');
      text.textContent = r.text;
      card.appendChild(text);
    }
    container.appendChild(card);
  });

  // Link to forest page
  const link = document.createElement('a');
  link.href = `https://reviewforest.org/${widget.data.slug}`;
  link.textContent = 'View our ReviewForest →';
  container.appendChild(link);

  // Track impression
  fetch(`${API}/${UUID}/events/impression`, { method: 'POST' });
}
```

### Platform Logos

Use Google's favicon service to show platform logos. Map `platformType` to domain:

```javascript
const PLATFORM_DOMAINS = {
  'google': 'google.com',
  'facebook': 'facebook.com',
  'trustpilot': 'trustpilot.com',
  'amazon': 'amazon.com',
  'kununu': 'kununu.com',
  'glassdoor': 'glassdoor.com',
  'apple-appstore': 'apps.apple.com',
  'google-playstore': 'play.google.com'
};

function platformLogoUrl(platformType, size = 32) {
  const domain = PLATFORM_DOMAINS[platformType];
  return domain ? `https://www.google.com/s2/favicons?domain=${domain}&sz=${size}` : null;
}
```

Use in `<img>` tags. Available sizes: 16, 32, 64, 128. Show next to reviewer names or in the widget header alongside platform names.

### Review Links

The widget data includes review links in `data.platforms[].keys.reviewLink`. Build a map by platform type and use it to link reviews back to the source platform:

```javascript
// Build from widget data
const reviewLinks = {};
widget.data.platforms.forEach(p => {
  if (p.keys?.reviewLink) reviewLinks[p.type] = p.keys.reviewLink;
});

// Use in review cards: reviewLinks[review.platformType]
```

### Design Guidance

- **Platform logo placement:** Show the platform logo + name inline with the star rating (same row), not as a separate footer element. This creates an immediate visual association between the rating and its source
- **Platform name:** Use `platforms[].type` for display (e.g. "Google"), NOT `platforms[].name` which is the business listing name, not the platform
- **Review links:** Make platform labels clickable, linking to `platforms[].keys.reviewLink`
- **Forest link:** Always link to the forest page: `https://reviewforest.org/{slug}`
- **Branding:** Include a "Powered by ReviewForest" footer or similar attribution
- **Appearance:** Use `config.appearance` to determine light/dark styling
- **Mobile:** Respect `config.hide_on_mobile` if rendering a floating widget
- **Star ratings:** Integer 1-5 per review, decimal string for aggregate (e.g. "4.8")
- Platform types: `google`, `facebook`, `trustpilot`, `amazon`, `kununu`, `glassdoor`, `apple-appstore`, `google-playstore`

## Approach 2: JS Embed (Zero Code)

Paste two tags into the site's `<head>` or before `</body>`:

```html
<script src="https://widgets.reviewforest.org/main.js" defer charset="UTF-8"></script>
<div class="reviewforest-app-WIDGET_UUID_HERE"></div>
```

Replace `WIDGET_UUID_HERE` with the actual UUID.

### CSS Customization

The widget renders in shadow DOM. To override styles, target the widget container:

```css
/* Position overrides for floating widgets */
.reviewforest-app-UUID {
  z-index: 9999 !important;
}
```

For deeper customization of the forest widget, see the help center article on CSS customization.

### Multiple Widgets

Add multiple `<div>` elements with different UUIDs. The loader script only needs to be included once.

```html
<script src="https://widgets.reviewforest.org/main.js" defer charset="UTF-8"></script>
<div class="reviewforest-app-UUID_1"></div>
<div class="reviewforest-app-UUID_2"></div>
```

## Authenticated API (Widget Management)

For creating and managing widgets programmatically. Requires an API key.

Read [references/api.md](references/api.md) for full endpoint documentation including:
- `GET /v1/forests` — List forests (to get forest IDs)
- `POST /v1/widgets` — Create a new widget
- `PATCH /v1/widgets/{uuid}` — Update widget config
- `GET /v1/widgets` — List all widgets
- `GET /v1/forests/{id}/trees` — Get planted trees
- `POST /v1/forests/{id}/trees` — Plant trees via API

All authenticated requests require `apikey` header.

## Important Notes

- **CORS:** Public endpoints support CORS — call them directly from browser JavaScript, no backend proxy needed
- **Error handling:** An invalid or inactive UUID returns `{ count: 0, data: [] }`. Always check that `data[0]` exists before accessing fields
- **Impression tracking:** `POST /v1/widgets/{uuid}/events/impression` returns HTTP 200 with an empty body — this is expected, not an error
- **Sort order:** When rendering natively, respect the widget's configured sort order (`config.reviews_sort_by` and `config.reviews_sort_order`) by passing them as query params to the reviews endpoint
- Widget UUIDs are v4 UUIDs, not MongoDB ObjectIds
- Forest slugs are URL-friendly strings (e.g., "my-business")
- Review `score` is an integer 1-5 per review; aggregate `score` in widget data is a string (e.g., "4.8")
- Reviews may have empty `text` — filter with `showReviewOnlyWithText=true` if needed
