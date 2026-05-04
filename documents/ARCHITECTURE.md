# ARCHITECTURE.md

> The technical map. Stack, components, data model, and how they fit together.
> Update when architectural decisions change. Keep it current — this is the doc
> that makes a fresh AI conversation immediately productive.

## Stack

- **Backend:** Node.js + TypeScript with Hono (or Fastify). _[Confirm or change]_
- **Database:** Postgres via Supabase or Neon (free tier)
- **Job queue:** BullMQ backed by Redis (Upstash free tier)
- **Hosting:** Railway or Render, single service
- **Frontend:** Next.js (or plain HTML + HTMX for v0 simplicity) _[Decide]_
- **Auth (our users):** Supabase Auth magic links, or hardcoded for v0
- **Auth (streaming platforms):** OAuth 2.0 (Spotify), MusicKit JS + JWT (Apple Music)

## High-level component diagram

```
┌─────────────┐         ┌──────────────────┐
│  Web Client │◄───────►│   Backend API    │
└─────────────┘         │   (REST/JSON)    │
                        └────────┬─────────┘
                                 │
                ┌────────────────┼────────────────┐
                ▼                ▼                ▼
        ┌──────────────┐  ┌────────────┐  ┌────────────────┐
        │  Postgres    │  │   Redis    │  │ Track          │
        │  (state)     │  │  (queue)   │  │ Resolution     │
        └──────────────┘  └─────┬──────┘  │ Service        │
                                │         └────────────────┘
                                ▼
                        ┌──────────────┐
                        │ Sync Workers │
                        └──────┬───────┘
                               │
                ┌──────────────┼──────────────┐
                ▼                             ▼
        ┌───────────────┐           ┌──────────────────┐
        │ Spotify       │           │ Apple Music      │
        │ Adapter       │           │ Adapter          │
        └───────┬───────┘           └─────────┬────────┘
                │                             │
                ▼                             ▼
        ┌───────────────┐           ┌──────────────────┐
        │ Spotify API   │           │ Apple Music API  │
        └───────────────┘           └──────────────────┘
```

## Components

### 1. Backend API server
Standard REST service. Responsibilities:
- User account management (just our handful of friends in v0)
- OAuth callback handlers for Spotify and Apple Music
- Group and playlist CRUD
- Serves the web UI's data needs
- Enqueues sync jobs

### 2. Postgres database
Source of truth for everything except music data itself. See data model below.

### 3. Redis + job queue
Drives the sync workers. Recurring jobs poll connected playlists; on-demand jobs propagate detected changes.

### 4. Sync workers
Background processes that:
- Poll each connected mirror playlist on a schedule
- Diff polled state against last-known state stored in DB
- On detected changes, resolve tracks across platforms and propagate to other mirrors
- Handle rate limits, token refresh, and transient failures gracefully

### 5. Track resolution service
Module (not separate service in v0 — just a clean interface) that handles:
- Mapping a platform-specific track ID to our canonical track
- Finding the equivalent track on other platforms (ISRC first, fuzzy match fallback)
- Caching mappings forever once resolved
- Confidence scoring for matches

### 6. Streaming platform adapters
One module per platform, behind a common interface:
```typescript
interface StreamingAdapter {
  getPlaylist(playlistId, accessToken): Promise<Playlist>
  addTracks(playlistId, trackIds, accessToken): Promise<void>
  removeTracks(playlistId, trackIds, accessToken): Promise<void>
  searchTrack(query, accessToken): Promise<TrackMatch[]>
  refreshToken(refreshToken): Promise<TokenSet>
}
```

This isolation means platform weirdness stays contained. Adding Tidal in v1 = writing one new adapter.

### 7. Web client
Minimal. v0 needs:
- Connect Spotify button (OAuth flow)
- Connect Apple Music button (MusicKit flow)
- Pick which of your playlists is the group mirror
- Show sync status (last synced, recent changes, any errors)

No design system, no marketing. Tailwind defaults. Internal-only.

## Data model (rough v0 schema)

```sql
-- Our users (the friend group)
users (
  id, email, display_name, created_at
)

-- A user's connected streaming account
streaming_accounts (
  id, user_id, platform ('spotify'|'apple_music'),
  platform_user_id, access_token_encrypted,
  refresh_token_encrypted, expires_at, created_at
)

-- The group(s) — v0 has one
groups (
  id, name, created_at
)

group_members (
  group_id, user_id, joined_at
)

-- The canonical playlist (source of truth)
canonical_playlists (
  id, group_id, name, created_at
)

-- Tracks in canonical playlist, ordered
canonical_playlist_tracks (
  canonical_playlist_id, canonical_track_id,
  position, added_by_user_id, added_at
)

-- Our internal track representation
canonical_tracks (
  id, isrc, title, primary_artist,
  album, duration_ms, created_at
)

-- Cross-platform track ID mappings
track_platform_mappings (
  canonical_track_id, platform, platform_track_id,
  match_confidence, match_method ('isrc'|'fuzzy'|'manual'),
  created_at
)

-- A user's mirror playlist on a specific platform
mirror_playlists (
  id, user_id, streaming_account_id,
  canonical_playlist_id, platform, platform_playlist_id,
  last_synced_at, last_known_track_ids (jsonb),
  created_at
)

-- Sync events (for debugging and audit)
sync_events (
  id, mirror_playlist_id, event_type ('add'|'remove'|'poll'|'error'),
  details (jsonb), created_at
)

-- Tracks we couldn't resolve (for review)
unresolved_tracks (
  id, source_platform, source_track_id,
  source_metadata (jsonb), reason, created_at
)
```

## Sync flow (the core algorithm)

When a track is added on Apple Music:

1. Sync worker polls user's Apple Music mirror playlist.
2. Diff against `mirror_playlists.last_known_track_ids` reveals new track.
3. Look up Apple Music track ID in `track_platform_mappings`.
   - If found: use existing `canonical_track_id`.
   - If not found: fetch metadata, create `canonical_tracks` row, attempt to resolve to other platforms, populate `track_platform_mappings`.
4. Append to `canonical_playlist_tracks`.
5. Update `mirror_playlists.last_known_track_ids`.
6. Enqueue propagation jobs for each *other* mirror in the same group.
7. Each propagation job: fetch canonical state, diff against its own `last_known_track_ids`, call adapter's `addTracks` for the missing track using the platform-specific ID.

Removals work the same way in reverse.

## Track resolution algorithm

Layered approach, highest confidence first:

1. **ISRC exact match.** If the source track has an ISRC and we can find a track with the same ISRC on the target platform via search, that's a definitive match.
2. **Fuzzy metadata match.** Normalize title (lowercase, strip punctuation, remove "feat. X" parentheticals). Compare title + primary artist + duration (within ~2 seconds). Score 0-100 using Levenshtein distance.
3. **Confidence thresholds:**
   - ≥90: auto-add
   - 70-89: auto-add but flag in sync_events for review
   - <70: don't add, log to `unresolved_tracks`, show in UI for manual resolution
4. **Cache forever.** Once a mapping is established at ≥70 confidence, it's permanent. Over time we build up a proprietary cross-platform music graph.

## Polling strategy

- Default poll interval: **3 minutes** per mirror playlist
- Adaptive: playlist not modified in 24h drops to 15-minute interval; 7+ days drops to hourly
- Active hint: when a user opens the web UI, immediately enqueue polls for their mirrors
- Backoff: on rate limit errors, exponential backoff up to 1 hour; on auth errors, mark mirror as "needs reauth" and stop polling

## Token handling

- Access tokens stored encrypted at rest (use pgcrypto or app-level encryption with a key in env vars)
- Refresh tokens stored encrypted with stricter access controls
- Sync workers refresh access tokens automatically when they expire
- If refresh fails (user revoked access), mark account as disconnected and notify user via email

## What's deliberately simple in v0

- No multi-tenancy (one group, one app instance)
- No real-time anything (polling only)
- No mobile (web only)
- No reordering sync (just adds/removes)
- No conflict resolution UI (last-write-wins; revisit if it becomes a real problem)
- Tracks that fail to resolve just don't sync; user sees them in the UI

## Known risks and mitigations

| Risk | Mitigation |
|---|---|
| Apple MusicKit JS is finicky | Budget extra time; start integration early |
| Track resolution is imperfect | Surface failures clearly in UI; don't hide them |
| Spotify rate limits at scale | Won't matter at v0 scale; revisit if ever public |
| Refresh tokens silently expire | Explicit "needs reauth" state and user notification |
| YouTube Music has no official API | Out of scope for v0; revisit later |
