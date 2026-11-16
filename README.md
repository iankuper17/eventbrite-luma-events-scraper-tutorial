# Eventbrite & Luma Events Scraper tutorial

Tutorial for the **[Eventbrite & Luma Events Scraper](https://apify.com/hiringdata/eventbrite-luma-events-scraper)** on [Apify Store](https://apify.com/hiringdata/eventbrite-luma-events-scraper) (`hiringdata/eventbrite-luma-events-scraper`).

Search **public events from Eventbrite and Luma (lu.ma)** in one run. Filter by **city**, **keyword**, **category**, and **date range**, or paste **event, organizer, calendar, and search URLs**. Every event comes back in **one normalized schema**: start and end time with **timezone** (local and UTC), venue, full address, **latitude/longitude**, online flag, **organizer**, **ticket price range and currency**, sold-out status, image, and URL.

No login and no browser are required. Unknown values are returned as `null`, never guessed. You pay only for events saved to your dataset. Duplicates, filtered-out, and already-seen events are not charged.

This repository is documentation only. It does **not** include the actor's source code. To run it, use the [Apify Store page](https://apify.com/hiringdata/eventbrite-luma-events-scraper) or the API examples below.

## What it does

| | This actor |
|---|---|
| Platforms | Eventbrite + Luma in one run, one schema |
| Input | City + keyword + category + date range, and/or Eventbrite event / organizer / search URLs and Luma event / calendar / city URLs |
| Times | `startLocal` / `endLocal` with UTC offset, `startUtc` / `endUtc`, IANA `timezone` |
| Location | Venue, address, city, region, postal code, country, coordinates, plus `isOnline` |
| Organizer & tickets | Organizer name/URL, free flag, min/max price, currency, sold-out, spots remaining (Luma) |
| Honest data | Fields the site doesn't publish stay `null`; hidden Luma addresses are **not** replaced by approximate points |
| Monitoring | "Only new events since the last run" for scheduled alerts |
| Charging | Duplicates, filtered-out and already-seen events are **never charged** |
| Export | Download from Apify Console as JSON, **CSV**, or Excel, or pull the dataset through the API |
| Politeness | Honors `robots.txt` (wildcards included) and rate-limits each site |

The actor only reads public event pages and the public JSON these websites load in the browser. You are responsible for how you use the data and for complying with the platforms' terms.

## Use cases

- **Event lead generation** — find organizers and venues that run events in your niche. The dataset includes the organizer name and public profile URL when the platform publishes them.
- **Tech event monitoring** — watch AI, startup, and developer events in a city on both Eventbrite and Luma, or schedule a daily run that returns only newly listed events.
- **Market research** — track how many events, what prices, and which formats appear in a city or category.
- **City guides, calendars, and newsletters** — feed a weekly "what's on" digest, a map, or an app with structured event data.
- **Sponsorship and community teams** — monitor new meetups, hackathons, and conferences on Luma and Eventbrite.

## Pricing

Pay-per-event pricing. You are charged for each **event result** saved to your dataset, plus Apify's standard actor-start fee.

| Event | Price |
|---|---|
| **Event result** (Free plan) | **$0.002** each ($2.00 per 1,000 events) |
| Bronze | $0.0017 each ($1.70 per 1,000 events) |
| Silver | $0.0014 each ($1.40 per 1,000 events) |
| Gold, Platinum, and Diamond | $0.0012 each ($1.20 per 1,000 events) |
| Actor start | $0.00005 per run (Apify's standard start fee) |

Duplicates (the same event found by two searches), events outside your date range or filters, and events skipped in "only new events" mode are **not charged**. Set **Max events in total** (or a maximum cost per run in Apify Console) and the actor stops at your budget.

Example from the actor README: 1,000 events cost **$2.00** on the Free plan.

## How to scrape Eventbrite events (and Luma) in Apify Console

1. Open the actor on Apify Store: [apify.com/hiringdata/eventbrite-luma-events-scraper](https://apify.com/hiringdata/eventbrite-luma-events-scraper).
2. Click **Try for free** (or **Start** if you already have it). This opens the actor in [Apify Console](https://console.apify.com/actors/hiringdata~eventbrite-luma-events-scraper).
3. Add one or more **locations** (for example `San Francisco`, `London`, `New York`, `Berlin`, or `online` for Eventbrite online events). For ambiguous names add the region: `Portland, OR`.
4. Optionally add **keywords** (`AI`, `yoga`, `jazz`, `startup`…). Each keyword is a separate search on each platform.
5. Optionally pick a **date range** (`2026-12-01`, `today`, `+30 days`), an **Eventbrite category**, and/or a **Luma category**.
6. Optionally paste **direct URLs**:
   - Eventbrite event: `https://www.eventbrite.com/e/...-tickets-1234567890`
   - Eventbrite organizer (all live events): `https://www.eventbrite.com/o/organizer-name-1234567890`
   - Eventbrite search: `https://www.eventbrite.com/d/ca--san-francisco/music--events/`
   - Luma event: `https://lu.ma/abc123`, Luma calendar: `https://lu.ma/your-calendar`, Luma city: `https://lu.ma/sf`
7. Click **Start**. When the run finishes, **export Eventbrite events to CSV** (or JSON or Excel) from the dataset tab, or send them to a webhook, Zapier, Make, or n8n.

**Eventbrite details.** Eventbrite search pages list title, times, venue, coordinates, and categories, but not the organizer or prices. With **Fetch organizer & ticket prices** on (the default), the actor reads each event's public event data to add organizer, price range, currency, sold-out flag, status, and exact UTC times. That is one extra request per Eventbrite event, so it is slower than list-only scraping.

**Luma cities.** Luma's own discover cities (about 87, for example San Francisco, New York, London, Berlin, Medellín, Singapore) are matched by name. For other cities the actor looks up the city's coordinates on Eventbrite and runs a Luma "nearby" search around them; the run summary shows which method was used. Luma's search can mix in far-away events when few local ones match, so Luma location searches keep only events within **100 km** of the city centre (events dropped this way are counted as `outsideSearchArea` in `RUN_SUMMARY` and are not charged).

**Tip:** to get alerts for new events, enable **Only new events since the last run**, set a monitor key, and schedule the actor daily. You only receive (and pay for) events you have not seen before.

If a search cannot be completed, it is listed with a clear reason in the log and in `RUN_SUMMARY` in the run's key-value store; other searches continue. Enable **Fail the run if any search fails** for strict pipelines.

## Input fields

Provide at least one location or one URL in `startUrls`. Defaults below match the actor's input schema.

| Field | Type | Default | Description |
|---|---|---|---|
| `locations` | array of strings | — | One city per line, for example `San Francisco`, `London`, `New York`, `Berlin`. For ambiguous names add the region: `Portland, OR`. Eventbrite `region--city` slugs such as `ca--san-francisco` are also accepted. Use `online` for Eventbrite online events. Each location is searched on every selected platform. |
| `keywords` | array of strings | — | Optional. Each keyword runs its own search on each platform (for example `AI`, `yoga`, `jazz`, `startup`). Leave empty to get all events in the location. |
| `platforms` | array of `eventbrite`, `luma` | `["eventbrite", "luma"]` | Where to search for the locations above. Start URLs always use the platform of the URL. |
| `startDate` | string | — | Only events happening on or after this date (the event's local date). `YYYY-MM-DD`, `today`, `tomorrow`, or `+7 days`. Multi-day events that overlap the range are kept. |
| `endDate` | string | — | Only events happening on or before this date. `YYYY-MM-DD`, `today`, `tomorrow`, or `+30 days`. |
| `eventbriteCategory` | string | `"none"` | Optional Eventbrite category for location searches (for example `business`, `science-and-tech`, `music`, `arts`, `food-and-drink`). |
| `lumaCategory` | string | `"none"` | Optional Luma discover category for location searches (for example `ai`, `tech`, `arts`, `climate`, `crypto`). |
| `startUrls` | array of strings | — | Optional Eventbrite event (`/e/`), organizer (`/o/`), or search (`/d/`) URLs, and Luma event, calendar, or city URLs. |
| `fetchDetails` | boolean | `true` | Eventbrite: load each event's public data for organizer, price range, currency, sold-out flag, status, and exact UTC times. One extra request per Eventbrite event. Luma lists already include these fields. |
| `includeDescription` | boolean | `false` | Adds the full event description (plain text). One extra request per event. Off by default; the short summary is included when the platform provides it. |
| `freeOnly` | boolean | `false` | Keep only events the platform marks as free. Events with unknown price are dropped. |
| `strictKeywordMatch` | boolean | `false` | Platform search is fuzzy. Enable to keep only events whose title, summary, or description contains every word of the keyword. |
| `excludeKeywords` | array of strings | — | Drop events whose title contains any of these words. |
| `maxEventsPerSearch` | integer | `100` | Cap per location × keyword × platform search (and per URL). `0` = as many as the platform returns (Eventbrite search stops at 49 pages × 20 events). |
| `maxEventsTotal` | integer | `0` | Hard cap on results (and cost) for the whole run. `0` = unlimited. |
| `onlyNewEvents` | boolean | `false` | Skip events already delivered in previous runs with the same monitor key. Skipped events are not charged. |
| `monitorKey` | string | `"default"` | Name of the monitoring list. Use different keys for different searches. |
| `maxConcurrency` | integer (1–10) | `3` | How many searches or URLs run in parallel. Requests to each site are additionally rate-limited. |
| `failOnPartialErrors` | boolean | `false` | Fail the run if any search fails. By default the run succeeds if at least one search worked; failures are listed in `RUN_SUMMARY`. |
| `proxyConfiguration` | object | `{ "useApifyProxy": false }` | Not needed in the publisher's tests: both sites served public data without a proxy. Enable only if you get blocked. |

### Input example

```json
{
  "locations": ["San Francisco", "London"],
  "keywords": ["AI"],
  "platforms": ["eventbrite", "luma"],
  "startDate": "today",
  "endDate": "+30 days",
  "maxEventsPerSearch": 100,
  "startUrls": [
    "https://www.eventbrite.com/o/mellow-sessions-33211338325",
    "https://lu.ma/sf"
  ]
}
```

Category search for a date range:

```json
{
  "locations": ["New York"],
  "eventbriteCategory": "music",
  "lumaCategory": "arts",
  "startDate": "2026-12-01",
  "endDate": "2026-12-31",
  "freeOnly": true
}
```

## Output fields

| Field | Description |
|---|---|
| `platform`, `id`, `url` | Source (`eventbrite` / `luma`), the platform's event ID, public event URL |
| `title`, `summary`, `description` | Name, short summary (Eventbrite), full plain-text description (only with **Include full description**) |
| `startLocal`, `endLocal`, `timezone` | Local time with UTC offset and the IANA timezone |
| `startUtc`, `endUtc` | Same instants in UTC |
| `isOnline`, `locationType` | Online flag and the platform's location type |
| `venueName`, `address`, `street`, `city`, `region`, `postalCode`, `country` | Venue and address as published |
| `latitude`, `longitude` | Coordinates when the venue is public |
| `addressVisibility` | `public`, or Luma's `guests-only` when the host hides the address (venue/coordinates are then `null`) |
| `organizerName`, `organizerUrl`, `organizerId`, `hostNames` | Eventbrite organizer, or Luma calendar / host |
| `isFree`, `priceMin`, `priceMax`, `currency` | Ticket price range as published by the platform |
| `isSoldOut`, `spotsRemaining`, `registrationStatus` | Availability info when available |
| `categories`, `tags` | Eventbrite categories/formats and organizer tags; Luma categories (with description) and calendar tags |
| `imageUrl`, `guestCount`, `status`, `publishedAt` | Cover image, Luma guest count, Eventbrite status (`live`, `started`, `canceled`…) and publish time |
| `searchInput`, `scrapedAt` | Which search/URL produced the row, and when |

A run summary (per search/URL status, events found, out of date range, duplicates, saved, resolved location) is saved as `RUN_SUMMARY` in the run's key-value store.

### Sample output (Eventbrite)

The record below is a real Eventbrite event from dataset `jkyztuuMJWdwP05x7` (a live run on 14 November 2026). Title, times, prices, and availability can change when the organizer updates the listing. `description` is present when `includeDescription` is enabled.

```json
{
  "platform": "eventbrite",
  "id": "2004063370489",
  "url": "https://www.eventbrite.com/e/women-in-ai-tech-digital-innovation-networking-san-francisco-tickets-2004063370489",
  "title": "Women in AI, Tech & Digital Innovation Networking San Francisco",
  "summary": "Connect with inspiring women in artificial intelligence, technology, startups and digital innovation!",
  "description": null,
  "startUtc": "2026-11-17T03:00:00.000Z",
  "endUtc": "2026-11-17T06:00:00.000Z",
  "startLocal": "2026-11-16T19:00:00-08:00",
  "endLocal": "2026-11-16T22:00:00-08:00",
  "timezone": "America/Los_Angeles",
  "isOnline": false,
  "locationType": "offline",
  "venueName": "Last Drop Tavern",
  "address": "550 Powell Street, San Francisco, CA 94108",
  "street": "550 Powell Street",
  "city": "San Francisco",
  "region": "CA",
  "postalCode": "94108",
  "country": "US",
  "latitude": 37.7899597,
  "longitude": -122.4085332,
  "addressVisibility": "public",
  "organizerName": "Business Minds Events",
  "organizerUrl": "https://www.eventbrite.com/o/business-minds-events-82796727253",
  "organizerId": "82796727253",
  "hostNames": [],
  "isFree": false,
  "priceMin": 9.85,
  "priceMax": 33.85,
  "currency": "USD",
  "isSoldOut": false,
  "spotsRemaining": null,
  "registrationStatus": "tickets_available",
  "categories": [
    "Startups & Small Business",
    "Business & Professional",
    "Meeting or Networking Event"
  ],
  "tags": [
    "Networking",
    "Women",
    "Startup",
    "Sanfrancisco"
  ],
  "imageUrl": "https://img.evbuc.com/https%3A%2F%2Fcdn.evbuc.com%2Fimages%2F1196846127%2F2108009601763%2F1%2Foriginal.20261029-120044?auto=format%2Ccompress&q=75&sharp=10&s=b60b024aba40e9baaf98d326dd8c11b4",
  "guestCount": null,
  "status": "live",
  "publishedAt": "2026-10-29T12:01:46Z",
  "searchInput": "eventbrite | San Francisco | AI",
  "scrapedAt": "2026-11-14T21:48:04.465Z"
}
```

### Sample output (Luma)

Same live run and dataset as above. Luma lists do not include a short `summary`; enable **Include full description** to get the full text. `spotsRemaining` and `guestCount` are Luma fields.

```json
{
  "platform": "luma",
  "id": "evt-cJ8w96a4OO6FBzw",
  "url": "https://luma.com/zcbrxhsr",
  "title": "How can AI kill us all? Eight existential risk scenarios with Prof. Geoffrey Hinton",
  "summary": null,
  "description": null,
  "startUtc": "2026-11-20T02:15:00.000Z",
  "endUtc": "2026-11-20T03:45:00.000Z",
  "startLocal": "2026-11-19T18:15:00-08:00",
  "endLocal": "2026-11-19T19:45:00-08:00",
  "timezone": "America/Los_Angeles",
  "isOnline": false,
  "locationType": "offline",
  "venueName": "The Commonwealth Club",
  "address": "The Commonwealth Club, 110 The Embarcadero, San Francisco, CA 94105, USA",
  "street": "110 The Embarcadero, San Francisco",
  "city": "San Francisco",
  "region": "California",
  "postalCode": null,
  "country": "United States",
  "latitude": 37.7933452,
  "longitude": -122.3925028,
  "addressVisibility": "public",
  "organizerName": "Existential Risk Observatory",
  "organizerUrl": null,
  "organizerId": "usr-LK2PVcczMf1IJkR",
  "hostNames": [
    "Existential Risk Observatory"
  ],
  "isFree": true,
  "priceMin": 0,
  "priceMax": 0,
  "currency": null,
  "isSoldOut": true,
  "spotsRemaining": 0,
  "registrationStatus": "waitlist",
  "categories": [],
  "tags": [],
  "imageUrl": "https://images.lumacdn.com/uploads/oc/7e84d152-5f31-4252-ac14-32a69336f92a.png",
  "guestCount": 280,
  "status": null,
  "publishedAt": null,
  "searchInput": "luma | San Francisco | AI",
  "scrapedAt": "2026-11-14T21:48:03.383Z"
}
```

## Known limits

These limits are documented in the actor README:

- **Eventbrite search** stops at 49 pages of 20 events (**about 980**) per search. Split large cities by keyword, category, or a shorter date range. Use **Max events per search** to control size.
- **Luma discover cities:** about **87** named cities are matched by name. Other cities use a nearby search around Eventbrite's coordinates, with a fixed **100 km** radius. The radius is not configurable.
- **Luma hidden addresses** (`guests-only`): the actor returns the city and leaves venue, street, and coordinates `null`, rather than a misleading approximate point.
- **Luma `summary`** is empty in list results. Enable **Include full description** for the full text. Luma categories also appear with that option.
- **Luma online events** come through calendar or event URLs, not through location searches. `location: "online"` is skipped for Luma.
- **Luma category URLs** such as `lu.ma/tech` are rejected, because their results depend on the visitor's IP location. Use `lumaCategory` with `locations` instead.
- The actor does **not** scrape attendees. Luma featured guest lists are dropped. Host names are included because they are shown publicly as the event's organizers.
- With a date range set, events with unknown dates are dropped. Multi-day events that overlap the range are kept.

## Run it from the API

Create an API token in [Apify Console → API & Integrations](https://console.apify.com/settings/integrations). Store it in an environment variable. Do **not** commit the token or put it in a URL that might be logged.

The actor id is `hiringdata/eventbrite-luma-events-scraper` (in REST URLs: `hiringdata~eventbrite-luma-events-scraper`).

The synchronous endpoint waits up to 300 seconds and then returns dataset items. For larger runs, start the actor asynchronously and fetch the dataset when the run succeeds. To **export Eventbrite events to CSV** from the API, request dataset items with `?format=csv` after the run finishes.

### cURL

```bash
curl -X POST \
  "https://api.apify.com/v2/actors/hiringdata~eventbrite-luma-events-scraper/run-sync-get-dataset-items" \
  -H "Authorization: Bearer $APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "locations": ["San Francisco", "London"],
    "keywords": ["AI"],
    "platforms": ["eventbrite", "luma"],
    "startDate": "today",
    "endDate": "+30 days",
    "maxEventsPerSearch": 100
  }'
```

To start a run without waiting:

```bash
curl -X POST \
  "https://api.apify.com/v2/actors/hiringdata~eventbrite-luma-events-scraper/runs" \
  -H "Authorization: Bearer $APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"locations":["San Francisco"],"keywords":["AI"],"platforms":["eventbrite","luma"],"startDate":"today","endDate":"+30 days"}'
```

Then read items from `https://api.apify.com/v2/datasets/<defaultDatasetId>/items` once the run has finished (add `?format=csv` for a CSV export). See the [Run Actor](https://docs.apify.com/api/v2/actors-runs-post), [Run Actor synchronously and get dataset items](https://docs.apify.com/api/v2/actor-run-sync-get-dataset-items-post), and [Get dataset items](https://docs.apify.com/api/v2/dataset-items-get) endpoints.

### Python (`apify-client`)

```bash
pip install apify-client
```

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])

run_input = {
    "locations": ["San Francisco", "London"],
    "keywords": ["AI"],
    "platforms": ["eventbrite", "luma"],
    "startDate": "today",
    "endDate": "+30 days",
    "maxEventsPerSearch": 100,
}

run = client.actor("hiringdata/eventbrite-luma-events-scraper").call(run_input=run_input)

print(f"Dataset: https://console.apify.com/storage/datasets/{run.default_dataset_id}")
for item in client.dataset(run.default_dataset_id).iterate_items():
    print(item["platform"], item["title"], item.get("startLocal"), item.get("organizerName"), item["url"])
```

### Node.js (`apify-client`)

```bash
npm install apify-client
```

```javascript
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({
    token: process.env.APIFY_TOKEN,
});

const input = {
    locations: ['San Francisco', 'London'],
    keywords: ['AI'],
    platforms: ['eventbrite', 'luma'],
    startDate: 'today',
    endDate: '+30 days',
    maxEventsPerSearch: 100,
};

const run = await client.actor('hiringdata/eventbrite-luma-events-scraper').call(input);

console.log(`Dataset: https://console.apify.com/storage/datasets/${run.defaultDatasetId}`);
const { items } = await client.dataset(run.defaultDatasetId).listItems();
for (const item of items) {
    console.log(item.platform, item.title, item.startLocal, item.organizerName, item.url);
}
```

### Apify CLI and MCP

```bash
echo '{
  "locations": ["San Francisco"],
  "keywords": ["AI"],
  "platforms": ["eventbrite", "luma"],
  "startDate": "today",
  "endDate": "+30 days"
}' | apify call hiringdata/eventbrite-luma-events-scraper --silent --output-dataset
```

You can also call the actor from the [Apify MCP server](https://mcp.apify.com/?tools=fetch-actor-details,hiringdata/eventbrite-luma-events-scraper). Official client docs: [JavaScript](https://docs.apify.com/api/client/js/docs) and [Python](https://docs.apify.com/api/client/python/docs/quick-start).

## FAQ

**Which cities work?** Eventbrite: any place Eventbrite has a search page for (city names like `London` or `San Francisco` are resolved by Eventbrite; the resolved place is shown in `RUN_SUMMARY`). Luma: its discover cities by name, other cities via a nearby search around Eventbrite's coordinates for that city, limited to 100 km.

**Is there a Luma events API in this actor?** The actor is a scraper, not an official Luma API. It reads the same public JSON that [lu.ma](https://lu.ma) loads in the browser and returns a normalized schema you can call through the Apify API, the JS/Python clients, or the Apify MCP server.

**How many events can one search return?** Eventbrite search stops at 49 pages of 20 events (about 980) per search; split big cities by keyword, category, or shorter date ranges. Luma lists are paged 50 at a time. Use **Max events per search** to control size.

**Why are some Luma venues and coordinates empty?** Many Luma hosts show the address to registered guests only. In that case Luma publishes only the city and an approximate point; the actor returns the city and leaves venue, street, and coordinates `null` rather than a misleading location.

**Why is the Luma `summary` empty?** Luma lists don't include a short summary. Enable **Include full description** to get the full text.

**Does it scrape attendees?** No. Luma's featured guest lists are dropped. Host names are included because they are shown publicly as the event's organizers.

**Do I need proxies?** Not in the publisher's tests: both sites served their public data to the actor on the Apify platform without a proxy. A proxy option is available if you ever get blocked.

**Is it legal?** The actor only reads public event pages and the public JSON these websites load in the browser, checks each site's `robots.txt` before every request, rate-limits itself, and doesn't log in. You're responsible for how you use the data and for complying with the platforms' terms.

**Can I get only new events every day?** Yes: enable **Only new events since the last run**, set a monitor key, and schedule the actor.

**Can I use it from code or AI agents?** Yes, via the Apify API, the JS/Python clients, or the Apify MCP server.

## Links

- Actor on Apify Store: [https://apify.com/hiringdata/eventbrite-luma-events-scraper](https://apify.com/hiringdata/eventbrite-luma-events-scraper)
- Apify Console: [https://console.apify.com/actors/hiringdata~eventbrite-luma-events-scraper](https://console.apify.com/actors/hiringdata~eventbrite-luma-events-scraper)
- Publisher profile: [https://apify.com/hiringdata](https://apify.com/hiringdata)
- Open an issue on the actor's [Issues](https://apify.com/hiringdata/eventbrite-luma-events-scraper/issues/open) tab if an event page or city does not work or you need another platform.

The static landing page for this tutorial lives in [`docs/index.html`](docs/index.html).
