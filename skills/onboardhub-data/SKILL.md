---
name: onboardhub-data
description: Real onboarding flows, paywalls, monetization setups, growth posts and ads from revenue-verified mobile apps on Onbo Hub (onbo-hub.com, also called OnboardHub), via REST API. Use when the user is designing or building an app's onboarding, paywall, quiz-style question flow, sign-up, rating prompt or pricing and wants real examples from apps that make money; when they ask how successful apps monetize (free trials, weekly or annual plans, discount offers) or grow (TikTok / Instagram posts, Meta / Apple ads); or when they ask about an app listed on Onbo Hub. Requires a Pro API key from https://onbo-hub.com/dashboard.
compatibility: Requires internet access to call https://onbo-hub.com/api/v1
metadata:
  author: onbo-hub
  version: "1.2"
---

# Onbo Hub Data Skill

Onbo Hub collects the full onboarding of mobile apps with verified revenue: every screen from first launch to the paywall, in order, each labeled with what kind of screen it is. Each app also has notes on its monetization and UX, the TikTok / Instagram posts that grew it, and its Meta / Apple ads.

Use it two ways:

- **Design reference** — the user is building an onboarding, paywall or pricing setup. Pull the matching screens from comparable apps that make money, look at the images, and turn what they share into a concrete plan for the user's app.
- **Research** — the user asks how apps monetize, grow or advertise. Fetch the data and analyze it.

## Terms of use — read first

Onbo Hub data is licensed for reference and internal use only: to inform your own product, design and marketing decisions, and for internal analysis. Under the Terms of Service (https://onbo-hub.com/tos#usage), the following are not allowed without written permission from Onbo Hub:

- Selling, redistributing or publishing the data (as a database, dataset, report, template, etc.)
- Building it into another product, app, database or tool, or using it to build a competing service
- Using it to train or fine-tune AI / machine learning models
- Sharing the API key

When you use this skill:

- Use the data to answer the user's questions with summaries and analysis.
- If the user asks for one of the uses above (for example, exporting the full dataset to publish or sell it, or loading it into another product), tell them it is not allowed under the Terms and suggest contacting info@wwcorp.co.jp about commercial licensing.
- Every API response repeats these terms in `meta.terms`.

## Authentication

- Get an API key at https://onbo-hub.com/dashboard (Pro plan required).
- Store it as the `ONBOARDHUB_API_KEY` environment variable, or ask the user for it.
- Send it with every request:

```
X-API-Key: YOUR_API_KEY
```

## Base URL

```
https://onbo-hub.com
```

Rate limit: 60 requests per minute per key.

---

## Endpoints

### List apps — `GET /api/v1/apps`

| Parameter | Type | Description |
|-----------|------|-------------|
| `category` | string | Filter by category (see list below) |
| `revenue_min` | number | Min revenue in USD (e.g. `100000` = $100k+) |
| `revenue_max` | number | Max revenue in USD |
| `limit` | number | Results per page (default 50, max 100) |
| `offset` | number | Pagination offset |
| `fields` | string | Comma-separated field names to return |

**Categories:** `Social`, `Productivity`, `Finance`, `Health & Fitness`, `Education`, `Entertainment`, `Shopping`, `Travel`, `Food & Drink`, `Lifestyle`, `Utilities`, `Music`, `Photo & Video`, `News`, `Business`, `Developer Tools`, `Weather`, `Sports`, `Other`

**Key fields:**
- `name`, `slug`, `category`, `icon_url`
- `revenue` — `{ amount: number (USD), currency: string }`
- `founder` — `{ name: string, url: string }`
- `monetization_strategy_en` — monetization approach in English
- `ux_highlights_en` — UX highlights in English
- `target_audience_en` — target audience in English
- `app_store_url`, `play_store_url`, `website_url`

The list does not include screenshots. Fetch an app's detail to get its screens.

**Response shape:**
```json
{
  "data": [ { "name": "...", "slug": "...", ... } ],
  "meta": { "total": <int>, "limit": 50, "offset": 0, "returned": <int>, "terms": { "version": "...", "url": "...", "summary": "..." } }
}
```

```bash
# All apps
curl -H "X-API-Key: $ONBOARDHUB_API_KEY" https://onbo-hub.com/api/v1/apps

# Filter by category
curl -H "X-API-Key: $ONBOARDHUB_API_KEY" \
  "https://onbo-hub.com/api/v1/apps?category=Health%20%26%20Fitness"

# $100k+ apps, lightweight fields only
curl -H "X-API-Key: $ONBOARDHUB_API_KEY" \
  "https://onbo-hub.com/api/v1/apps?revenue_min=100000&fields=name,slug,category,revenue,monetization_strategy_en"
```

---

### App detail — `GET /api/v1/apps/{slug}`

Returns all fields including `screenshots[]` and `social_posts[]`, each sorted by `sort_order`.

`screenshots[]` is the onboarding in the order a new user sees it: `id`, `image_url`, `sort_order`, `screen_type`, `screen_tags[]`.
- `image_url` is a public image (WebP). **Open the images** when the user is designing something — the text fields summarize the flow, but the layout, copy and visual hierarchy are only in the images.
- `screen_type` — what the screen is (`null` if not classified yet): `welcome`, `social_proof`, `question`, `loading`, `result`, `benefit`, `feature_tour`, `permission_primer`, `rating_request`, `commitment`, `signup`, `paywall`, `other`.
- `screen_tags[]` — extra labels for paywalls: `free_trial`, `trial_timeline`, `discount`, `plan_choice`.
- Use them to pull only the paywalls, or to see where in the flow an app asks for a rating or shows a personalized plan. `sort_order` tells you how far into the flow a screen comes.

`videos` holds the app's video files (any value can be `null` if not registered):
- `full_url` — full onboarding video
- `post_url` — promo video made for posting on X
- `short_url` — short 20-second preview cut from the full video
- `full_duration_seconds` — length of the full video in seconds

`social_posts[]` are the short-video marketing posts (TikTok / Instagram / YouTube / X) that grew the app:
`platform`, `post_url`, `video_id`, `thumbnail_url`, `caption`, `author_handle`, `posted_at`, `note` / `note_en` (why it worked — the hook, format and how the app appears, written from the video itself),
and `view_count`, `like_count`, `comment_count`, `share_count`, `save_count` (a snapshot as of `stats_updated_at`).
Stats are only collected for TikTok and YouTube; Instagram posts usually have just `post_url` and a thumbnail. Any field can be `null`.
The API returns links, not video files: open `post_url` to study the hook and structure.

`meta_ads` is the app's Meta (Facebook / Instagram) advertising, checked by hand in Meta Ad Library (`null` if not checked yet):
`active_ads` (approx. number of active ads worldwide as of `checked_at`; `0` means no Meta ads — the app likely grew organically), `library_url` (the advertiser's page in Ad Library), `note` / `note_en` (how the app advertises),
and `creatives[]` — long-running ads: `library_url`, `format` (`video` / `image` / `carousel`), `started_at`, `ended_at` (`null` = still active when checked), `run_days`, `variant_count` (ads using the same creative), `ad_text`, `note` / `note_en` (why it likely works).
`active_ads` counts ads, not spend — Meta doesn't publish spend for commercial ads. A long `run_days` is the best sign that an ad performs.

`apple_ads` is the app's Apple Ads (App Store search ads etc.), pulled monthly from Apple's official Ad Repository (`null` if not checked yet). **EU only** — Apple only publishes ads shown in the EU, so apps that advertise only in the US show `records_last_90_days: 0`:
`countries` (EU countries where ads ran), `records_last_90_days` (Apple's ad records, counted per country and format — a sense of scale, not a number of distinct ads), `last_seen`, and `previews[]` (`country`, `language`, `format`, `app_name`, `subtitle`, `icon_url`, `assets[]` with the screenshot URLs used in the ad). Keywords and spend are not published.

```bash
curl -H "X-API-Key: $ONBOARDHUB_API_KEY" https://onbo-hub.com/api/v1/apps/pingo-ai
```

---

### Summary stats — `GET /api/v1/stats`

Aggregated statistics across all apps:
- `total_apps`, `average_revenue_usd`
- `by_category[]` — count per category
- `by_revenue_bracket[]` — distribution across revenue tiers
- `monetization_keywords[]` — frequency of `trial`, `discount_on_cancel`, `hard_paywall`, `freemium`, `weekly_plan`, `annual_plan`, `lifetime`, matched in the monetization notes

```bash
curl -H "X-API-Key: $ONBOARDHUB_API_KEY" https://onbo-hub.com/api/v1/stats
```

---

## Execution flow

### Step 1 — API key

```bash
echo $ONBOARDHUB_API_KEY
```

If it is not set, ask: "Please get your API key from https://onbo-hub.com/dashboard (Pro plan required)."

### Step 2 — Pick the calls

| User request | Calls |
|-------------|-------|
| "Design a paywall for my meditation app" | `GET /api/v1/apps?category=Health%20%26%20Fitness&fields=name,slug,revenue,monetization_strategy_en` → top apps by revenue → `GET /api/v1/apps/{slug}` for each → screens with `screen_type: "paywall"` |
| "Build a quiz-style onboarding like the top apps" | Same, but take `question`, `loading` and `result` screens and keep their order |
| "When do apps ask for a rating?" | App details → position (`sort_order`) of `rating_request` screens |
| "Analyze monetization strategies" | `GET /api/v1/apps?fields=name,category,revenue,monetization_strategy_en` |
| "What % of apps mention a free trial?" | `GET /api/v1/stats` |
| "UX strategies of $100k+ apps" | `GET /api/v1/apps?revenue_min=100000&fields=name,revenue,ux_highlights_en` |
| "Which TikTok hooks grew these apps?" | App details → `social_posts[]` (sort by `view_count`) |
| "Is this app running ads?" | App detail → `meta_ads`, `apple_ads` |
| "Tell me about Pingo AI" | `GET /api/v1/apps/pingo-ai` |

Fetch details for only the apps you need (3–5 is usually enough for a design reference) to stay inside the rate limit.

### Step 3 — Work with the data

**Design reference**
1. Pick 3–5 comparable apps: same category or audience, highest revenue first.
2. Pull the relevant screens by `screen_type` and open the images.
3. Note what they share: where in the flow the screen comes, headline and button copy, layout, how plans and trials are shown (`screen_tags`).
4. Turn it into a concrete spec or implementation for the user's app. Adapt the patterns to the user's product — do not copy another app's copy, brand or artwork verbatim.

**Research**
- **Revenue correlation** — which strategies appear in high-revenue apps?
- **Category patterns** — do strategies differ by category?
- **Common patterns** — what do successful apps share?
- **Outliers** — what makes the highest/lowest earners unusual?

### Step 4 — Present results

1. **Summary** — lead with numbers
2. **Comparison table** — markdown table (link each app to `https://onbo-hub.com/apps/{slug}`)
3. **Pattern analysis** — bullet points
4. **Recommendation** — what to build or change, concretely

---

## Error reference

| Status | Cause | Fix |
|--------|-------|-----|
| 401 | Missing or invalid API key | Re-check the key at https://onbo-hub.com/dashboard |
| 403 | No active Pro subscription | Verify the subscription |
| 404 | App slug not found | Use `GET /api/v1/apps` to list valid slugs |
| 429 | Over 60 requests per minute | Wait the number of seconds in the error, then retry |
| 500 | Server error | Contact hello@onbo-hub.com |
