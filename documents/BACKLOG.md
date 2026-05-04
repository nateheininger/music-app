# BACKLOG.md

> What's actively next, and what's been deferred. Keep "Deferred" generous — every
> "ooh that would be cool" idea goes here, not into the current build. This is the
> dam that protects v0 from scope creep.

## Now (actively working on)

_Nothing yet — project is in planning phase._

---

## Next up (queued for v0, in rough order)

### Phase 1: Spotify-only proof of concept (week 1-2)
- [ ] Set up repo, basic backend skeleton, deploy a "hello world" to Railway/Render
- [ ] Implement Spotify OAuth flow end-to-end
- [ ] Read a Spotify playlist via API and log it
- [ ] Set up Postgres schema (minimal: users, streaming_accounts)
- [ ] Build a poll-and-log loop that detects when *I* add a song to a Spotify playlist

### Phase 2: Persistence + diffing (week 3-4)
- [ ] Full database schema per ARCHITECTURE.md
- [ ] Implement diff logic (compare current playlist state to last known)
- [ ] Set up BullMQ job queue with Redis
- [ ] Convert poll loop into recurring background job
- [ ] Write add/remove events to `sync_events` table

### Phase 3: Apple Music integration (week 5-7)
- [ ] Apple Developer account setup, MusicKit JWT generation
- [ ] MusicKit JS in the web client for user auth
- [ ] Apple Music adapter implementing the common interface
- [ ] Parity with Spotify: read playlist, poll, detect changes

### Phase 4: Cross-platform track resolution (week 8-9)
- [ ] ISRC-based matching
- [ ] Fuzzy fallback (Levenshtein on normalized title + artist match + duration window)
- [ ] Confidence scoring + thresholds
- [ ] `track_platform_mappings` table populated automatically
- [ ] `unresolved_tracks` table populated for failures
- [ ] Manual resolution UI (low-fi)

### Phase 5: Multi-user + sync propagation (week 10-11)
- [ ] Group + group_members concept
- [ ] Canonical playlist as source of truth
- [ ] On detected change in mirror A, propagate to all other mirrors in same group
- [ ] Add the friend group's accounts
- [ ] First end-to-end: I add song on Apple Music, friend sees it on Spotify within 5 min

### Phase 6: Real-world hardening (week 12+)
- [ ] Refresh token handling and re-auth flow
- [ ] Error handling, retry with backoff, dead letter queue
- [ ] Sync status UI (last synced, recent events, errors)
- [ ] Use it for an actual party/event
- [ ] Patch everything that breaks

---

## Deferred (post-v0, in no particular order)

### Platform expansion
- [ ] Tidal integration
- [ ] Deezer integration
- [ ] YouTube Music integration (only if API situation improves or there's strong demand)
- [ ] Amazon Music (low priority — small share)
- [ ] SoundCloud (different use case but possibly relevant)

### Form factor
- [ ] iOS native app
- [ ] Android native app
- [ ] React Native cross-platform mobile
- [ ] PWA improvements (offline view, installable)

### Social / group features
- [ ] In-app song adding (bypasses polling, instant sync)
- [ ] Party mode (real-time party view, anyone can add)
- [ ] Voting / upvoting on tracks
- [ ] "Who added this" attribution displayed everywhere
- [ ] Reactions / comments on tracks
- [ ] Multiple groups per user
- [ ] Public/discoverable group playlists
- [ ] Time-bounded event playlists (auto-archives after a date)
- [ ] Now playing view that any host can drive

### Sync improvements
- [ ] Playlist reordering sync
- [ ] Conflict resolution UI for ambiguous changes
- [ ] Real-time sync via active polling boost when users are online
- [ ] Webhooks if any platform ever supports them
- [ ] Smarter adaptive polling (ML-based prediction of when playlists change)
- [ ] Bulk import (sync an entire library, not just one playlist)

### Track resolution improvements
- [ ] MusicBrainz integration for richer metadata
- [ ] Manual override UI for incorrect matches
- [ ] User-reported bad matches feedback loop
- [ ] Region/availability handling (track exists on platform X but not in user's region)
- [ ] Explicit vs. clean version detection and matching
- [ ] Live vs. studio version detection

### Product / business
- [ ] Marketing site
- [ ] Pricing page
- [ ] Subscription billing (Stripe)
- [ ] Free tier limits
- [ ] Onboarding flow for non-technical users
- [ ] Email notifications (sync errors, weekly digests)
- [ ] Referral / invite system
- [ ] Analytics on usage

### Operational
- [ ] Admin dashboard
- [ ] Customer support ticketing
- [ ] Audit logs visible to users
- [ ] Data export (your playlists, your way)
- [ ] Account deletion + data purge
- [ ] GDPR / CCPA compliance review

### Wild ideas to explore later
- [ ] Cross-platform "music graph" as an API offering for other developers
- [ ] AI-suggested tracks based on group's collective taste
- [ ] DJ mode (auto-mix between tracks for parties)
- [ ] Spotify Jam / Apple Music SharePlay integration
- [ ] Smart speaker integration (queue from canonical playlist)
- [ ] Calendar integration (auto-create event playlists)

---

## Rejected (considered and explicitly chose not to do)

_Empty — populate as ideas get explicitly rejected with reasoning logged in DECISIONS.md._
