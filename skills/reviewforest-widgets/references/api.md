# ReviewForest API Reference

Base URL: `https://api.reviewforest.org`

## Public Endpoints (No Auth Required)

### Get Widget by UUID

```
GET /v1/widgets/{uuid}
```

Returns widget configuration and forest data. No authentication needed.

**Response** (simplified):
```json
{
  "count": 1,
  "data": [{
    "uuid": "abc-123",
    "type": "2in1",
    "sub_type": "badge",
    "config": {
      "appearance": "light",
      "behaviour": "floating",
      "position": "bottom-right",
      "logo_of_review_platform": true,
      "number_of_reviews": true,
      "number_of_trees_planted": true,
      "star_rating": true,
      "verified_by_rf": true,
      "hide_on_mobile": false,
      "reviews_scroll_type": "auto",
      "reviews_sort_by": "date",
      "reviews_sort_order": "desc",
      "show_review_only_with_text": false
    },
    "status": 1,
    "name": "My Business",
    "data": {
      "category": "business",
      "totalTreeAmount": 142,
      "reviewTreeAmount": 120,
      "additionalTreeAmount": 22,
      "slug": "my-business",
      "name": "My Business",
      "active": true,
      "score": "4.8",
      "reviewAmount": 120,
      "reviewAmountOnPlatform": 95,
      "displaySettings": { ... },
      "platforms": [
        { "keys": { "placeId": "...", "reviewLink": "https://..." }, "name": "My Business", "type": "google" }
      ],
      "platformsOrder": ["google", "facebook"]
    },
    "subscription": {
      "plantingReviewLimit": 500,
      "status": "active",
      "plan": "premium"
    }
  }]
}
```

### Get Widget Reviews

```
GET /v1/widgets/{uuid}/reviews
```

Returns paginated reviews. No authentication needed.

**Query Parameters:**
| Parameter | Type | Default | Values |
|-----------|------|---------|--------|
| `sortBy` | string | `date` | `date`, `name`, `score` |
| `order` | string | `desc` | `asc`, `desc` |
| `pageSize` | number | `10` | `10`, `15`, `20`, `25`, `50`, `100` |
| `page` | number | `1` | any positive integer |
| `showReviewOnlyWithText` | boolean | `false` | `true`, `false` |

**Response:**
```json
{
  "query": { "pageSize": 10, "sortBy": "date", "order": "desc", "page": 1 },
  "count": 95,
  "data": [
    {
      "uniqueId": "f7cc727c9db25c0803dffb5d",
      "name": "John D.",
      "score": 5,
      "text": "Excellent service and great to see trees being planted!",
      "date": "2025-12-15T10:30:00.000Z",
      "platformType": "google",
      "id": "693f510e54f359ac56746b30"
    }
  ]
}
```

### Get Forest Counter

```
GET /v1/widgets/{slug}/counter
```

Returns tree count and basic forest metadata. No auth. Use the forest slug (from the widget data's `data.slug` field).

**Response:**
```json
{
  "active": true,
  "id": "608a814677b4f80db24913a2",
  "name": "My Business",
  "slug": "my-business",
  "totalTreeAmount": 142,
  "type": "single"
}
```

Note: This endpoint only returns `totalTreeAmount`, not broken down by review/additional trees. For detailed tree counts and review scores, use the widget endpoint instead.

### Track Widget Events

```
POST /v1/widgets/{uuid}/events/{type}
```

Track impressions and clicks. Type: `impression` or `click`. No auth required.

---

## Authenticated Endpoints (API Key Required)

Pass `apikey` header with all requests. Get your API key from https://app.reviewforest.org → Settings → API.

### List All Widgets

```
GET /v1/widgets
Header: apikey: YOUR_API_KEY
```

**Query Parameters:** `sortBy` (createdAt), `order` (asc/desc), `pageSize`, `page`

### Create Widget

```
POST /v1/widgets
Header: apikey: YOUR_API_KEY
Content-Type: application/json
```

**Required fields:**
| Field | Type | Values |
|-------|------|--------|
| `data_id` | string | Forest ID (MongoDB ObjectId) |
| `type` | string | `2in1`, `counter`, `forest`, `mini`, `review-score`, `testimonial-carousel` |

**Optional fields:**
| Field | Type | Values |
|-------|------|--------|
| `sub_type` | string | `badge`, `testimonial`, `banner-review-score`, `tree-counter`, `review-score` |
| `appearance` | string | `light`, `dark` |
| `behaviour` | string | `floating`, `fixed-position` |
| `position` | string | `bottom-left`, `bottom-right` |
| `name` | string | Widget display name |
| `logo_of_review_platform` | boolean | Show platform logos |
| `number_of_reviews` | boolean | Show review count |
| `number_of_trees_planted` | boolean | Show tree count |
| `verified_by_rf` | boolean | Show "Verified by ReviewForest" |
| `star_rating` | boolean | Show star rating |
| `ask_for_reviews` | boolean | Show "Write a review" CTA |
| `ask_for_reviews_type` | number | `1` (to platform), `2` (to ReviewForest) |
| `hide_on_mobile` | boolean | Hide on mobile devices |
| `forest_content` | string | `only-forest`, `whole-page` |
| `text_version` | string | `full`, `short` |
| `font_size` | string | `small`, `medium` |
| `reviews_scroll_type` | string | `auto`, `manual` |
| `reviews_sort_by` | string | `name`, `score`, `date` |
| `reviews_sort_order` | string | `asc`, `desc` |
| `show_review_only_with_text` | boolean | Only show reviews with text |

### Update Widget Config

```
PATCH /v1/widgets/{uuid}
Header: apikey: YOUR_API_KEY
Content-Type: application/json
```

Body: any of the optional config fields from Create.

### List Forests

```
GET /v1/forests
Header: apikey: YOUR_API_KEY
```

**Query Parameters:** `sortBy`, `order`, `pageSize`, `page`

Returns all forests for the authenticated account. Use the forest `id` as `data_id` when creating widgets.

### Get Forest Reviews

```
GET /v1/forests/{id}/reviews
Header: apikey: YOUR_API_KEY
```

### Get Forest Trees

```
GET /v1/forests/{id}/trees
Header: apikey: YOUR_API_KEY
```

### Plant Trees

```
POST /v1/forests/{id}/trees
Header: apikey: YOUR_API_KEY
Content-Type: application/json
```

**Body:**
```json
{
  "quantity": 5,
  "names": ["Alice", "Bob"],
  "occasion": "Team milestone"
}
```

---

## Widget Type Reference

| Type + Sub-type | Widget Name | Description |
|----------------|-------------|-------------|
| `2in1` + `badge` | 2-in-1 Badge | Combined review score + tree counter badge |
| `review-score` + `badge` | Review Score Badge | Shows aggregate rating |
| `review-score` + `banner-review-score` | Review Score Banner | Banner-style review score |
| `counter` + `tree-counter` | Tree Counter Badge | Shows trees planted count |
| `mini` + `review-score` | Mini Review Score | Compact rating display |
| `mini` + `tree-counter` | Mini Tree Counter | Compact tree count |
| `forest` | Forest Widget | Full interactive forest with reviews |
| `testimonial-carousel` + `testimonial` | Testimonial Carousel | Rotating review display |

## Forest Page URL

Each forest has a public page at: `https://reviewforest.org/{slug}`
