# HasData plugin for Cursor

The public web as structured JSON, from inside Cursor. One hosted MCP server covers Google and Bing results, Amazon and Walmart listings, Google Maps and Yelp places, Zillow and Redfin properties, Airbnb and Booking stays, Indeed and Glassdoor postings, TikTok, Instagram, Facebook and YouTube, plus a general page scraper for everything else.

The scraping runs on HasData infrastructure. There is no headless browser to install, no proxy pool to keep alive and no container to run.

```
https://mcp.hasdata.com/mcp
```

[![MCP](https://img.shields.io/badge/MCP-remote%20%7C%20streamable%20HTTP-6366f1?style=flat-square)](https://modelcontextprotocol.io)
[![Cursor plugin](https://img.shields.io/badge/Cursor-plugin-000000?style=flat-square)](https://cursor.com/marketplace)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)

**1,000 free credits every month, no card required.**

## Contents

- [What you need](#what-you-need)
- [Install](#install)
- [Authentication](#authentication)
- [What is inside](#what-is-inside)
- [Sites covered](#sites-covered)
- [Narrowing the tool list](#narrowing-the-tool-list)
- [Example prompts](#example-prompts)
- [Errors and failure paths](#errors-and-failure-paths)
- [Credits and the free tier](#credits-and-the-free-tier)
- [Repository layout](#repository-layout)
- [FAQ](#faq)
- [HasData links](#hasdata-links)
- [Contributing](#contributing)
- [License](#license)

## What you need

Cursor, and a HasData API key from the [dashboard](https://app.hasdata.com/sign-up?utm_source=github&utm_medium=syndication&utm_campaign=cursor-plugin). The key is free to create with no card. This is a remote server, so a URL and a header is the whole setup.

## Install

From the Cursor marketplace, or with the slash command in chat:

```
/add-plugin hasdata
```

To install straight from this repository instead, point Cursor at it and it reads `plugins/hasdata`.

## Authentication

Cursor asks for the key when you install the plugin. The manifest declares a `HASDATA_API_KEY` variable, and the bundled `mcp.json` sends it as a header:

```json
{
  "mcpServers": {
    "hasdata": {
      "type": "http",
      "url": "https://mcp.hasdata.com/mcp",
      "headers": { "x-api-key": "${HASDATA_API_KEY}" }
    }
  }
}
```

To change the key later, open the plugin in Cursor settings and edit the variable. The tool list loads without a key. A missing or wrong key fails on the tool call, usually as an error inside the tool result, which is the failure to expect when the tools appear but nothing comes back.

This install uses the API key. The server also accepts OAuth 2.1, but only on a connection that does not send `x-api-key`. The bundled file is read-only after install, so do not delete its `headers` block to switch modes.

## What is inside

### Agent

`web-data-researcher` turns a question about the live web into the right calls across every service, then reports what the data supports and what it does not. It routes by subject rather than by wording, prefers a dedicated tool over the page scraper, and states how many records a conclusion rests on.

### Skills

Cross-service workflows, for the jobs that need several sites at once.

| Skill | What it does |
| :--- | :--- |
| `price-across-retailers` | Prices one product on Amazon, Walmart, Google Shopping and a named Shopify storefront, matching on brand and model before comparing |
| `local-business-dossier` | Profiles a local business from Google Maps, Yelp, Yellow Pages and Facebook, and reconciles what the four directories disagree on |
| `serp-visibility` | Ranks a domain or a query on Google, Bing and DuckDuckGo in one pass, with the AI Overview, the Copilot answer and Search Assist |
| `creator-shortlist` | Vets creators on TikTok, Instagram and YouTube, and says which engagement questions each platform can actually answer |

### Commands

One slash command per site, for the job people actually run.

| Command | What it returns |
| :--- | :--- |
| `/serp-snapshot` | A snapshot of who ranks for a query, with the AI Overview and the questions people ask |
| `/bing-vs-google` | How a query ranks on Bing, and where that differs from Google |
| `/ddg-baseline` | An unpersonalised ranking for a query, used as a baseline |
| `/trend-check` | Whether interest in a term is rising or falling, with the regions and the related queries |
| `/image-sourcing` | Find images for a subject at a usable size and check they actually load |
| `/lit-sweep` | A literature sweep on a topic, with citation counts and the papers that cite the leaders |
| `/amazon-listing-audit` | What one Amazon listing looks like to a buyer, and what reviewers complain about |
| `/walmart-stock-check` | Price and availability for a Walmart item, pinned to one store |
| `/catalog-scan` | What a Shopify store sells, by collection and by price band |
| `/local-leads` | A contact list of local businesses for a category and a city, from Google Maps |
| `/local-vendors` | A vetted shortlist of local businesses for a trade and a place |
| `/yelp-reputation` | What Yelp reviewers say about one business, grouped into themes |
| `/zillow-comps` | Comparable sales for an address or an area, pulled from Zillow |
| `/comp-set` | A comparable set for a home, built from recent Redfin sales |
| `/stay-shortlist` | A shortlist of Airbnb stays for a destination, dates and party size |
| `/hotel-shortlist` | A shortlist of Booking.com hotels for a destination, dates and party |
| `/rate-check` | What a hotel actually costs for given dates, across every source Google compares |
| `/route-fares` | Fares for a route and date, ranked with the emissions and the price level |
| `/hiring-map` | Who is hiring for a role across several cities, with salaries where employers publish them |
| `/salary-benchmark` | What a role pays in a given market, from Glassdoor postings |
| `/tiktok-pulse` | What an account posts on TikTok and which videos actually performed |
| `/profile-brief` | A brief on a public Instagram account and what it has been posting |
| `/page-snapshot` | What a public Facebook page looks like right now, with its recent posts |
| `/video-research` | What ranks on YouTube for a query, and what the leading videos actually say |
| `/page-extract` | Pull named fields off a web page, rendering it first when the page needs it |

### Rules

A rule per site, written from measured payloads rather than from documentation. Each covers what a tool description cannot say. Some identifiers are marketplace-scoped. Some keys are absent rather than empty. Some failures answer 200 with an error string inside, and some numbers arrive as strings.

The rules are agent-requested, so they stay out of context until the agent judges one relevant.

| Rule | Covers |
| :--- | :--- |
| `hasdata-umbrella` | The full server, how `?apis=` filtering works, and how tool names map to services |
| `hasdata-google-search` | Google results, AI Overview, news and shopping |
| `hasdata-bing` | Bing results, ads and the Copilot answer |
| `hasdata-duckduckgo` | DuckDuckGo results and the Search Assist answer |
| `hasdata-google-images` | Google Images results, filtered by size, colour and type |
| `hasdata-google-trends` | Interest over time, regional breakdowns and related queries |
| `hasdata-google-scholar` | Scholar results, citation graphs and formatted citations |
| `hasdata-amazon` | Amazon search, product, review and seller data |
| `hasdata-walmart` | Walmart search, product and review data |
| `hasdata-shopify` | Products and collections from any Shopify storefront |
| `hasdata-google-maps` | Maps places, reviews, photos and posts |
| `hasdata-yelp` | Yelp business search, place details and reviews |
| `hasdata-yellowpages` | Yellow Pages business listings and place details |
| `hasdata-facebook` | Public Facebook pages and their post feed |
| `hasdata-zillow` | Zillow listings and property details |
| `hasdata-redfin` | Redfin listings and property details |
| `hasdata-airbnb` | Airbnb stays and listing details |
| `hasdata-booking` | Booking.com hotel search and property details |
| `hasdata-google-hotels` | Google Hotels properties, rates and per-source pricing |
| `hasdata-google-flights` | Google Flights itineraries, fares and emissions |
| `hasdata-indeed` | Indeed job listings and postings |
| `hasdata-glassdoor` | Glassdoor listings, salaries and employer ratings |
| `hasdata-tiktok` | TikTok profiles, post feeds, keyword search and comments |
| `hasdata-instagram` | Public Instagram profiles and post feeds |
| `hasdata-youtube` | YouTube search, video metadata, channels and transcripts |
| `hasdata-web-scraping` | Arbitrary pages, JavaScript rendering and field extraction |

## Sites covered

Google Search, Google Images, Google Trends, Google Scholar, Google Maps, Google Hotels, Google Flights, Bing, DuckDuckGo, Amazon, Walmart, Shopify storefronts, Yelp, Yellow Pages, Zillow, Redfin, Airbnb, Booking.com, Indeed, Glassdoor, TikTok, Instagram, Facebook, YouTube, and any other public page through the Web Scraping tool.

## Narrowing the tool list

The full endpoint exposes every tool at once. A model choosing among dozens of similarly shaped tools picks wrong more often than one choosing among five, and every tool description occupies context whether or not it gets called.

The marketplace copy of `mcp.json` is read-only, and an edit there is overwritten on update. To filter, add your own server in Cursor's user MCP config. That file interpolates `${env:NAME}`, not the plugin's `${HASDATA_API_KEY}`:

```json
{
  "mcpServers": {
    "hasdata-amazon-walmart": {
      "type": "http",
      "url": "https://mcp.hasdata.com/mcp?apis=amazon,walmart",
      "headers": { "x-api-key": "${env:HASDATA_API_KEY}" }
    }
  }
}
```

Services combine comma-separated or by repeating the parameter. If every name is unknown the server returns HTTP 400. If at least one name is valid, the unknown ones are ignored, so `amazon,walmrart` returns only Amazon.

The service segment is the one that appears in every tool name, which follows `hasdata_<service>_<group>_<method>`. It does not always match the product name. Google Search is `google_serp`, flights are `google_travel_flights` and hotels are `google_travel_hotels`.

## Example prompts

```
Who ranks in the top ten for "best crm for small business" in the US, and is our domain in the AI Overview?

Compare the price of the Dyson V15 on Amazon and Walmart for delivery to 78701.

Build me a contact list of plumbers in Austin with phone, site and rating.

What did recent comparable sales go for near 512 Elm St, and how many sales is that based on?

Which channels rank for "web scraping tutorial" on YouTube, and what do the top three actually cover?
```

## Errors and failure paths

A successful call is not the same as a useful answer, and these services fail in ways that look like data.

Several answer HTTP 200 with an error string where the payload should be. TikTok does this for a handle that does not exist, and Shopify does it for a URL that is not a classic storefront, with `requestMetadata.status` still reading `ok`.

Several omit a key rather than returning it empty. On Bing and DuckDuckGo the ad block and the AI answer are absent when the page has none, and a Yelp search with no match drops `organicResults` while still returning `ads`.

Several never return nothing. DuckDuckGo answers a nonsense query with ten loosely related results, Bing does the same, and Google Maps returns a single guessed place.

A validation failure names the field and is not billed. It often arrives inside the tool result rather than as an HTTP status. A 401 means the key is missing or wrong, and it arrives on the tool call, not when the tool list loads.

The per-site rules carry the specific cases. Read them rather than inferring one site from another.

## Credits and the free tier

Every successful call spends credits from the connected account. A call that fails validation is not billed. A request that returns an empty result set is a successful call and is billed.

The free tier is 1,000 credits a month with no card. Current per-call costs are on the [plans page](https://hasdata.com/prices?utm_source=github&utm_medium=syndication&utm_campaign=cursor-plugin).

## Repository layout

```
.cursor-plugin/marketplace.json   marketplace manifest
plugins/hasdata/
  .cursor-plugin/plugin.json      plugin manifest
  assets/logo.svg
  mcp.json                        the hosted MCP server
  agents/                         the research subagent
  skills/                         cross-service workflows
  commands/                       one slash command per site
  rules/                          one .mdc per site, agent-requested
  README.md
```

The repository follows the [cursor/plugin-template](https://github.com/cursor/plugin-template) layout. To check it, run the template validator with this repository as the working directory:

```bash
node /path/to/plugin-template/scripts/validate-template.mjs
```

## FAQ

### Do I need an API key for each site?

No. One HasData key covers every site in the table above. There is no Google Cloud project to set up for Maps or YouTube, and no Zillow or Yelp partner approval to obtain.

### Does this run a browser on my machine?

No. The rendering, proxying and parsing happen on HasData infrastructure, and Cursor talks to one HTTPS endpoint.

### What does it read?

Public pages that a signed-out visitor can see, returned as structured JSON.

### Why are there so many rules?

Because the traps are specific to each site and a generic rule would be wrong about most of them. They load only when relevant, so the set costs nothing until the agent reaches for one.

### Can I use one site instead of all of them?

Yes. Each service also ships as a standalone MCP server behind a shorter URL. `?apis=` on this plugin's bundled file will not stick, because Cursor keeps that file read-only. Add a separate server in your own MCP config, as in Narrowing the tool list.

### Is HasData affiliated with the sites it reads?

No. HasData is an independent web data provider and is not affiliated with, endorsed by or sponsored by any of the sites listed here. All trademarks belong to their owners.

## HasData links

| | |
| :--- | :--- |
| Product and request builder | [HasData APIs](https://hasdata.com/apis/?utm_source=github&utm_medium=syndication&utm_campaign=cursor-plugin) |
| Server documentation | [MCP server docs](https://docs.hasdata.com/mcp-server?utm_source=github&utm_medium=syndication&utm_campaign=cursor-plugin) |
| Per-service MCP servers | [MCP servers](https://hasdata.com/mcp?utm_source=github&utm_medium=syndication&utm_campaign=cursor-plugin) |
| Client walkthroughs | [MCP clients and integrations](https://hasdata.com/integrations/mcp?utm_source=github&utm_medium=syndication&utm_campaign=cursor-plugin) |
| Plans and credit costs | [Plans and credit costs](https://hasdata.com/prices?utm_source=github&utm_medium=syndication&utm_campaign=cursor-plugin) |

## Contributing

The rules and commands are written from measured API payloads rather than from documentation, so a correction is welcome when a field or a failure mode has changed. Open an issue with the tool name and the response you saw.

## License

MIT
