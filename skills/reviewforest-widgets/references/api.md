# ReviewForest API Reference

Base URL: `https://api.reviewforest.org`

Full interactive documentation: https://reviewforest.org/apidoc/ | Swagger: https://reviewforest.org/apidoc/swagger.json

## Authentication

The forest endpoints in this reference require the `apikey` header:

```
apikey: YOUR_API_KEY
```

### The Website-Widgets key

For website code, use a key with the access type **Website-Widgets**. Create it at https://app.reviewforest.org/integrations/public-api: click **Create new key**, set **Type of data access** to **Website-Widgets** (the default is **Read-only**), then click **Create key**. The account has to be activated first (plan chosen, payment details added); the dashboard asks for it otherwise. The key list shows each key's type in the **Data access** column. In the API this access type is the scope `website-display`.

A Website-Widgets key can only read, and only these seven routes:

| Route | Returns |
|-------|---------|
| `GET /v1/forests` | all review forests in the account |
| `GET /v1/forests/{forestId}` | one forest with its platforms |
| `GET /v1/forests/{forestId}/reviews` | a forest's reviews |
| `GET /v1/forests/{forestId}/trees` | a forest's planted trees |
| `GET /v1/widgets` | the account's widgets and their settings |
| `GET /v1/widgets/{uuid}` | one widget's settings and summary data |
| `GET /v1/widgets/{uuid}/reviews` | the reviews one widget shows, after its dashboard filters. Public: it answers without any key |

Any other route that needs a key answers this one with `401` or `403`. It can't write either: wherever a key is needed, methods other than `GET` get `403`. The custom rendering in the skill needs only the four forest routes. Two widget calls answer without any key, because the embed widgets make them from every visitor's browser: `GET /v1/widgets/{uuid}/reviews` and `GET /v1/widgets?uuids=…`. Each call to `GET /v1/widgets/{uuid}` or `GET /v1/widgets?uuids=…` counts as an impression of those widgets, so don't poll them.

For this key type the responses leave out email addresses, billing and subscription references, and the access tokens and owner details of connected Google and Facebook accounts. Trees come without `invoiceNumber` and `unPaid`. Everything else the seven routes return stays readable: forest names and slugs, scores and counts, display settings, connected platforms with their listing IDs (`platforms[].keys`), reviews, trees with reviewer names and photos, widget settings, the account's company ID, plan, subscription status and planting limit (the widget routes return these even without a key), and recurring extra-tree settings.

**Once the key is in a page, treat all of that as public.** Anyone can copy the key from the page source and call these routes, for **every** review forest in the account, not only the one the page shows. So call these routes on the server or at build time and keep the key out of the browser; for a public page the keyless widget routes are enough. Create a separate key for the website rather than reusing an existing one, so it can be deleted on its own. Never put a **Read-only** or **Full access** key into frontend code: those keys also read invoices and account details, and Full access can change data.

## Pagination

The forest list endpoints support pagination via query parameters:

| Parameter | Type | Default | Values |
|-----------|------|---------|--------|
| `pageSize` | number | `10` | `10`, `15`, `20`, `25`, `50`, `100` |
| `page` | number | `1` | any positive integer |

Paginated responses include:

```json
{
  "query": { "pageSize": 10, "page": 1, "sortBy": "...", "order": "..." },
  "count": 95,
  "data": [...]
}
```

`count` is the total number of items. Use `count` and the `pageSize` from the response's `query` to calculate total pages. The widget review feed (`GET /v1/widgets/{uuid}/reviews`) uses the widget's own reviews-per-page setting when you don't pass `pageSize`, so take the page size from its response too, never assume 10.

---

## GET /v1/forests

List all forests for the authenticated account.

When letting a user choose a forest, request `pageSize=100` first. If `count` is still larger than the number of returned items, fetch additional pages until you have the full list.

**Query Parameters:**

| Parameter | Type | Default | Values |
|-----------|------|---------|--------|
| `sortBy` | string | `createdAt` | `additionalTreeAmount`, `createdAt`, `name`, `score`, `reviewAmount`, `reviewTreeAmount`, `totalTreeAmount` |
| `order` | string | `desc` | `asc`, `desc` |
| `pageSize` | number | `10` | `10`, `15`, `20`, `25`, `50`, `100` |
| `page` | number | `1` | any positive integer |
| `platformTypes` | string | _(omit)_ | comma-separated platform types; omit to include all. Values must be valid platform types — passing `all` returns `400` |

**Response:**

```json
{
  "query": { "sortBy": "createdAt", "order": "desc", "pageSize": 10, "page": 1 },
  "count": 2,
  "data": [
    {
      "id": "608a814677b4f80db24913a2",
      "name": "My Business",
      "type": "single",
      "category": "business",
      "slug": "my-business",
      "slugId": "123456",
      "active": true,
      "score": "4.8",
      "reviewAmount": 120,
      "reviewAmountOnPlatform": 3193,
      "totalTreeAmount": 142,
      "reviewTreeAmount": 120,
      "additionalTreeAmount": 22,
      "plantingReviewLimit": -1,
      "isPlantingTrees": false,
      "treeNumbers": {
        "thisPeriod": 5,
        "thisWeek": 3,
        "thisMonth": 12,
        "thisYear": 85,
        "lastPeriod": 0,
        "lastWeek": 4,
        "lastMonth": 15,
        "lastYear": 57
      },
      "displaySettings": {
        "allowDownloadingTreeCertificate": false,
        "showTreeAges": true,
        "showReviewerInitialOnly": false,
        "showReviewTextAndScore": true,
        "useFormalAddressing": false,
        "logo": "https://..."
      },
      "plantingProjects": [
        { "projectId": 2, "trees": 128 },
        { "projectId": 1, "trees": 14 }
      ],
      "platforms": ["platformId1", "platformId2"],
      "platformsOrder": ["google", "facebook"],
      "pinnedPlatform": "google",
      "projectId": 4,
      "createdAt": "2024-01-15T10:00:00.000Z",
      "updatedAt": "2025-12-20T14:30:00.000Z"
    }
  ]
}
```

Note: In the list endpoint, `platforms` contains an array of platform IDs (strings), not full platform objects. Use `GET /v1/forests/{forestId}` for detailed platform info.

---

## GET /v1/forests/{forestId}

Get detailed forest data including full platform information.

**Response** (returns the forest object directly, not wrapped in an array):

```json
{
  "id": "608a814677b4f80db24913a2",
  "name": "My Business",
  "type": "single",
  "category": "business",
  "slug": "my-business",
  "slugId": "123456",
  "active": true,
  "score": "4.8",
  "reviewAmount": 120,
  "reviewAmountOnPlatform": 3193,
  "totalTreeAmount": 142,
  "reviewTreeAmount": 120,
  "additionalTreeAmount": 22,
  "plantingReviewLimit": -1,
  "isPlantingTrees": false,
  "treeNumbers": {
    "thisPeriod": 5,
    "thisWeek": 3,
    "thisMonth": 12,
    "thisYear": 85,
    "lastPeriod": 0,
    "lastWeek": 4,
    "lastMonth": 15,
    "lastYear": 57
  },
  "displaySettings": {
    "allowDownloadingTreeCertificate": false,
    "showTreeAges": true,
    "showReviewerInitialOnly": false,
    "showReviewTextAndScore": true,
    "useFormalAddressing": false,
    "logo": "https://..."
  },
  "plantingProjects": [
    { "projectId": 2, "trees": 128 },
    { "projectId": 1, "trees": 14 }
  ],
  "platforms": [
    {
      "id": "platformId1",
      "type": "google",
      "typeDisplayName": "Google",
      "name": "My Business on Google",
      "active": true,
      "score": "4.9",
      "reviewAmount": 80,
      "reviewAmountOnPlatform": 3181,
      "reviewTreeAmount": 80,
      "isPlantingTrees": true,
      "keys": {
        "placeId": "ChIJ..."
      },
      "reviewNumbers": {
        "total": 80,
        "thisWeek": 2,
        "thisMonth": 8,
        "thisYear": 55
      },
      "lastCrawlingTimestamp": 1703001600,
      "lastReviewDate": "2025-12-18T09:00:00.000Z",
      "createdAt": "2024-01-15T10:00:00.000Z",
      "updatedAt": "2025-12-20T14:30:00.000Z"
    }
  ],
  "platformsOrder": ["google", "facebook"],
  "pinnedPlatform": "google",
  "projectId": 4,
  "createdAt": "2024-01-15T10:00:00.000Z",
  "updatedAt": "2025-12-20T14:30:00.000Z"
}
```

Key notes:
- `platforms[].name` is the business listing name on that platform, NOT the platform display name. Use `platforms[].typeDisplayName` for the human-readable platform name.
- `score` is a string (e.g. "4.8"), not a number.
- `category`: "business", "employee", "product", "application", or null (for brand forests).
- `type`: "single" (regular forest) or "brand" (brand forest combining multiple forests). **Brand forests have no `platforms`/`platformsOrder`** — instead they return a `channels[]` array, where each channel is itself a forest object (with its own `platforms[]`). Guard for `platforms` being absent before iterating it.
- `reviewAmount` — reviews imported into ReviewForest; `reviewAmountOnPlatform` — total reviews on the source platform(s). Both exist at forest level and per platform.
- `plantingReviewLimit`: -1 means unlimited.
- `displaySettings.showReviewerInitialOnly` and `displaySettings.showReviewTextAndScore` are the customer's choices for how reviews appear. The embed widgets show initials instead of the full name when the first is true, and hide review text and stars when the second is false. The API returns full names and texts either way, so custom rendering has to apply both settings itself.
- `displaySettings.logo` — URL of the customer's logo for their forest page (the API already resolves it to the page-logo version), or `null`. Use the string as-is.
- `platforms[].keys` — platform-specific identifiers. Different per platform type: `placeId` (Google), `companyWebsite` (Trustpilot), `fbPageId` (Facebook), etc.

---

## GET /v1/forests/{forestId}/reviews

List reviews for a forest.

**Query Parameters:**

| Parameter | Type | Default | Values |
|-----------|------|---------|--------|
| `sortBy` | string | `date` | `date`, `name` |
| `order` | string | `desc` | `asc`, `desc` |
| `pageSize` | number | `10` | `10`, `15`, `20`, `25`, `50`, `100` |
| `page` | number | `1` | any positive integer |
| `showReviewOnlyWithText` | boolean | `false` | `true`, `false` — filter to only return reviews that have text |

**Response:**

```json
{
  "query": { "sortBy": "date", "order": "desc", "pageSize": 10, "page": 1, "showReviewOnlyWithText": false },
  "count": 95,
  "data": [
    {
      "name": "John D.",
      "score": 5,
      "text": "Excellent service and great to see trees being planted!",
      "date": "2025-12-15T10:30:00.000Z",
      "platformType": "google",
      "url": "https://reviewforest.org/my-business?review=6756c0f9b9052872cc902dcc"
    }
  ]
}
```

Each review has these fields and no others: `{ name, score, text, date, platformType, url }`; `name`, `score` and `text` can be missing; `url` links to the review on the ReviewForest forest page. This endpoint returns the plain `text` only — it does **not** return `title` or the structured `texts[]`/`ratings[]` arrays. For employer and software platforms (Kununu, Glassdoor, G2) that plain `text` is a flattened or shortened version, or empty: Kununu reviews join their topics into one string with the topic keys in parentheses, Glassdoor reviews carry only the summary. The topic-by-topic content comes with review-type **trees** and with `GET /v1/widgets/{uuid}/reviews` — see [Structured review text](#structured-review-text) under the trees endpoint. `text` is missing or `null` when the review has no plain text, for example a rating without written text, or an OMR review (OMR reviews have only structured text). `showReviewOnlyWithText=true` counts structured text as text, so such reviews can still appear in the filtered list without `text`; skip them client-side too.

When present, `score` is a number from 1 to 5 and can have a decimal (OMR reviews, for example, come in halves like 4.5). It can be missing for a few imported reviews whose rating could not be read (Amazon, for example), so render stars only when `Number.isFinite(score)`.

This is the complete review list of the forest. Use it, not the trees feed, when the user wants all reviews. For a list on a public page, use `GET /v1/widgets/{uuid}/reviews` instead (see [Keyless widget routes](#keyless-widget-routes)): it applies the widget's filters and leaves out reviews whose reviewer asked to be anonymized. That holds for pages rendered on the server or at build time too; keep `/reviews` for views behind a login.

---

## GET /v1/forests/{forestId}/trees

List a forest's tree records. The list can include trees that are not settled yet, and with a Website-Widgets key the response doesn't show that status (`unPaid` is stripped), so take planted totals from the forest's aggregate counts and don't label a single row as a confirmed planting.

**Query Parameters:**

| Parameter | Type | Default | Values |
|-----------|------|---------|--------|
| `sortBy` | string | `date` | `date`, `name` |
| `order` | string | `desc` | `asc`, `desc` |
| `pageSize` | number | `10` | `10`, `15`, `20`, `25`, `50`, `100` |
| `page` | number | `1` | any positive integer |

**Response:**

```json
{
  "query": { "sortBy": "date", "order": "desc", "pageSize": 10, "page": 1 },
  "count": 129,
  "data": [
    {
      "id": "6757947854f359ac56bcc52e",
      "forestId": "5e6f68a238fda2b2bff9692a",
      "name": "Tenna Mouritzen",
      "date": "2024-12-09T12:05:45.000Z",
      "score": 5,
      "title": "5 stars from me",
      "text": "Great service!",
      "type": "review",
      "platformType": "trustpilot",
      "url": "https://www.trustpilot.com/reviews/...",
      "pageTreeUrl": "https://reviewforest.org/demo-page?review=...",
      "plantingProject": "Eden: People+Planet",
      "plantingProjectId": 2,
      "uniqueId": "6756c0f9b9052872cc902dcc"
    },
    {
      "id": "66794df0c140d280933f062e",
      "forestId": "5e6f68a238fda2b2bff9692a",
      "name": "Noel Smith",
      "date": "2024-06-24T10:44:01.000Z",
      "type": "additionalTree",
      "occasion": "Business Celebration",
      "pageTreeUrl": "https://reviewforest.org/demo-page?review=...",
      "plantingProject": "Eden: People+Planet",
      "plantingProjectId": 2,
      "uniqueId": "b9914fbf-9fd0-4f71-97f5-c1794d444d38"
    }
  ]
}
```

Key notes:
- Tree `type` is either `"review"` (a tree for a review) or `"additionalTree"` (added manually).
- The feed is not a complete review list: it holds only the reviews that planted a tree (a forest can pause or cap planting), plus the manually planted trees. For all reviews use `GET /v1/forests/{forestId}/reviews`.
- Review trees have `score`, `text`, `title`, `platformType`, and `url` (link to the original review on the source platform). They may also carry `texts[]`/`ratings[]` for structured reviews — see [Structured review text](#structured-review-text) below.
- On a public page, show trees only as counts or with their date and planting project, and take reviews from `GET /v1/widgets/{uuid}/reviews`. A review tree's reviewer details (name, photo, text, review link, `pageTreeUrl`) belong behind a login: the feed can still hold a copy of a review whose reviewer asked to be anonymized, and its rows can't be matched to widget reviews reliably.
- Review trees can also have `profilePicture`, the reviewer's photo URL, when the platform provides one (Google, Google Play and Facebook reviews). Today it points at Google's or Facebook's image servers (`*.googleusercontent.com`, `*.ggpht.com`, `*.fbcdn.net`, `*.fbsbx.com`). After an upcoming ReviewForest release it points at `https://api.reviewforest.org` instead (photos already hosted on `reviewforest.org` or its subdomains keep their address), and photos from any other host are left out. If the page has a Content Security Policy that restricts `img-src`, allow `https://api.reviewforest.org` there now, plus those platform hosts for as long as photos come from them, and any `reviewforest.org` image origin that shows up in the feed.
- Additional trees have `occasion` instead of review data.
- `pageTreeUrl` — link to the tree on the ReviewForest forest page.
- `url` — direct link to the original review on the source platform (only on review trees).
- With a Website-Widgets key, the billing fields `invoiceNumber` and `unPaid` are stripped from the response.

### Structured review text

Review-type trees carry the review text in one of two forms (`GET /v1/widgets/{uuid}/reviews` returns the same fields for every review a widget shows; see the [interactive documentation](https://reviewforest.org/apidoc/)):

- **Simple** — the `text` field holds the full text as a string (with an optional `title`).
- **Structured** — `texts[]` and/or `ratings[]` arrays break the review down by topic. Each item usually has `id` (topic key) and `text` (may contain basic HTML — sanitize before rendering); some Kununu items are plain strings, so render a string item as is, skip empty items, and fall back to the plain `text` when no topic content is left. Common on employer/product platforms (Kununu, Glassdoor, G2).

```json
{
  "type": "review",
  "name": "Jane S.",
  "score": 4,
  "title": "Mostly positive",
  "texts": [
    { "id": "pros", "text": "Great team and culture" },
    { "id": "cons", "text": "Could improve communication" }
  ],
  "ratings": [
    { "id": "workLife", "text": "Good balance overall" },
    { "id": "salary", "text": "Competitive" }
  ],
  "platformType": "kununu"
}
```

A tree may have `texts` only, `ratings` only, or both — combine them when rendering. If neither array has entries, fall back to the plain `text` field.

#### Review topic keys

Display labels for the topic `id` values. For any `id` not listed, humanize the key itself as a fallback (the list can grow):

| Key | English Label |
|-----|--------------|
| `advice` | Advice |
| `atmosphere` | Work atmosphere |
| `career` | Career/further training |
| `communication` | Communication |
| `cons` | Cons |
| `environment` | Environmental/social awareness |
| `equality` | Equal rights |
| `image` | Image |
| `leadership` | Superiors' behavior |
| `negative` | What I find bad about the employer |
| `oldColleagues` | Interaction with older colleagues |
| `positive` | What I find good about the employer |
| `problems` | What problems does this product solve for you? |
| `pros` | Pros |
| `salary` | Salary/benefits |
| `suggestion` | Suggestions for improvement |
| `tasks` | Interesting tasks |
| `teamwork` | Team spirit |
| `whatDoYouDislike` | What do you dislike? |
| `whatDoYouLikeTheBest` | What do you like best? |
| `whatProblemsWereSolved` | What problems were solved? |
| `workConditions` | Work conditions |
| `workLife` | Work-life balance |

---

## Keyless widget routes

Both answer without an API key, because the embed widgets call them from every visitor's browser. Use them for anything shown on a public page.

`GET /v1/widgets?uuids={uuid}` (several UUIDs comma-separated) returns `{ count, data: [widget] }`. Each widget carries its settings, its `forestId` and, under `data`, the forest summary: `score`, `reviewAmountOnPlatform` (the review count the badges show), `reviewAmount` (reviews imported into ReviewForest), `totalTreeAmount`, `slug`, `displaySettings`, and for single forests `platforms[]` (`type`, `name`, `keys`; missing when the forest has none). An unknown or deleted UUID answers `200` with `{ count: 0, data: [] }`, so check that the widget is there. Each call counts as an impression of the widget, so don't poll it. For a brand forest the summary adds up only its active channels; after a campaign ends, `GET /v1/widgets/{slug}/counter` (also without a key) returns the stored final `totalTreeAmount`.

`GET /v1/widgets/{uuid}/reviews` returns the reviews that widget shows, paged (the reviews are in `data`). It applies the widget's dashboard filters and leaves out reviews whose reviewer asked to be anonymized. Optional query parameters: `page`, `pageSize` (1-100), `sortBy` (`date`, `name`, `score`), `order` (`asc`, `desc`), `showReviewOnlyWithText`, `keywordsInclude`, `keywordsExclude`, `minScore`; a parameter you pass replaces the widget's own setting. Each review carries `id`, `uniqueId`, `name`, `score`, `date`, `text`, `texts`, `title`, `ratings`, `platformType`, `platform`, `profilePicture`, `url`, `language` and `isVerified`, each only when it is set. This route is not counted as an impression.

## Forest Page URL

Each forest has a public page at: `https://reviewforest.org/{slug}`
