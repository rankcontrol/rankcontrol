---
name: rankcontrol
description: Operate the user's RankControl workspace (rctrl.com) from the command line - AI search visibility and citations (ChatGPT, Perplexity, Claude, Gemini, Grok, Google AI Mode), SEO content planning, article generation and publishing to their CMS, backlinks and outreach, social repurposing, brand memory, and reports. Use this whenever the user asks about AI visibility, AI citations, GEO/AEO, being cited by ChatGPT, their content calendar, publishing articles, AI traffic attribution, or anything in their RankControl account, even if they don't name RankControl explicitly.
homepage: https://rctrl.com/docs
metadata: {"openclaw":{"emoji":"📈","requires":{"bins":[],"env":[]}}}
---

# RankControl CLI

RankControl (https://rctrl.com) wins Google rankings and AI citations for a
website: it plans content, writes articles, publishes them as native posts on
the customer's own CMS, and tracks citations and traffic across AI
search engines. The `rankcontrol` CLI is the whole workspace in the terminal.
Every command prints JSON.

Full reference: https://rctrl.com/docs (also https://rctrl.com/docs/llms.txt)

## Install and authenticate

```bash
# Install (or use `npx rankcontrol ...` without installing)
npm install -g rankcontrol

# A human at the keyboard: browser device authorization
rankcontrol login

# Headless / CI / agent sandboxes: use an API key from
# Dashboard -> Settings -> API Keys
export RANKCONTROL_API_KEY=rctrl_pk_...
```

Verify before doing anything else:

```bash
rankcontrol score
```

If it errors, stop and sort out auth first. An `HTTP 401` means no valid key;
an `HTTP 403` means the key is missing a scope for that command (scopes are
set per key in the dashboard; read-only keys are common). Tell the user which
scope is missing rather than retrying.

## Ground rules

1. **Dry run means dry run.** Commands marked **dry run** below do nothing
   until `--confirm` is added. That gap exists so a human can approve actions
   that are publicly visible (publishing an article, emailing a prospect,
   retiring a live link) or that are costly to run (article generation).
   Show the user the dry-run output and let them decide. Never add
   `--confirm` on your own initiative.
2. **Reads are safe.** Every `get`/`list`-style command is free to run as
   often as needed to answer a question.
3. **Rate limits are server-side.** Costly routes are capped per minute. On an
   `HTTP 429`, back off and slow down; do not retry in a tight loop.
4. **Propose, then commit.** `plan-content` only drafts candidate titles.
   Nothing lands on the calendar until `commit-titles`, which is the review
   gate. Show proposed titles to the user before committing.
5. **Sanity-check surprising data before alarming anyone.** A score that
   drops to zero overnight, or long runs of checks with empty responses, can
   be a checking-pipeline hiccup rather than a real visibility collapse.
   Present it as "the data shows X, worth verifying" and offer to file a
   ticket instead of declaring a crisis.
6. Something broken or surprising? `rankcontrol support "<subject>" --message
   "<details>"` opens a ticket with the RankControl team (capped 3/min).

## The core loop

```bash
rankcontrol score                # where visibility stands
rankcontrol ideas                # scored gaps: uncovered queries, citation gaps
rankcontrol plan-content --citation-gaps   # propose titles (nothing scheduled)
# human reviews titles ->
rankcontrol commit-titles --file approved.json
rankcontrol generate <contentId> --confirm  # write the article (after approval)
rankcontrol publish <contentId>             # dry run: shows what would happen
rankcontrol publish <contentId> --confirm   # human said go
rankcontrol visibility --days 30            # watch the trend
```

## Command reference

### Setup

| Command | What it does |
| --- | --- |
| `login` | Browser device authorization; saves a key to `~/.rankcontrol`. Scoped/read-only keys are created in the dashboard instead. |
| `logout` | Remove the locally stored key. |
| `mcp` | Start the MCP server on stdio (same tools, for MCP clients). |

### Overview and reports

| Command | What it does |
| --- | --- |
| `funnel` | AI pipeline, last 30 days: crawler hits, AI-referred visits. |
| `report` | Executive summary: last 30 days vs the 30 before. |
| `wins` | Biggest wins: most-cited page, cite-rate jump, ranking climb, best backlink. |
| `agent-activity` | Recent runs per agent lane. `--per-agent <n>`, `--since-days <n>`. |
| `jobs` | Recent agent runs and async job status. `--limit <n>`. |
| `traffic` | Page views, visitors, sessions, bounce rate. `--days <n>`. |
| `engagement` | Per-page views and citations with 30-day sparklines. |

### Visibility

| Command | What it does |
| --- | --- |
| `score` | Composite visibility score with per-pillar subscores. |
| `visibility` | Daily AI visibility trend. `--days 30\|60\|90`. |
| `sov` | Share of voice vs competitors. `--days <n>`. |
| `sources` | Domains AI answers cite for tracked queries. `--days <n>`. |
| `sentiment` | How AI answers frame the brand when cited. `--days <n>`. |
| `citations` | Recent citation checks. `--model chatgpt\|perplexity\|claude\|gemini\|grok\|google_ai_mode`, `--limit <n>`. |
| `queries` | The tracked-query pool checked weekly across AI engines. |
| `query-add <queryText>` | Add a query to the pool. |
| `query-edit <queryId> <text>` | Rewrite a tracked query; future checks use the new text. |
| `query-track <queryId> <on\|off>` | Pause or resume weekly checks; tracking consumes a plan slot. |
| `query-remove <queryId>` | Delete a query from the pool. |
| `competitors` | Tracked competitors used in share-of-voice comparisons. |
| `competitor-add <name> <url>` | Track a competitor (max 10). |
| `competitor-remove <competitorId>` | Stop tracking a competitor. |
| `crawler-access` | Is the site's edge or robots.txt blocking GPTBot and ClaudeBot. |

### Content

Article lifecycle: `planned` (title on the calendar, no body yet) -> `draft`
(body written, ready to publish) -> `published` (live on the customer's site)
-> `archived`. There is no "approved" status; generation progress is a
separate field, not a status. `content` and `ideas` return bare JSON arrays.

| Command | What it does |
| --- | --- |
| `content` | List content pages with status. |
| `ideas` | Scored idea backlog: uncovered queries, citation gaps, quick wins. |
| `plan-idea <queryText>` | Put one idea on the calendar. `--title` to pin the headline. |
| `plan-content` | Generate candidate titles. Nothing schedules until `commit-titles`. Filters: `--topics a,b`, `--content-types a,b`, `--max-difficulty <n>`, `--intents informational,commercial,transactional`, `--citation-gaps`, `--ranking-gaps`. |
| `commit-titles` | Approve reviewed titles onto the calendar from JSON (`--file` or stdin). |
| `capacity` | Remaining plan slots on the calendar. |
| `generate <contentId>` | Write a planned article's body. **Dry run**; costly, runs on `--confirm`. |
| `publish <contentId>` | Publish to the connected CMS. **Dry run** until `--confirm`. |
| `reschedule <contentId> <date>` | Move a planned article to a day (`YYYY-MM-DD`). |
| `delete-planned <contentId>` | Remove a planned title; it returns to the idea pool. |
| `archive <contentId>` | Archive an article out of the working set. |
| `topics` | The pillar list (topic clusters) ideas and articles group under. |
| `topics-set <topics...>` | Replace the FULL pillar list; read `topics` first and send every topic that should exist. |
| `optimizer` | Published pages ranked by citability, worst first, with open fixes. |
| `internal-links <contentId>` | Inbound and outbound internal links for an article. |

### Article policy

| Command | What it does |
| --- | --- |
| `settings` | Show the article policy: per-article defaults plus scheduling mode. |
| `settings-set` | Merge-patch the policy; only flags you pass change. Flags: `--auto-publish on\|off`, `--auto-generate on\|off`, `--images on\|off`, `--title-in-hero on\|off`, `--section-infographics on\|off`, `--related-reading on\|off`, `--youtube on\|off`, `--emojis on\|off`, `--internal-links <n>`, `--external-links <n>`, `--instructions <text>`, `--flexible-schedule on\|off`, `--image-style <set>`. |

### Site links

| Command | What it does |
| --- | --- |
| `site-pages` | Pages used for in-article links and Related Reading. |
| `detect-links <url>` | Scan a sitemap for site pages. `--blog-root` to crawl a page instead. |
| `add-pages <urls...>` | Add page URLs for internal linking; dedupes, titles auto-fill. |

### Repurposing

| Command | What it does |
| --- | --- |
| `repurpose [contentId]` | The repurpose queue, or full drafts for one article. |
| `repurpose-generate <contentId>` | Draft social posts. **Dry run**; `--platforms a,b` to narrow. |
| `repurpose-edit <draftId>` | Edit a draft. `--body`, `--title`. |
| `repurpose-channels` | Connected Postiz channels and their ids. |
| `repurpose-push <draftId>` | Send to Postiz. **Dry run**; `--channels id1,id2`, `--when now\|schedule\|draft`, `--date <iso>`. |
| `repurpose-mark-posted <draftId>` | Mark a draft posted when published manually. |

### Link building

| Command | What it does |
| --- | --- |
| `backlinks` | The backlink table, newest first. `--status <status>`. |
| `backlink-stats` | Totals plus outreach pipeline counts. |
| `outreach-prospects` | Prospect sites per published article, with found contacts. |
| `outreach-find-contact <backlinkId>` | Find a contact email for a prospect. |
| `outreach-draft-reply <backlinkId>` | AI-draft a reply to an inbound response. Nothing sends. |
| `outreach-queue <backlinkId>` | Queue an outreach email. **Dry run**; `--subject`, `--body`. |
| `outreach-status <backlinkId> <status>` | Move a prospect: `identified`, `contacted`, `replied`, `link_placed`, `rejected`. |
| `network` | Link Network credits, membership state, and placements. |
| `network-opt-in <on\|off>` | Join or leave the Link Network. **Dry run**; leaving retires live links. |
| `network-remove-placement <id>` | Retire one placement. **Dry run**; visible on the partner site. |

### Social

| Command | What it does |
| --- | --- |
| `social` | Thread prospects (Reddit/X) with rules context. `--platform`, `--status`, `--age ranked\|new`. |
| `social-stats` | Thread pipeline counts. |
| `social-status <threadId> <status>` | Move a thread through the pipeline. |
| `social-draft-reply <threadId>` | AI-draft a reply for review. `--mention none\|natural\|founderOpen`. Nothing posts. |

### Brand

| Command | What it does |
| --- | --- |
| `brand` | Brand profile, products, and buyer profiles in one read. |
| `brand-set` | Update identity: `--name`, `--industry`, `--description`, or `--json` for authors (EEAT bylines) and styleReferenceUrls (arrays replace the stored list). |
| `brand-profile-set` | Patch voice and style fields. `--json '{"tone":"..."}'` or `--json @file.json`. |
| `brand-product` | Create, update, or delete a product via `--json` (create: name, description, category; update: productId; delete: productId plus `"del": true`). |
| `brand-icp` | Same semantics for buyer profiles (create: title, industry, demographics). |

### Team

| Command | What it does |
| --- | --- |
| `team` | Members and pending invites with per-screen permissions. |
| `team-invite <email>` | Invite a member. **Dry run**; `--perms content=write,analytics=read`. |
| `team-revoke <invitationId>` | Revoke a pending invite. |
| `team-remove <userId>` | Remove a member. Requires `--confirm`. |

### Analytics setup and integrations

Setup-time commands; most workspaces touch these once.

| Command | What it does |
| --- | --- |
| `analytics-sources` | Which source writes human-traffic and AI-crawler data. |
| `analytics-set-source <dataType> <source>` | Select the writer: `humanTraffic`/`aiCrawlers` x `embed`/`wordpress_plugin`/`cloudflare`/`none`. |
| `analytics-activate <screen>` | One-time activation of the Analytics or Reports screen. |
| `cloudflare-connect` | OAuth URL for read-only Cloudflare crawler analytics (human opens it). |
| `cloudflare-zones` / `cloudflare-zone <id> <name>` | List zones on the grant; pick the one to poll. |
| `framer-install-embed` | Install visit tracking on the Framer project. **Dry run**; installing also publishes the site. |
| `webflow-custom-code` | Apply the tracking loader via Webflow Custom Code. **Dry run**. |
| `shopify-install-url <shop>` | Mint an install link for the Shopify app (human approves in browser). |

### Support

| Command | What it does |
| --- | --- |
| `support <subject>` | Message the RankControl team. `--message <text>` required, `--page <url>` optional. Capped 3/min. |

## Worked examples

**Morning visibility check, then propose content for the gaps:**

```bash
rankcontrol score
rankcontrol citations --limit 20
rankcontrol ideas | jq '.[:5]'
rankcontrol plan-content --citation-gaps
# show the proposed titles to the user; if they approve some:
echo '{"titles":[...approved subset...]}' | rankcontrol commit-titles
```

**Publish flow (the human decides at each paid or public step):**

```bash
# publishable = body written but not live yet
rankcontrol content | jq '[.[] | select(.status=="draft")][0]'
rankcontrol publish <contentId>            # dry run: previews what would go live
# report the dry-run result; only after an explicit yes:
rankcontrol publish <contentId> --confirm
```

Dry-run responses come back as `{"dryRun":true,...}` with a hint naming both
forms: `confirm: true` for the API and MCP, `--confirm` for the CLI.

**Repurpose a published article to social drafts:**

```bash
rankcontrol repurpose
rankcontrol repurpose-generate <contentId> --platforms linkedin,x
rankcontrol repurpose <contentId>          # read the drafts back
# edits, then push only where the user wants it:
rankcontrol repurpose-push <draftId> --channels <id> --when draft
```

## Troubleshooting

| Symptom | Meaning |
| --- | --- |
| `HTTP 401` | No valid key. Run `rankcontrol login` or set `RANKCONTROL_API_KEY`. |
| `HTTP 403` | Key lacks the scope for this command. Tell the user which command failed; scopes are edited in Dashboard -> Settings -> API Keys. |
| `HTTP 429` | Rate limit. Wait a minute; slow the loop. |
| `{"dryRun":true,...}` | Not an error. The action is previewed and needs `--confirm`. |
