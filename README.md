# Debriefing MCP server

Debriefing is competitive intelligence for B2B teams. It watches your competitors, finds their moves (a price change, a launch, a new hire), and backs every signal with the source evidence. This MCP server lets Claude, ChatGPT, Cursor, and any other MCP client work on a Debriefing workspace.

This repository holds the public listing for the server: this README and `server.json` for the [official MCP Registry](https://registry.modelcontextprotocol.io). The server itself is hosted. There is nothing to install or run.

- Website: https://debriefing.io
- Docs: https://debriefing.io/developers?tab=mcp
- Server card: https://debriefing.io/.well-known/mcp/server-card.json

## Endpoint

| | |
| --- | --- |
| URL | `https://debriefing.io/mcp` |
| Transport | Streamable HTTP |
| Registry name | `io.debriefing/debriefing` |
| Tools | 41 |

## Auth

- **OAuth 2.1** (Claude, ChatGPT, and other OAuth clients). The person signs in to Debriefing, approves the app, and picks the scopes: `read` (always), `write` (changes that cost nothing), and `spend` (actions that use a plan allowance). PKCE S256, dynamic client registration, and Client ID Metadata Documents are supported. Metadata: `https://debriefing.io/.well-known/oauth-authorization-server` and `https://debriefing.io/.well-known/oauth-protected-resource`.
- **API key** (other clients). Create a key in the Debriefing app and send it as `Authorization: Bearer <key>` or `X-API-Key: <key>`. A key acts as the admin who created it.
- **No credential.** `tools/list` and the four public tools work without one.

Each tool checks its scope, the person's role, and the plan. A refusal comes back as `isError` with the link that fixes it.

## Connect

**Claude** (claude.ai, Desktop): Settings, Connectors, Add custom connector, URL `https://debriefing.io/mcp`.

**Claude Code:**

```
claude mcp add --transport http debriefing https://debriefing.io/mcp
```

**Cursor and other JSON-config clients:**

```json
{
  "mcpServers": {
    "debriefing": { "url": "https://debriefing.io/mcp" }
  }
}
```

**ChatGPT:** add `https://debriefing.io/mcp` as a connector in developer mode.

## Tools

### Public tools (no sign-in)

They read only the briefs and guides Debriefing publishes on debriefing.io.

| Tool | What it does |
| --- | --- |
| `search_public_briefs` | Searches the competitive briefs Debriefing publishes on debriefing.io about named companies, newest first. Each brief is dated and every finding cites a public source. Filter by words in the brief and by company name, slug, or domain. With no arguments, lists the newest briefs. Needs no Debriefing account. Returns the page URL of each brief. |
| `get_public_brief` | Gets one competitive brief that Debriefing published on debriefing.io: the summary, each finding with its category and date, and the numbered public sources with their URLs. Name the brief by its URL, or by company_slug and slug from search_public_briefs. With company_slug only, gets the newest brief about that company. Needs no Debriefing account. |
| `search_guides` | Searches the public competitive intelligence library on debriefing.io: how-to guides, glossary terms, tool comparisons, alternatives pages, company profiles, and market intel posts. Returns the title, a short excerpt, and the page URL of each match, best match first. Needs no Debriefing account. |
| `get_free_brief_link` | Gets the debriefing.io links where a person requests a free competitive brief about their own company and competitors, signs up for a workspace, or compares plans. It only returns links. It sends nothing and creates nothing. |

### Read tools (scope `read`)

Read the workspace. They change nothing.

| Tool | What it does |
| --- | --- |
| `get_workspace` | Gets the Debriefing workspace this connection works on: its name and website, who you are in it and your role, the scopes this connection holds, and the plan. Call it first when you are not sure what you may do. |
| `get_plan_and_limits` | Gets the workspace plan and every limit an agent runs into: competitor slots, the weekly research allowance, the daily caps on battlecards, ICP refreshes and positioning drafts, share link lifetimes, monitoring cadences, and which features need a paid plan. Includes the pricing link. |
| `list_competitors` | Lists the competitors this Debriefing workspace tracks, with the id, name, website, and category of each. Call it first to get the competitor_id that other tools take. |
| `get_competitor` | Gets the dossier for one tracked competitor: company facts, the risk and opportunity it poses to this workspace, and the stored reads on positioning, product, pricing, funding, traffic, and more. Use it when the user asks about one competitor in depth. |
| `search_signals` | Searches open signals, newest first. A signal is a competitor move that Debriefing found, such as a price change or a launch, with the source evidence. Filter by competitor_id, severity (low, medium, high), a text query, or a since time. Use it when the user asks what a competitor changed. |
| `list_digests` | Lists the digests Debriefing wrote for this workspace, newest first. A digest is a brief that sums up competitor moves over a period. Filter by type, such as daily_digest. |
| `get_digest` | Gets one digest by id or slug, or the newest digest when no id is given. Pass type daily_digest for the newest daily digest. Use it when the user asks for the latest brief. |
| `list_battlecards` | Lists the battlecards Debriefing wrote for this workspace, one for each competitor that has one. A battlecard holds talk tracks for sales calls against that competitor. |
| `get_battlecard` | Gets one battlecard by competitor_id or battlecard_id. Use it when the user prepares a sales call against a competitor. |
| `get_evidence` | Gets the evidence behind one signal: each source excerpt with its URL and date. Use it when the user asks where a claim comes from. |
| `get_company_profile` | Gets the workspace's own company profile: description, category, target customer, positioning lists, and the ideal customer profile. Debriefing reads every competitor move against it. |
| `get_delivery_settings` | Gets how often Debriefing runs a monitoring cycle for the workspace, the cadences the plan allows, and whether a finished cycle goes out by email and to Slack. Admin only. |
| `list_members` | Lists the people in the workspace with their roles. An admin also sees pending invitations. |
| `what_changed` | Lists everything new in the workspace since a time: signals Debriefing found, digests it wrote, and battlecards it wrote or rewrote. Use it to catch up, such as since the user's last visit. |

### Write tools (scope `write`)

Change the workspace. They cost nothing.

| Tool | What it does |
| --- | --- |
| `add_competitor` | Adds a competitor to the workspace's core watchlist from its website. Debriefing then monitors it. Admin only. The plan sets how many core competitors fit; get_plan_and_limits shows the free slots. |
| `rename_competitor` | Changes the display name of one tracked competitor. Admin only. |
| `remove_competitor` | Takes one core competitor off the watchlist. Debriefing stops monitoring it and keeps its history. Admin only, on a paid plan. |
| `add_monitored_page` | Asks Debriefing to watch one more page of a core competitor, such as its pricing page. Changes on it then show up as signals. Up to 10 pages per competitor. Admin only, on a paid plan. |
| `remove_monitored_page` | Stops watching a page that someone added on a core competitor. Name it by page_id or url. Admin only, on a paid plan. |
| `pin_signal` | Pins a signal for the whole team, or removes the pin with pinned false. |
| `save_signal` | Saves a signal to your own saved list, or removes it with saved false. Only you see your saved signals. |
| `add_signal_note` | Adds a talking point to a signal. The team sees it in the workspace. It never appears on a public link unless the link includes notes. |
| `rate_signal` | Tells Debriefing whether a signal was useful to you. For not useful, add reasons and a comment. A new rating replaces your old one. |
| `pin_digest` | Pins a digest for the whole team, or removes the pin with pinned false. |
| `add_digest_note` | Adds a talking point to a digest. The team sees it in the workspace. |
| `rate_digest` | Scores a digest from 1 to 5, says what is missing from it, or both. A new rating replaces your old one. |
| `add_battlecard_note` | Adds what the team knows about a core competitor that no page says. The next rewrite of its battlecard reads the note as framing, never as a verified fact. |
| `update_company_profile` | Changes fields of the workspace's company profile. Only the fields you send change. The lists replace the old lists. Admin only. Locked while the first brief runs. |
| `set_monitoring_cadence` | Sets how often Debriefing runs a monitoring cycle. The plan decides which cadences are allowed; get_delivery_settings lists them. Admin only, on a paid plan. |
| `set_delivery_channels` | Turns delivery of a finished cycle by email and to Slack on or off. Only the channels you send change. Slack needs a paid plan and a Slack connection made in the app. Admin only. |
| `create_share_link` | Makes a public link to one signal that anyone with the address can open. A new link replaces the old one, which stops working. The plan sets how long a link can live; the free plan and Starter allow 24 hours. |
| `invite_member` | Sends an email invitation to join the workspace as a member. The invitation lasts 7 days. Inviting the same address again sends a fresh email. Admin only, on a paid plan. |

### Spend tools (scope `spend`)

Use a plan allowance, such as the weekly research allowance or a daily cap.

| Tool | What it does |
| --- | --- |
| `research_competitor` | Starts the first research on one core competitor that has none yet: Debriefing reads its site and outside sources. It uses one update from the weekly research allowance. Admin only. |
| `refresh_watchlist` | Refreshes the research on the core watchlist now, instead of at the next cycle. It uses one update from the weekly research allowance. Admin only. |
| `generate_battlecard` | Writes or rewrites the battlecard for one core competitor now, instead of at the next cycle. It takes about a minute; read it afterwards with get_battlecard. The competitor needs finished research. Needs a paid plan or the free first week, and counts toward a daily cap per workspace. |
| `update_icp` | Sets the ideal customer profile, then re-reads how much each tracked competitor overlaps it. Only the dimensions you send change; an empty list clears one. Admin only. Counts toward a daily cap per workspace. Locked while the first brief runs. |
| `draft_positioning` | Reads the company's own website and drafts the empty positioning fields: value propositions, differentiators, and exclusions. Filled fields stay as they are. Read the result later with get_company_profile. Admin only. Counts toward a daily cap per workspace. |

## Support

Questions and bugs: open an issue here, or write to the contact on https://debriefing.io.

## License

The files in this repository are MIT licensed. The Debriefing service and its source code are proprietary.
