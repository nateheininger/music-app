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

---

## 2026-05-08: Hosting on Render, database on Supabase

**Decision:** Host the app on Render (separate web service + background worker service). Use Supabase for Postgres, Auth, and Vault (for OAuth refresh token storage). Drop Railway and Neon as alternatives previously listed in ARCHITECTURE.md.

**Context:** The original stack decision (2026-05-04) listed "Railway or Render" and "Supabase or Neon" without committing. Time to commit. User has prior experience with Railway and Supabase but is open to better options.

**Reasoning:**
- *Worker-first architecture fits Render better.* The app is fundamentally background-worker-heavy: sync workers polling Spotify and Apple Music every few minutes, plus eventual cron-like adaptive polling and propagation jobs. Render treats background workers and cron jobs as first-class service types. Railway has no dedicated worker service type, so workers tend to get shoehorned into web service processes or cron hacks.
- *Predictable pricing at the shape we'll grow into.* Once we have web + worker + (eventually) cron always-on, Render's flat per-service pricing (~$7/service starter) is easier to budget than Railway's usage-based model for steady workloads.
- *Reliability matters for success criterion #2.* Project goal is "runs for 30 days without manual intervention." Railway has had a recurring pattern of platform-level outages and free-tier deploy restrictions during peak hours. Render has a more boring, steady track record — which is what we want.
- *Supabase over Neon for security primitives.* Supabase Vault gives encrypted-at-rest secret storage with a managed key, which is the right tool for OAuth refresh tokens (long-lived, high-value secrets). Supabase Auth + RLS provide defense-in-depth at the database layer if we adopt it later. Neon is pure Postgres with no auth or secret management — would force us to bolt on Clerk/Auth0 separately and never get DB-layer security primitives. Both are SOC 2 Type II compliant.
- *Familiarity tradeoff is acceptable.* User has used Railway and Supabase before but is fine learning Render. Solo project hours are real, but the cost of fighting the wrong platform later compounds worse than the cost of learning a new dashboard now.

**Alternatives considered:**
- *Railway for hosting:* Better DX, faster initial deploys, project canvas UI is well-loved. Loses on worker support, pricing predictability for always-on services, and platform reliability.
- *Fly.io for hosting:* Strong for global multi-region apps. Overkill for a 4-friend group; no free tier for new users; more infrastructure-level control than we need.
- *Neon for database:* Excellent serverless Postgres with branching and scale-to-zero. But no auth, no secret management, and our workload (always-on workers polling) doesn't benefit from scale-to-zero anyway — workers keep the database warm.
- *Postgres on Render:* Possible, would give us private networking between app and DB. Loses Supabase Auth and Vault, which we want.

**Revisit when:**
- Render's pricing becomes painful at scale (unlikely until many groups).
- We need multi-region deployment for latency reasons (would push toward Fly.io).
- Supabase changes pricing or licensing in a way that hurts us.
- A platform offers a meaningfully better integration for Spotify or Apple Music APIs (none currently exist).

---

## 2026-05-08: OAuth refresh tokens stored in Supabase Vault, not in regular Postgres rows

**Decision:** Store OAuth refresh tokens (Spotify and Apple Music) in Supabase Vault rather than as encrypted columns in the `streaming_accounts` table. Access tokens can stay as encrypted columns since they're short-lived.

**Context:** Project priorities call out security as a deliberate concern, even at the cost of complexity. Refresh tokens are the highest-value credentials in the system — compromise of one means an attacker can impersonate a user against their streaming platform indefinitely until revoked.

**Reasoning:**
- Vault stores secrets encrypted with a key managed by Supabase, separate from the row data. Even a SQL injection that dumps `streaming_accounts` doesn't expose refresh tokens.
- Defense-in-depth: combines with TLS in transit, encryption at rest at the disk level, and (eventually) RLS policies.
- Access tokens expire in ~1 hour and are lower-value, so the operational simplicity of pgcrypto column encryption is fine for those.

**Alternatives considered:**
- *All tokens as pgcrypto-encrypted columns:* Simpler, one mechanism to maintain. Loses the separation-of-concerns benefit for the highest-value credentials.
- *App-level encryption with key in Render env vars:* Works, but now we're managing key rotation and storage ourselves. Vault is the managed answer to that.
- *Skip encryption, rely on TLS + disk encryption:* Inadequate for this risk profile.

**Revisit when:**
- Supabase Vault pricing or limits become a constraint.
- We move off Supabase (would need to migrate token storage strategy).
- A security review identifies a better pattern.

---

## 2026-05-08: Backend framework is Fastify

**Decision:** Use Fastify (with TypeScript) as the backend framework. Drop Hono from consideration.

**Context:** ARCHITECTURE.md previously listed "Hono (or Fastify)" without a commitment. Time to commit.

**Reasoning:**
- *Workload fit.* The app is a long-running Node service with persistent worker processes, BullMQ + Redis connections, and pgcrypto/Vault-encrypted token handling. This is exactly Fastify's lane. Hono's main differentiator — runtime portability to edge environments like Cloudflare Workers — is the one feature we've explicitly designed against by picking Render and a worker-heavy architecture.
- *Ecosystem fit for the actual integrations.* Spotify and Apple Music OAuth flows benefit from Fastify's mature plugin ecosystem (well-trodden patterns for OAuth, sessions, request validation). Hono's middleware ecosystem is younger, particularly for niche provider integrations — would mean writing more custom adapter code as a solo developer.
- *Long-term scaling.* Fastify is what production Node services run on at scale. If we ever break the app into services (e.g., one per platform integration, or extracting the track resolution module into a service), Fastify is the obvious foundation. Nothing about Hono helps that future.
- *TypeScript DX is a wash in practice.* Hono advocates cite better TypeScript inference, and they're right at the margin. Fastify with `@fastify/type-provider-zod` closes the gap to ~90% of Hono's DX, which is more than enough.

**Alternatives considered:**
- *Hono:* Real winner if we were ever going to deploy to Cloudflare Workers, Vercel Edge, or Deno. We aren't, and our workload (long-running workers, BullMQ, persistent DB connections) is the wrong shape for edge runtimes anyway.
- *Express:* The "safe boring" choice. Loses on TypeScript ergonomics, JSON Schema validation, and raw performance. No reason to pick it over Fastify in 2026.
- *NestJS:* Too heavy for a solo project. Convention overhead pays off with a team, not a single developer.
- *Encore.ts:* Interesting (managed infra + TypeScript) but adds a layer of platform lock-in we don't need on top of the platform decisions we've already made.

**Revisit when:**
- We ever want to push OAuth callbacks or webhook receivers to the edge for latency reasons (would push toward Hono — but webhook-style sync from streaming APIs doesn't currently exist).
- The app outgrows a single Node service and we genuinely need to architect for microservices (Fastify still works there, but might re-evaluate whether something like Encore.ts saves enough infra work to justify the switch).

---

## 2026-05-08: Frontend is Next.js (App Router) + Tailwind

**Decision:** Build the web client with Next.js App Router and Tailwind. Pull in shadcn/ui components à la carte (button, card, alert, etc.) when needed. No dashboard template — start from scratch.

**Context:** ARCHITECTURE.md previously listed "Next.js (or plain HTML + HTMX for v0 simplicity)" without committing. v0 is genuinely small enough that HTMX would work, so this needs a longer-term justification.

**Reasoning:**
- *MusicKit JS is the hardest part of v0 frontend, and it assumes React/Next.js context.* Apple's MusicKit web SDK requires real browser JavaScript with token handling, event listeners, and OAuth-like flows. Every existing tutorial, Stack Overflow answer, and AI-assisted code suggestion for MusicKit JS assumes a React/Next.js environment. Doing it inside HTMX would be a much lonelier path right when we'd want help.
- *The long-term product vision is mobile-shaped.* PROJECT.md says the social/group/party-mode layer is where this becomes a real product. That layer is mobile-native by definition (people at parties don't pull out laptops). Next.js gives us a clean path to a PWA, and component sharing with React Native if we ever go that direction. HTMX makes both of those harder.
- *Skill transferability for solo learning.* Time spent learning Next.js compounds across many future projects. Time spent learning HTMX is interesting but narrower.
- *No Next.js feature is wasted at our scale.* App Router + server components + Tailwind is the standard 2026 stack. The boring default is genuinely the right answer here.

**Anti-decision: don't grab a dashboard template.** The 100+-page admin templates floating around (Apex, Shadcn Admin, etc.) would bury us in unfamiliar code. Start with plain Next.js + Tailwind, copy in shadcn/ui components only as needed. Fight scope creep in the UI layer the same way we fight it in the backend.

**Alternatives considered:**
- *HTML + HTMX:* Genuinely well-suited to the v0 surface (3-4 pages, mostly read-only with some interactivity for OAuth flows). Loses on MusicKit ecosystem fit and on long-term mobile/PWA path.
- *Plain React + Vite (no Next.js):* Lighter than Next.js. Loses server components, file-based routing, and the API route convenience for OAuth callbacks. Not worth the savings.
- *SvelteKit:* Smaller ecosystem and worse fit for the React-centric MusicKit JS examples.

**Revisit when:**
- We're confident the long-term path is native mobile apps (Swift/Kotlin or React Native), making the web a permanent secondary surface — Next.js still works fine there, just less critical.
- A specific Next.js limitation actively gets in the way (none expected at our scale).

---

## 2026-05-08: Mobile path deliberately undecided; v0 web stack must keep it open

**Decision:** Do not commit to a mobile strategy yet. Keep the v0 web stack capable of evolving into a PWA, and capable of sharing code with React Native if we go that route.

**Context:** PROJECT.md and prior decisions repeatedly defer mobile to "v1+" without specifying the path. The long-term product vision (party mode, in-app song adding, social features) is mobile-native, so the mobile path will eventually need to be chosen — but not yet. This decision exists to make the deferral explicit and to constrain the v0 stack so we don't accidentally close doors.

**Reasoning:**
- *Premature mobile decisions are expensive.* Choosing native (Swift + Kotlin), React Native, or PWA each implies different team skills, different backend API shapes, different deployment pipelines, and different feature timelines. Picking now, before v0 is validated, would optimize for a future we don't know we'll have.
- *The web stack we pick today shouldn't foreclose options.* Next.js + Tailwind keeps PWA and React Native paths cleanly open (PWA is a configuration change; React Native can share components, hooks, and logic with a Next.js codebase). HTMX would have closed both. This is part of why we picked Next.js even though HTMX would technically work for v0.
- *Three paths we explicitly want to keep available:*
  1. **Native mobile apps (Swift/Kotlin or React Native) later.** Web stays as a status/admin surface forever.
  2. **Mobile-optimized PWA.** One Next.js codebase serves desktop and a phone-friendly install experience.
  3. **Cross-platform via React Native sharing components with Next.js.** Most ambitious; most code reuse.

**Alternatives considered:**
- *Commit to React Native now.* Premature; v0 isn't validated, and learning React Native solo on top of everything else would balloon the v0 timeline.
- *Commit to PWA now.* Reasonable but would add scope to v0 (manifest, service worker, offline behavior, mobile UX testing). Defer.
- *Commit to "web only forever."* Cuts off the social/party-mode product vision. No.

**Revisit when:**
- v0 is validated against the success criteria.
- A friend group member or early user surfaces a real mobile use case during a v0 event.
- We start designing party mode or in-app song adding (these features force the decision).

---

## 2026-05-09: Project will use simpler stack during foundation/learning phase

**Decision:** While the developer (sole contributor) is learning fundamentals, the project will deviate from the documented production stack. Specifically: plain JavaScript before TypeScript, plain Node + npm scripts before Fastify, plain `.env` files before Supabase Vault for secrets, and a single laptop dev environment before Render deployment. The documented stack in ARCHITECTURE.md and prior decisions remains the *target* — this entry establishes a deliberate, traceable gap between target and current state during the learning phase.

**Context:** Solo developer is at near-zero coding experience as of day 1 of build. Original DECISIONS.md entries (2026-05-04 and 2026-05-08) describe a production-grade stack chosen for the right reasons but not appropriate as a starting point for someone who has not yet written a function, used Git, or installed Node. Today's environment-setup session (Homebrew, Node 22 LTS, terminal fundamentals) confirmed the gap is real.

**Reasoning:**
- *Building security and architectural sophistication on top of unfamiliar fundamentals leads to brittle code.* Encrypted token storage, JWT signing with ES256, BullMQ workers, and Postgres schema design are all things that will be built — but built badly if attempted before the developer can confidently write and debug a basic function.
- *The documented decisions are not wrong; they are not yet appropriate.* Fastify, Supabase Vault, Render workers, TypeScript — all correct for the production system. None are correct for "I'm learning what a variable is."
- *Foundation work compounds; rework does not.* Time spent learning JavaScript fundamentals carries forward into TypeScript later. Time spent fighting TypeScript while also learning JavaScript wastes both.
- *Migration milestones rather than a single big switch.* As skills become comfortable, we migrate piece by piece — JavaScript to TypeScript when types start feeling more like a help than a hindrance; plain Node scripts to Fastify when the API surface justifies a framework; `.env` to Vault when refresh tokens actually exist and need protecting.

**Alternatives considered:**
- *Stick strictly to the documented stack from day one.* Rejected: too much concept load at once, high risk of demoralization or copying-without-understanding.
- *Pay a developer to build v0.* Rejected: defeats the developer's stated goal of learning to code, and the $500/year v0 budget rules it out anyway.
- *Abandon the documented decisions entirely.* Rejected: those decisions are correct for the eventual production system, and re-deciding from scratch later would lose the reasoning captured during the strategy phase.
- *Use a beginner-oriented stack permanently (e.g., stay on plain JavaScript forever).* Rejected: TypeScript and the production architecture exist in the docs for real reasons (type safety on the platform adapters, secret-handling for high-value credentials). The target is correct.

**Revisit when:**
- Each migration milestone hits — feel free to migrate to TypeScript, Fastify, or Vault as soon as the relevant fundamentals feel comfortable, rather than waiting for a single big rewrite.
- v0 is validated against PROJECT.md success criteria and the stack gap has fully closed.
- The learning trajectory diverges materially from expectations (e.g., project takes much longer or shorter than the 8-14 month informal estimate, or the developer's preferences shift toward a different stack as they learn).
