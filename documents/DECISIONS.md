# DECISIONS.md

> Append-only log of key decisions and the reasoning behind them.
> Don't edit old entries. If a decision is reversed, add a new entry referencing the old one.
> This prevents re-litigating the same questions and helps future-you (and AI collaborators)
> understand not just *what* the project looks like but *why*.

Format:
```
## YYYY-MM-DD: Short title
**Decision:** What we decided.
**Context:** What prompted this decision.
**Reasoning:** Why this option over the alternatives.
**Alternatives considered:** What else we looked at.
**Revisit when:** What signal would make us reconsider.
```

---

## 2026-05-04: v0 will support Spotify and Apple Music only

**Decision:** v0 supports only Spotify and Apple Music. No Tidal, Deezer, or YouTube Music.

**Context:** The friend group spans multiple platforms. Tempting to support them all from day one for "completeness."

**Reasoning:** Apple Music + Spotify covers ~70% of US streaming users and our friend group's primary platforms. Adding more platforms multiplies integration work, edge cases, and track resolution complexity. v0 is about proving the core sync thesis, not maximizing coverage.

**Alternatives considered:** Supporting all five platforms; supporting Spotify only.

**Revisit when:** v0 is validated and at least one v0 user actively requests another platform.

---

## 2026-05-04: YouTube Music is out of scope until further notice

**Decision:** Don't integrate YouTube Music in v0 or v1.

**Context:** Several friend group members or potential users may use YouTube Music.

**Reasoning:** No official API. The community library `ytmusicapi` works by scraping internal endpoints — fragile and could break anytime, with no recourse. Building on it for a production product is risky. Not worth the engineering effort or ongoing maintenance burden until we have committed users specifically requesting it.

**Alternatives considered:** Integrating via `ytmusicapi`; building our own scraper; partnering with YouTube (unrealistic at this stage).

**Revisit when:** We have paying users explicitly asking for YouTube Music, OR Google publishes an official playlist-management API.

---

## 2026-05-04: Polling-only sync, no webhooks/push

**Decision:** Detect playlist changes via periodic polling. Default 3-minute interval per mirror playlist.

**Context:** Real-time sync would be a nicer UX.

**Reasoning:** No streaming API offers webhooks for playlist changes. Polling is the only option. 3 minutes is a reasonable balance between freshness and API quota usage at v0 scale. Adaptive polling (slower for dormant playlists) keeps it manageable as we grow.

**Alternatives considered:** Polling more frequently (1 min) — too API-intensive; polling less frequently (10+ min) — too laggy for party use cases.

**Revisit when:** Polling becomes a cost or rate-limit problem at scale, OR we add an in-app song-add feature (which would bypass polling for those additions).

---

## 2026-05-04: Defer mobile apps until v1+

**Decision:** v0 is web-only. No iOS or Android apps.

**Context:** Party/group use cases are mobile-native. Mobile would arguably be the better v0 form factor.

**Reasoning:** Native apps add 2-3x the engineering effort, deployment complexity, and review cycles. v0 is about validating the core sync engine, not the form factor. The friend group can use a mobile-friendly web UI to monitor sync status; actual song-adding still happens in their native streaming apps in v0.

**Alternatives considered:** React Native or Flutter for cross-platform mobile; iOS-only native.

**Revisit when:** v0 sync is validated and we want to add in-app song adding or party mode — those features need mobile to feel right.

---

## 2026-05-04: Skip playlist reordering sync in v0

**Decision:** Sync adds and removes only. If someone reorders tracks in their mirror, those reorderings don't propagate.

**Context:** Reordering is a legitimate playlist operation users might expect to sync.

**Reasoning:** Reordering is significantly more complex than add/remove (conflict resolution between concurrent reorders, position tracking, partial syncs that look like reorders). Most party/group playlists don't depend on order — the value is the set of tracks. Cutting this dramatically simplifies v0.

**Alternatives considered:** Full reordering sync; one-way reordering (only the canonical order propagates out).

**Revisit when:** Multiple users complain about ordering inconsistency, OR we add party mode where order matters.

---

## 2026-05-04: Sync existing playlists rather than creating new ones in-app

**Decision:** Each user picks an *existing* playlist on their platform to act as their mirror. Our app doesn't create playlists.

**Context:** Could go either way — we could create the playlist ourselves to control it.

**Reasoning:** Letting users pick an existing playlist means they can use whatever name, art, and conventions they're comfortable with on their native platform. Less friction. Also avoids edge cases where our created playlist gets deleted or renamed by the user.

**Alternatives considered:** Creating playlists via our app with a standardized name; hybrid approach where user picks but we rename.

**Revisit when:** Users report confusion about which playlist is the synced one, OR we add features that need consistent playlist naming.

---

## 2026-05-04: Track resolution uses ISRC first, fuzzy fallback

**Decision:** Match tracks across platforms by ISRC when available; fall back to fuzzy metadata matching (title + artist + duration) with confidence scoring.

**Context:** Different platforms use different track IDs. Need a strategy to identify the same song across platforms.

**Reasoning:** ISRC is a definitive recording identifier and catches 70-80% of cases cleanly. Fuzzy matching with explicit confidence scoring handles the rest while making failures debuggable. Caching all resolved mappings means each track only needs to be resolved once across the system's lifetime.

**Alternatives considered:** Fuzzy-only (less reliable); third-party services like 7digital or Gracenote (cost money, not justified at v0 scale); MusicBrainz lookups (good supplement, possibly add later).

**Revisit when:** Resolution accuracy drops below 90%, OR we discover MusicBrainz / a third-party meaningfully improves match quality at acceptable cost.

---

## 2026-05-04: Stack choice — Node.js + TypeScript + Postgres

**Decision:** Backend in Node.js + TypeScript with Hono or Fastify. Postgres via Supabase or Neon. Redis via Upstash. Hosted on Railway or Render.

**Context:** Need to pick a stack to actually start building.

**Reasoning:** TypeScript gives type safety for the multi-platform adapter pattern, which is critical for keeping the sync logic correct. Postgres is the right call for relational data with JSON columns where useful. Free tiers across the board cover v0 scale. _[Note: confirm or change based on your actual preferences.]_

**Alternatives considered:** Python + FastAPI; Go; Ruby on Rails. All are viable; choice is largely about familiarity and ecosystem fit.

**Revisit when:** Performance or scaling needs outgrow the stack (unlikely for years), OR a different stack offers a meaningfully better library for one of the platform APIs.
