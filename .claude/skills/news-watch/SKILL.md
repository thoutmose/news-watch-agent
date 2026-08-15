---
name: news-watch
description: Daily headlines digest and weekly synthesized deep-dive across economy/health/industry/weather/science/AI for USA, Europe, China, Japan, and Taiwan. Use for "/news-watch", "/news-watch headlines", "/news-watch deep-dive", "check the news for X".
---

# News Watch

Two modes, one skill, two sub-agents. Sibling project to `tech-watch-agent`/`culture-watch-agent`, same repo split, same "don't fabricate — verify before reporting" discipline, applied to current-events news instead of tech releases or culture recommendations.

Data lives in the private `thoutmose/news-watch-config` repo (regions, categories, Discord thread map) — always sync it live first. Reports get written locally to `~/Developer/news-watch-agent/reports/` (gitignored, disposable — Discord/chat is the durable record).

## 0. Sync config

```bash
if [ -d ~/Developer/news-watch-config/.git ]; then
  git -C ~/Developer/news-watch-config pull --ff-only
else
  git clone https://github.com/thoutmose/news-watch-config ~/Developer/news-watch-config
fi
```

Read `~/Developer/news-watch-config/regions.md`. `##` headers are regions; a `Categories:` line beneath each (comma-separated) lists that region's categories — defaults to economy, health, industry, weather, science, AI if the line is absent (see the file's own leading comment).

## 1. Parse the invocation

- `/news-watch` or `/news-watch headlines` → **headlines mode**, every region, trailing 1 day.
- `/news-watch deep-dive` → **deep-dive mode**, every region, trailing 7 days.
- `/news-watch headlines "<Region>"` / `/news-watch deep-dive "<Region>"` → scoped to one region (case-insensitive substring match against `regions.md` headers; ambiguous or no match → tell the user and list the real region names, don't guess).
- Resolve the period with Bash (`date`), never guess a relative phrase — same convention as tech-watch/culture-watch.

## 2. Research in parallel waves

For each (region, category) pair in scope, dispatch one `Agent` call:

- **headlines mode** → `subagent_type: headline-scanner`, passing region, category, the resolved 1-day period, today's date.
- **deep-dive mode** → `subagent_type: news-analyst`, passing region, category, the resolved 7-day period, today's date.

Launch in waves of at most 5 pairs at a time (launch a wave's calls together in one message, wait for it, then the next) rather than all ~30 at once — the same shared-WebSearch-budget lesson learned running tech-watch-routine applies here. A single-region invocation (≤6 pairs) can run in one wave.

## 3. Compile one report per region

`~/Developer/news-watch-agent/reports/<region-slug>/<headlines|deep-dive>-<YYYY-MM-DD>.md`:

```markdown
# <Region> — <Headlines | Deep Dive> (<period label>)
_Generated <YYYY-MM-DD>, covering <date range>_

## <Category 1>
<that category's agent output verbatim>

## <Category 2>
...
```

One section per category that ran, in the order listed in `regions.md`. If a category's agent found nothing verified, its section says so plainly (from the agent's own "No verified headlines..." line) rather than being omitted — a silent gap and a checked-and-quiet category mean different things.

## 3.5. Also append to the searchable archive

For every individual item actually kept in step 3 (not the "nothing verified" placeholders), append one JSON line to `~/Developer/news-watch-config/archive.jsonl` (create it if missing) shaped exactly:

```json
{"date": "YYYY-MM-DD", "region": "<Region>", "category": "<Category>", "headline": "<title>", "source": "<outlet>", "link": "<url>", "summary": "<1-2 sentences, under 300 chars>"}
```

Never rewrite existing lines — append only. This is what `#news-cli`'s `find` command searches (Discord's bot REST API has no general message-search endpoint, so this project keeps its own flat index rather than trying to query Discord for it). Push it back once done:

```bash
cd ~/Developer/news-watch-config && git add archive.jsonl && git commit -m "News watch: archive updates — <date>" && git push
```

## 4. Post to Discord

One post per region, into that region's thread (from `discord-threads.json`) — create the thread first if it doesn't have one yet (name it after the region). Post the full compiled report, splitting at Discord's ~2000-char limit between category sections (never mid-sentence). This project doesn't have a bot-token REST script the way culture-watch does yet — post via `curl` directly against the Discord REST API using a bot token if one is configured in the environment, or note in your final message that Discord posting needs to be wired before it can happen, and present the results in chat regardless (step 5 always happens either way).

**Use `curl` specifically, not a scripting language's HTTP client.** Confirmed live, not just inherited from the sibling projects' docs: a first attempt at this used Python's `urllib` and got a flat `403 Forbidden` from Discord's edge (Cloudflare fingerprint-blocks non-browser-like clients) even with a valid, correctly-authenticated request — switching the exact same request to `curl` worked immediately. Don't rediscover this the hard way twice.

## 5. Present the result directly

Show each region's digest/deep-dive in chat — headlines mode can be compact (category headers + the bullet list), deep-dive mode should show the full abstract + supporting items. Note the report file paths and whether Discord posting happened. Don't just say "done, check Discord."

## Notes for the future

`news-watch-routine` (daily headlines at 07:00 UTC, Monday = weekly deep-dive instead) live-clones `news-watch-config`, runs in sequential waves respecting the shared WebSearch budget, and posts via a Discord webhook rather than an embedded bot token (embedding the bot token directly was refused by Claude Code's own safety classifier when attempted for culture-watch-routine — a webhook sidesteps this and is the pattern all three `*-watch-routine`s now use). It can also be fired on-demand via `#news-cli`'s `run headlines`/`run deep-dive` commands, same `Mode:`/`Region:` fire-payload convention as its siblings.
