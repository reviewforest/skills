# ReviewForest API Reference

Base URL: `https://api.reviewforest.org`

Full interactive documentation: https://reviewforest.org/apidoc/ | Swagger: https://reviewforest.org/apidoc/swagger.json

## Authentication

All endpoints require the `apikey` header:

```
apikey: YOUR_API_KEY
```

Create an API key at https://app.reviewforest.org/integrations/public-api. Use **read-only** mode for client-side JavaScript.

## Pagination

All list endpoints support pagination via query parameters:

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

`count` is the total number of items. Use `count` and `pageSize` to calculate total pages.

---

## GET /v1/forests

List all forests for the authenticated account.

**Query Parameters:**

| Parameter | Type | Default | Values |
|-----------|------|---------|--------|
| `sortBy` | string | `createdAt` | `additionalTreeAmount`, `createdAt`, `name`, `score`, `reviewAmount`, `reviewTreeAmount`, `totalTreeAmount` |
| `order` | string | `desc` | `asc`, `desc` |
| `pageSize` | number | `10` | `10`, `15`, `20`, `25`, `50`, `100` |
| `page` | number | `1` | any positive integer |
| `platformTypes` | string | `all` | comma-separated platform types |

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
- `type`: "single" (regular forest) or "brand" (brand forest combining multiple forests).
- `reviewAmount` — reviews imported into ReviewForest; `reviewAmountOnPlatform` — total reviews on the source platform(s). Both exist at forest level and per platform.
- `plantingReviewLimit`: -1 means unlimited.
- `displaySettings.logo` — the customer's logo uploaded for their forest page. Can be a string URL or an object with `variants` (keys: `emailLogo`, `public`, `pageLogo`). Use `variants.pageLogo` for display.
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
      "title": "Great experience",
      "text": "Excellent service and great to see trees being planted!",
      "date": "2025-12-15T10:30:00.000Z",
      "platformType": "google"
    }
  ]
}
```

### Review Text Formats

Reviews can have two text formats:

**Simple review** — has `text` (string or null) and optionally `title` (string or null):

```json
{
  "name": "John D.",
  "score": 5,
  "title": "Great experience",
  "text": "Excellent service!",
  "platformType": "google"
}
```

**Structured review** — has `texts[]` and/or `ratings[]` arrays instead of (or in addition to) `text`. Each item has `id` (topic key) and `text` (may contain basic HTML — sanitize before rendering):

```json
{
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

A review may have `texts` only, `ratings` only, or both. Combine them when rendering. If neither array is present, fall back to the plain `text` field.

### Review Topic Keys

Use these display labels for the topic `id` values:

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

## GET /v1/forests/{forestId}/trees

List planted trees for a forest.

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
      "invoiceNumber": "91F1C875-0047",
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
      "invoiceNumber": "91F1C875-0038",
      "uniqueId": "b9914fbf-9fd0-4f71-97f5-c1794d444d38"
    }
  ]
}
```

Key notes:
- Tree `type` is either `"review"` (planted from a review) or `"additionalTree"` (manually planted).
- Review trees have `score`, `text`, `title`, `platformType`, and `url` (link to original review on the source platform). May also have `texts[]`/`ratings[]` for structured reviews (same format as in the reviews endpoint).
- Additional trees have `occasion` instead of review data.
- `pageTreeUrl` — link to the tree on the ReviewForest forest page.
- `url` — direct link to the original review on the source platform (only on review trees).

---

## POST /v1/forests/{forestId}/trees

Plant additional trees.

**Request Body:**

```json
{
  "quantity": 5,
  "names": ["Alice", "Bob"],
  "occasion": "Team milestone"
}
```

All fields are optional. `quantity` defaults to 1.

**Response:** Array of planted tree objects.

---

## Forest Page URL

Each forest has a public page at: `https://reviewforest.org/{slug}`
