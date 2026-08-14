# news-watch-agent

[![Repo: public](https://img.shields.io/badge/repo-public-2ea44f?style=flat-square)](https://github.com/thoutmose/news-watch-agent)
[![Delivers to Discord](https://img.shields.io/badge/delivers%20to-Discord-5865F2?style=flat-square&logo=discord&logoColor=white)](https://discord.com)
[![Claude Code skill](https://img.shields.io/badge/claude%20code-skill-D97757?style=flat-square)](https://code.claude.com/docs/en/skills)
[![Built by Claude](https://img.shields.io/badge/built%20by-Claude-D97757?style=flat-square)](https://claude.com/claude-code)

Daily headlines digest + weekly synthesized deep-dive across **economy, health, industry, weather, science, AI** for **USA, Europe, China, Japan, Taiwan**. Third sibling in the `*-watch` family alongside [tech-watch-agent](https://github.com/thoutmose/tech-watch-agent) (professional/tech) and [culture-watch-agent](https://github.com/thoutmose/culture-watch-agent) (personal culture/learning) — this one is macro/current-events news, which fits neither of those domains.

> **Built entirely by agentic AI**, same as its siblings — designed, written, and operated by Claude, working interactively with the project owner.

## Why headlines and deep-dives are separate pipelines

5 regions × 6 categories = 30 combinations. Running all 30 with deep synthesis every day doesn't fit inside a shared, capped WebSearch budget (this is the exact lesson tech-watch learned the hard way covering 46 topics daily — see its README's "Known limitations"). So the split mirrors tech-watch's own daily-hot-list/weekly-full-sweep design directly:

- **Daily**: `headline-scanner` — cheap, wide, verified headlines (3-5 bullets per category, source+link, no synthesis). All 30 combinations, every day, deliberately kept lightweight enough to actually afford that.
- **Weekly**: `news-analyst` — the expensive step. One synthesized deep-dive per region/category: the week's real throughline, not a longer bullet list, grounded in specific cited reporting.

Both agents share the same non-negotiable rule: every headline must trace to a real, dated article from a real outlet, confirmed in-window before being reported — misreporting news is a worse failure mode than a weak recommendation.

## Repositories

| Repo | Visibility | Role |
| --- | --- | --- |
| **[news-watch-agent](https://github.com/thoutmose/news-watch-agent)** (this repo) | Public | The `/news-watch` skill and its two sub-agents. |
| **[news-watch-config](https://github.com/thoutmose/news-watch-config)** | Private | `regions.md` (regions + categories), `discord-threads.json` (one thread per region). |

## Running it

```
/news-watch                        # headlines, every region, all categories
/news-watch headlines "Taiwan"      # headlines, one region
/news-watch deep-dive               # weekly synthesized deep-dive, every region
/news-watch deep-dive "China"       # deep-dive, one region
```

See [`SKILL.md`](.claude/skills/news-watch/SKILL.md) for exact behavior. On-demand only for now — no cloud routine yet (see Known limitations).

## Known limitations

- **No cloud routine yet.** Same starting point culture-watch had before `culture-watch-routine` existed. When one gets built, it should follow the now-proven pattern from both siblings: live-clone `news-watch-config`, run in sequential waves, authenticate to Discord via a **webhook** rather than an embedded bot token — Claude Code's safety classifier specifically refused embedding a Discord bot token directly into a routine's stored prompt when this was tried for culture-watch-routine (three attempts, three different ways); a webhook sidesteps that entirely and is what `tech-watch-routine` already uses successfully.
- **One thread per region, not per category** — 30 threads would be unreadable. Categories are sections within a region's post instead.
- **No Discord-native commands yet** (unlike tech-watch-cli-bot/culture-cli's `topic`/`log`) — `regions.md` is hand-edited. Not a priority until the on-demand pipeline itself is proven out.
