# Changelog

## 1.2.1

- The Cursor manifest now points at `mcp.json`, so the marketplace install registers the server.
- Cursor asks for the HasData API key at install through a declared `HASDATA_API_KEY` variable, instead of expecting it in the shell environment.
- MCP server type is `http`, matching the published plugins in `cursor/plugins`.
- Added `category`, `tags` and a minimum Cursor version. Dropped the `url` field from `author`, which the plugin schema does not allow.
- The umbrella rule and both READMEs now say the server lists 68 tools. A `?apis=` value fails only when every name is unknown; a mix keeps the valid names and ignores the rest. The installed `mcp.json` is read-only, so the filter example is a separate user MCP entry.
- Removed the instruction to delete the `headers` block for OAuth. This install uses the API key, and a wrong key fails on the tool call, not when the tool list loads.
- `/bing-vs-google` calls Google on the same server. `/profile-brief` ranks a post through the Instagram comments tool instead of saying the feed cannot. The Yelp rule treats a miss as an absent `organicResults` key, and does not read `ads` as matches. Google Search names the real overview, events and immersive-product tools.

## 1.2.0

- Instagram comments return per-post `likesCount`, `commentsCount` and `timestamp`, so the Instagram rule and the creator-shortlist skill now explain how to rank a feed by engagement instead of saying it cannot be done.
- Google Scholar rule covers the new case law tool, which takes `caseId` rather than `q`.
- Added a `.claude-plugin/plugin.json` manifest beside the Cursor one, so the same directory installs from marketplaces that read the Claude shape.

## 1.1.0

- Added the `web-data-researcher` agent, which routes a question about the live web to the right service and reports what the data supports.
- Rewrote the skill descriptions so they name the triggers that should reach for them.
- Expanded the repository README with the full command, rule and skill tables, failure paths and a FAQ.

## 1.0.0

- First release. One hosted MCP server, a rule and a slash command per site, and four cross-service skills.
