# OPEN_QUESTIONS.md

> Things we haven't figured out yet. When we resolve one, log the answer in
> DECISIONS.md and remove it from here. New questions get added as they come up.
> Keeping this list visible prevents you from forgetting open issues.

## Architectural / technical

### How do we handle a user disconnecting their streaming account mid-sync?
If a user revokes our access to their Spotify account, we should detect it on the next sync attempt (refresh token failure). What happens to their mirror playlist? Do we mark it as orphaned? Do other group members keep syncing without them? Probably yes, but need to define the exact state machine.

### What's the right behavior when the same track is added to multiple mirrors near-simultaneously?
User A adds track X on Spotify at 12:00:00. User B adds the same track X on Apple Music at 12:00:30. Both polls happen at 12:01:00. Both see the addition as new. Without dedup, we'd double-add to canonical and propagate the duplicate. Need to dedup by canonical_track_id at the canonical layer.

### Should the canonical playlist allow duplicates at all?
If User A adds track X, then a week later User B genuinely wants to add it again (some playlists do this for emphasis/looping), should we allow it? Probably allow but warn. Need to decide.

### How do we handle tracks that exist on one platform but not another?
A regional release might be on Spotify but not Apple Music in the user's country. Currently we'd log it to `unresolved_tracks` and skip propagation. Is that the right call, or should we substitute the closest match?

### What's our backup / recovery strategy if Postgres dies?
Daily snapshots are probably enough at v0 scale, but we should be explicit about RPO/RTO even if the answer is "lol whatever, restore from yesterday."

### How are encryption keys for tokens managed?
Storing encrypted refresh tokens is good, but where does the encryption key live? Env var on the host? Secret manager? For v0, env var is probably fine but worth being intentional.

### Do we need any kind of audit trail beyond `sync_events`?
For debugging, sync_events is great. For "user wants to know why their playlist looks the way it does," we may need richer history.

## Product / UX

### How do users discover and join groups in v1+?
For v0 we hardcode the friend group. For v1, the question of "how does Person A invite Person B to their group" matters a lot. Email invite? Shareable link? Code? Probably shareable link.

### When sync fails for a specific track, how do we communicate that to the user?
A Slack-style ephemeral notification? An email? A "needs attention" badge in the web UI? Probably the badge is enough for v0.

### Should there be a "this is the source of truth" platform per group?
Right now the canonical playlist is virtual — no platform owns it. But in practice, do groups want to designate one platform's version as primary (e.g., for naming, art, ordering)? Worth asking the friend group during v0 testing.

### What happens if the friend group disagrees on a track? (someone removes what someone else added)
Right now: last write wins, removal propagates. But socially this could be a flashpoint. Do we surface "X was removed by Y" so it's not invisible? Probably yes.

### Should we show what each person is currently listening to, separately from the playlist?
Tangential but potentially fun. Definitely not v0. Maybe v1 social layer.

## Business / strategic

### What's the right pricing model if this becomes a real product?
Per-user subscription? Per-group? Freemium with limits on group size or sync frequency? Soundiiz charges per-user (~$4.50/mo). Worth user research before deciding.

### How do we acquire users beyond the initial friend group?
Group products are inherently viral but only if we engineer for that. Need a strategy for the first 100 → 1,000 → 10,000 user transitions if v0 validates.

### What's our defensibility if Spotify or Apple builds this themselves?
Spotify Jam already does Spotify-only group listening. If they extend to cross-platform (unlikely but possible), we're toast. Apple is even less likely to. Our defensibility is probably (a) being multi-platform from day one, (b) social features they wouldn't prioritize, and (c) network effects within friend groups.

### Should we eventually be a B2B offering?
Wedding DJs, party planners, event venues all face this exact problem. Could be a more profitable niche than consumer.

## Personal / process

### How do I want to balance this with TAB and other commitments?
Realistic time per week. Worth being honest about so v0 doesn't drag on for 12 months and lose momentum.

### When do I bring on a co-founder or contractor, if ever?
Probably not until v0 is validated. But worth thinking about what skills would be most valuable to add — likely mobile dev or design.

### What's my actual exit / success scenario for this?
Lifestyle business doing $X/year? Sell it after Y users? Run it forever as a side project? Different answers should drive different decisions early.
