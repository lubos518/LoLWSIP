# LoLWSIP — Champion-Select Analytics (Personal Prototype)

LoLWSIP is a private, non-commercial desktop companion prototype for League of Legends designed to explore champion-level performance during champion select. Focusing strictly on champions within a player's personal champion pool, it evaluates current-patch statistical metrics: role-specific win rates, lane matchups against revealed enemy picks, synergies with allied champions, and blind-pick safety. Every metric is presented alongside its sample size and statistical uncertainty so limited-sample estimates are always transparent.

> **Status:** Private personal prototype / research project. Not publicly distributed and not monetized.

## Riot API Usage

Ranked Solo/Duo match data for the current patch is collected exclusively by a single local backend crawler — never directly by any desktop client.

- **Data Dragon:** Used to detect the current live patch version.
- **League-V4:** Used (`league entries` by tier and division) to sample ranked players in the target brackets.
- **Match-V5:** Used (`match IDs` by PUUID, `match details`, and `match timelines`) to fetch completed ranked solo games.

### Data Minimization & Privacy

Only anonymized, champion-level statistics are stored in the match database (`crawl.sqlite`):
- Patch, region, seed rank tier, game length/day
- Per-champion data: champion, role, team, win/loss, runes, items, summoner spells, bans, team objectives, and select early-game timeline metrics (e.g., gold/XP at 10 and 14 minutes, early deaths, skill order, starting purchases)

**What is never stored with match data:**
- No Summoner names, Riot IDs, account IDs, or PUUIDs enter the match database.
- Match IDs are stored strictly as keyed HMAC digests (`hmac.key`) solely for deduplication.
- PUUIDs exist only in temporary local frontier queues (`frontier-*.sqlite`) used to track crawl progress and are deleted at the end of each patch (or immediately upon any deletion request).

## Architecture

1. **Local Crawler (Rust):** Runs on a single machine, strictly reads Riot's rate-limit response headers, and throttles itself to stay below 90% of all application and method limits (with a 120-second cold-start window on startup).
2. **Offline Aggregation:** Periodically aggregates the anonymized match dataset into a compact static statistical model (shrinkage estimates for champion strength, lane matchups, and team synergies by rank bracket).
3. **Desktop Companion (Prototype):** Reads only the precomputed static model locally. It contains no Riot API key, makes zero requests to the Riot API, and only reads local champion select state (picked, banned, and available champions) to display champion-level statistics.

## Riot Policy Compliance

- **Champion-level analytics only:** Never scouts, rates, or profiles teammates or opponents, and never displays player ranks, match histories, or individual win rates.
- **No de-anonymization:** Never identifies or reveals hidden players in champion select.
- **No automation:** Never automates picks, bans, runes, or any other client interaction.
- **Strict rate-limit adherence:** Single key, single crawler instance, header-driven throttling below Riot's limits.
- **Non-commercial personal use:** Built strictly for personal use and prototyping.

## Scope (Current Prototype)

- **Queue:** Ranked Solo/Duo
- **Regions:** EUW, NA, KR, SEA
- **Tiers:** Gold & Platinum (testing rank-bracket statistical modeling)

---

*LoLWSIP isn't endorsed by Riot Games and doesn't reflect the views or opinions of Riot Games or anyone officially involved in producing or managing Riot Games properties. Riot Games, and all associated properties are trademarks or registered trademarks of Riot Games, Inc.*
