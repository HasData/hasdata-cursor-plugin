# HasData for Cursor

Public web pages as structured JSON, from inside Cursor. Google and Bing results, Amazon and Walmart listings, Maps and Yelp places, Zillow and Redfin properties, Airbnb and Booking stays, Indeed and Glassdoor postings, TikTok, Instagram, Facebook and YouTube, plus a general page scraper for everything else.

The scraping runs on HasData infrastructure, so there is no headless browser to install and no proxy pool to keep alive.

## Install

Install the plugin from the Cursor marketplace. Cursor asks for your HasData API key during install; get one at [app.hasdata.com/api-keys](https://app.hasdata.com/api-keys). The free tier does not need a card.

The bundled `mcp.json` sends that key as the `x-api-key` header. The tool list loads without a key. A missing or wrong key fails on the tool call, usually as an error inside the tool result, which is the failure to expect when tools appear but nothing returns. To change the key later, open the plugin in Cursor settings and edit the variable.

This install uses the API key. The server also accepts OAuth 2.1, but only on a connection that does not send `x-api-key`. The bundled file is read-only after install, so do not delete its `headers` block to switch modes.

## What is in the box

**One MCP server** at `https://mcp.hasdata.com/mcp`, exposing every HasData tool.

**A rule per site.** Each one is written from measured payloads and covers the traps that a tool description cannot: which identifiers are marketplace-scoped, which fields are absent rather than empty, which failures answer 200 with an error string inside, and which numbers are strings. They load only when the agent judges them relevant, so the whole set costs nothing until it is needed.

**A command per site**, for the jobs people actually run. Comparable sales from Zillow, a reputation read from Yelp, a hiring map from Indeed, a rate check on Google Hotels, and so on.

**Cross-service skills** for the work that needs several sites at once. Pricing a product across Amazon, Walmart and Google Shopping. Building a local business dossier from Maps, Yelp, Yellow Pages and Facebook. Checking visibility across Google, Bing and DuckDuckGo including the AI answers. Vetting creators across TikTok, Instagram and YouTube.

## Narrowing the tool list

The full endpoint exposes every tool at once, 68 of them. A model choosing among dozens of similarly shaped tools picks wrong more often than one choosing among five, and every tool description occupies context whether or not it gets called.

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

The service segment is the same one that appears in every tool name, which follows `hasdata_<service>_<group>_<method>`. It does not always match the product name: Google Search is `google_serp`, flights are `google_travel_flights` and hotels are `google_travel_hotels`.

## Billing

Every successful call spends credits from the connected account. A call that fails validation is not billed. A request that returns an empty result set is a successful call.

## Links

- [HasData](https://hasdata.com)
- [API documentation](https://docs.hasdata.com)
- [Per-service MCP servers](https://hasdata.com/mcp), for clients that only ever need one site

## License

MIT
