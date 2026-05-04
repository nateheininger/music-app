# PROJECT.md

> The north star. What we're building, who it's for, and what's in/out of scope.
> This doc changes rarely. Paste it into new AI conversations to re-ground context.

## What this is

A cross-platform playlist sync service that lets a group of friends collaborate on a single shared playlist, even when each person uses a different music streaming platform (Spotify, Apple Music, etc.).

When any group member adds a song to their version of the playlist on their native platform, that change propagates to everyone else's version on their respective platforms within a few minutes.

## The problem we're solving

Group playlists for parties, road trips, and shared events break down when friends are on different streaming services. Existing cross-platform tools (Soundiiz, TuneMyMusic, FreeYourMusic) focus on one-time migration, not ongoing collaboration. There's no good product built around persistent, shared, multi-platform group playlists.

## Who it's for

Primary user: a small friend group (3-8 people) where members are split across at least two streaming platforms and want to collaborate on shared playlists for events or ongoing use cases (parties, road trips, workouts, etc.).

Not for: solo users migrating between platforms (existing tools handle this), or large public collaborative playlists.

## v0 scope (the only thing that matters right now)

**Mission:** Four friends share one playlist across Spotify and Apple Music. Songs added on either platform appear on the other within 5 minutes.

**In scope for v0:**
- Spotify and Apple Music only
- One group, hardcoded membership (you and your friends)
- Sync existing playlists (don't create new ones through our app)
- Add and remove track sync (not reordering)
- Polling-based sync (no real-time)
- Minimal web UI for connecting accounts and viewing sync status
- Track resolution via ISRC + fuzzy fallback

**Explicitly out of scope for v0:**
- Mobile apps
- Tidal, Deezer, YouTube Music
- Real-time / party mode / in-app song adding
- Voting, reactions, social features
- Multiple groups per user
- Playlist creation through our app
- Playlist reordering sync
- Beautiful onboarding or marketing site
- Auth for non-friend-group users
- Billing or subscriptions
- Admin dashboard or analytics

## Success criteria for v0

We've succeeded when all four are true:
1. The friend group uses it for at least one real event without me babysitting it.
2. It runs for 30 days without manual intervention (no manual restarts, graceful re-auth).
3. Sync accuracy is >90% on first attempt (track shows up on other platforms 9/10 times within polling interval).
4. At least one friend uses it without me nagging them to add songs.

If all four hit, v0 is validated and we earn the right to plan v1.

## Beyond v0 (directional, not committed)

If v0 works, the most interesting direction is the **group/social layer** — party mode, in-app song adding, voting, "who added this," persistent friend-group playlists. The plumbing (cross-platform sync) becomes the boring foundation; the social experience is the product.

If v0 fails, the failure mode tells us where to dig: sync reliability issues mean we go deeper on architecture; lack of friend usage means the product idea itself needs rethinking.

## Budget and constraints

- Cash spend cap for v0: **$500/year** (mostly Apple Developer fee at $99)
- Time budget for v0: **100-200 hours**
- Timeline: **3-5 months** at part-time pace
- Built solo (for now)
