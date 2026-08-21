# Persistence, hosting and venue connectivity

Research for [issue #6](https://github.com/riolly/ewfc/issues/6). All pricing, tier and region claims checked **2026-08-21** against the provider's own page, linked inline.

## Summary

Run the app on **Cloudflare Workers** and the data on **Neon Postgres in `aws-ap-southeast-1` (Singapore)**. Build the match-day console around a **durable local outbox in IndexedDB**, not around optimistic UI alone.

The connectivity answer in one sentence: optimistic UI with retry is not enough, but you do not need a sync engine either, because there is one writer per match and match events are append-only, so a local append log plus an idempotent server endpoint covers every failure mode for a few hundred lines of code.

## The seam you have to fit

`src/server/db.ts` is 26 lines. It imports the `server-only` marker, holds a `Map<string, User>`, and exports three **synchronous** functions: `listUsers()`, `findUser(id)`, `updateUser(id, data)`.

Two call sites consume it:

- `src/lib/users.ts` calls all three from inside `'use server'` functions wrapped in `query()` and `action()`. These are already `async`, so adding `await` costs nothing.
- `src/routes/api/users.ts` exports `export const GET: APIHandler = () => Response.json(listUsers())`. This one has to become `async`.

That is the whole blast radius of swapping to a real database: make the three exports async, add `await` at four call sites, add one driver dependency, add `DATABASE_URL` to the `server` block of `env.ts`. The README's claim that "the db module is the only file that knows the data source" holds up. Nothing in the recommendation below is invasive.

Worth noting what the seam does *not* give you. It is a repository of plain functions, not a connection or a transaction handle. A money ledger needs multi-statement transactions (debit one row, credit another, insert an audit line, all or nothing). Whatever driver you pick, `db.ts` will need to grow a `withTransaction(fn)` export or expose the client itself. Decide that when you write the ledger, not now.

The typed env story matters here too. `env.ts` declares server vars as Standard Schema validators, and `virtual:env/server` reads them from `process.env` **at server boot** and validates then. So a `DATABASE_URL` in `env.ts` fails the boot loudly if it is missing, and never ends up in a build artifact. That is the right behaviour for a database URL and you get it for free.

## Latency from Indonesia, measured

Every provider comparison below is downstream of this table, so I measured it rather than guessing. TCP connect time is one round trip. Taken 2026-08-21 from Yogyakarta on Biznet fixed broadband (AS17451), best of repeated attempts.

| Target | Measured 1-RTT |
|---|---|
| Cloudflare edge, colo `CGK` (Jakarta) | **17 ms** |
| AWS `ap-southeast-1` (Singapore) | **54 ms** |
| Supabase pooler `ap-southeast-1` | 47 ms |
| AWS `ap-northeast-1` (Tokyo) | 128 ms |
| AWS `ap-southeast-2` (Sydney) | 144 ms |
| AWS `ap-south-1` (Mumbai) | 213 ms |
| AWS `us-east-2` (Ohio) | 276 ms |
| AWS `us-east-1` (Virginia) | 280 ms |

`https://cloudflare.com/cdn-cgi/trace` returned `colo=CGK`, confirming Cloudflare terminates in Jakarta for this network, so a Worker runs there.

Three things fall out of this:

1. Singapore is the only sane database region. Tokyo costs 2.4x, US East costs 5x, on every single query.
2. Anything that puts compute or data in the US is disqualified for the match console. 280 ms per round trip is a console that feels broken.
3. The gap between Jakarta compute at 17 ms and Singapore compute at 54 ms is real but small next to what a 4G link inside a concrete futsal hall adds. Do not over-optimise this. Get into Singapore or Jakarta and stop.

Caveat I want to be honest about: this is fixed broadband from a city, not a phone in a hall. It sets a floor, not an expectation. Someone should repeat these numbers on the actual match-day phone at the actual venue before anyone treats them as the design budget.

## A. Database

### Candidates

| Option | Nearest region to ID | Free tier (checked 2026-08-21) | Works from a v2 server module | Swap cost |
|---|---|---|---|---|
| **Neon Postgres** | `aws-ap-southeast-1` Singapore, 54 ms | 0.5 GB/project, 100 CU-h/project/mo, 100 projects, 5 GB egress, 6-hour PITR | Yes. `pg` over TCP on Node, or `@neondatabase/serverless` over HTTP/WebSocket on workerd | Low |
| **Supabase Postgres** | `ap-southeast-1` Singapore, 47 ms | 500 MB db, 5 GB egress, 2 active projects, **paused after 7 days idle** | Yes, same drivers | Low |
| **Turso / libSQL** | **Tokyo `aws-ap-northeast-1`, 128 ms.** No Singapore | 5 GB, 500M row reads/mo, 10M row writes/mo, 100 databases, 1-day PITR | Yes. `@tursodatabase/serverless` is fetch-only, so it works everywhere | Low |
| **Cloudflare D1** | Location hint `apac`, exact city undisclosed | 5 GB total, 5M row reads/day, 100k row writes/day | Only from a Worker binding, or the HTTP API | Medium. Ties the app to Cloudflare and needs `import { env } from "cloudflare:workers"` rather than the template's `virtual:env/server` |
| **SQLite on a Fly/Railway volume** | Wherever the box is | Free with the box | Yes, `node:sqlite` or better-sqlite3 | Low to write, high to operate |

Sources: [Neon pricing](https://neon.com/pricing), [Neon plans](https://neon.com/docs/introduction/plans), [Neon regions](https://neon.com/docs/introduction/regions), [Neon serverless driver](https://neon.com/docs/serverless/serverless-driver), [Supabase pricing](https://supabase.com/pricing), [Supabase regions](https://supabase.com/docs/guides/platform/regions), [Supabase production checklist](https://supabase.com/docs/guides/platform/going-into-prod), [Turso pricing](https://turso.tech/pricing), [Turso locations API](https://docs.turso.tech/api-reference/locations/list), [Turso TS quickstart](https://docs.turso.tech/sdk/ts/quickstart), [D1 pricing](https://developers.cloudflare.com/d1/platform/pricing/), [D1 data location](https://developers.cloudflare.com/d1/configuration/data-location/), [D1 read replication](https://developers.cloudflare.com/d1/best-practices/read-replication/).

### Why Turso loses despite the best free tier

Turso's free tier is the most generous here by a wide margin. It is also in the wrong hemisphere. [The Turso locations API](https://docs.turso.tech/api-reference/locations/list) lists exactly six AWS locations as of 2026-08-21: `us-east-1`, `us-east-2`, `us-west-2`, `eu-west-1`, `ap-northeast-1`, `ap-south-1`. No Singapore, no Jakarta. Turso's [own AWS GA post](https://turso.tech/blog/turso-aws-out-of-beta) confirms `ap-northeast-1` is what "East Asian locations" resolve to and `ap-south-1` covers "South Asian and Oceania locations". Indonesia sits between the two and gets whichever is assigned.

Tokyo measured 128 ms from Indonesia. If you also host in Singapore, every query pays a Singapore to Tokyo hop on top. I measured `api.turso.tech` at 168 ms, which is the US control plane, so even provisioning is slow from here.

Turso becomes the best option here the day it ships `ap-southeast-1`. Until then it is a 2.4x latency tax for a free tier this app will never come close to exhausting anyway. A community of tens of people generates kilobytes.

### Why not Supabase

Supabase is genuinely close, 47 ms measured to the Singapore pooler, and the free tier's 500 MB is plenty. The disqualifier is the pause rule. [Supabase's own docs](https://supabase.com/docs/guides/platform/going-into-prod) say free projects "may be paused" after 7 days of low activity, and restoring one is a manual click in the dashboard.

A futsal community plays weekly. Seven days is exactly the interval between sessions. Some weeks nobody opens the app between Saturdays, and the admin arrives at the hall to a paused database and a dashboard login he has to do on a phone on 4G. That is a bad Saturday. You can paper over it with a cron ping, but a design that needs a keepalive to survive its own usage pattern is the wrong design.

Neon's equivalent behaviour is much friendlier: it suspends the *compute* after 5 minutes and [reactivates "within a few hundred milliseconds"](https://neon.com/docs/introduction/scale-to-zero) on the next query. Free plan cannot disable it, which is fine, because a sub-second resume on the first tap of a match is invisible next to the mobile link. Nothing is ever paused in the sense of needing human intervention.

### Why not D1

D1 is free, fast from a Worker, and read replication costs nothing. Two problems. Its location hints top out at `apac` with no published city, so I cannot tell you where writes actually land, and writes always go to the primary. And it only reaches your code as a Worker binding, which means `db.ts` stops reading config from `virtual:env/server` and starts importing `env` from `cloudflare:workers`. Workable, and Cloudflare [documents that import explicitly](https://developers.cloudflare.com/workers/runtime-apis/bindings/), but it welds the data layer to one host and you lose the ability to run the server locally with plain `node server.js`.

Keep D1 in your pocket. If Neon's free tier changes shape, D1 is the fallback that costs nothing to switch to.

### Recommendation: Neon, Singapore

Free tier is comfortably sized for this app, Singapore is the closest region anyone offers, and it is plain Postgres, so nothing about the choice is a trap. If you outgrow free, Launch is metered pay-as-you-go with no monthly minimum.

One thing to fix on day one: **Neon Free retains only 6 hours of history**, so its point-in-time restore is not a backup for the money ledger. Add a scheduled `pg_dump` somewhere off Neon. A GitHub Actions cron committing a dump to a private repo is free and takes twenty minutes to set up. Given that the ticket calls the ledger the durability-critical part, a 6-hour restore window is not enough on its own.

## B. Hosting

### What SolidJS v2 actually supports

I checked the [v2 deployment docs](https://v2.solidjs.com/building-apps/deployment) against the template's own README. They agree, and the list is short:

- **Node**, the default. `vite build` emits `dist/client` and `dist/server`, and the included `server.js` serves both. Any host that runs `node server.js` works.
- **Cloudflare Workers** via `@cloudflare/vite-plugin`, with `wrangler.jsonc` pointing `main` at `virtual:solid-ssr-handler` and `nodejs_compat` on.
- **Netlify** via `@netlify/vite-plugin`, which turns the Fetchable entry into a streaming Netlify Function.
- **Nitro v3** via `nitro({ serverEntry: false })`, which then unlocks whatever Nitro presets target.
- **Other fetch runtimes**, Bun and Deno, by pointing them at `dist/server/server.js`.

There is no Vercel adapter. Vercel is only reachable through Nitro's preset, which nobody has verified against this template. That is enough uncertainty to leave Vercel out of a recommendation even though its `sin1` region is available on the free Hobby plan.

### Candidates

| Host | Compute near ID? | Free tier (checked 2026-08-21) | Persistent server? | v2 adapter |
|---|---|---|---|---|
| **Cloudflare Workers** | Yes, Jakarta `CGK`, 17 ms | 100k req/day, **10 ms CPU per invocation** | Isolates, not a process. Scale to zero with no meaningful cold start | Official |
| **Fly.io** | Yes, `sin` Singapore, 54 ms | No. Trial is 2 machine-hours or 7 days | Yes, always-on Node | Node default, works unchanged |
| **Railway** | Yes, `asia-southeast1-eqsg3a` Singapore | $1/mo credit, not enough for always-on. Hobby is $5/mo | Yes, always-on Node | Node default, works unchanged |
| **Render** | Yes, Singapore | 750 instance-hours/mo, but **spins down after 15 min idle with a ~1 minute cold start** | Yes on paid ($7/mo Starter) | Node default, works unchanged |
| **Netlify** | **No on free.** Functions default to `cmh` Ohio, 276 ms. Region config is Pro and Enterprise only | 100 GB bandwidth | Functions, not a process | Official |
| **Vercel** | `sin1` selectable, Hobby is single-region | Hobby free | Functions | None. Nitro preset only, unverified |

Sources: [Workers pricing](https://developers.cloudflare.com/workers/platform/pricing/), [Workers limits](https://developers.cloudflare.com/workers/platform/limits/), [Cloudflare network](https://www.cloudflare.com/network/), [Fly regions](https://fly.io/docs/reference/regions/), [Fly pricing](https://fly.io/docs/about/pricing/), [Fly free trial](https://fly.io/docs/about/free-trial/), [Railway pricing](https://railway.com/pricing), [Railway regions](https://docs.railway.com/reference/regions), [Render regions](https://render.com/docs/regions), [Render free tier](https://render.com/docs/free), [Netlify function config](https://docs.netlify.com/build/functions/optional-configuration/), [Vercel regions](https://vercel.com/docs/regions), [Vercel function regions](https://vercel.com/docs/functions/configuring-functions/region).

### The two disqualifications

**Netlify free is out**, which surprised me given it has a first-party v2 plugin. [Netlify's docs](https://docs.netlify.com/build/functions/optional-configuration/) put the functions region setting behind Pro and Enterprise. The default for sites created after October 2023 is `cmh`, Ohio. I measured Ohio at 276 ms. Every server function call on match day would pay that, before the mobile link. Pro starts at $19 per member per month, which is more than this whole project should cost.

**Render free is out** for the same practical reason from a different direction. [Render's free tier docs](https://render.com/docs/free) say a free web service spins down after 15 minutes without traffic and the cold start "takes about one minute". The admin opens the console at the hall, taps, and waits a minute staring at a loading page. Render's $7/mo Starter fixes it and the Singapore region is good, so Render is a fine paid option, just not a free one.

### The Cloudflare catch, stated plainly

Workers Free gives **10 ms of CPU time per invocation** ([limits doc](https://developers.cloudflare.com/workers/platform/limits/)). Exceeding it returns Error 1102. That is CPU, not wall clock, so waiting on the database does not count, but rendering a page does. Streaming SSR of a non-trivial Solid page on a cold isolate can plausibly blow 10 ms. Workers Paid raises it to 30 seconds by default for $5/mo minimum.

Two ways out, and I think both are defensible:

1. **Pay the $5.** It removes the ceiling, removes the 100k/day request cap, and is the cheapest always-fast option in this whole document.
2. **Set `ssr: false` in `vite.config.ts`.** The template supports this as a one-boolean change: "pages render on the client while server functions, sessions, and API routes keep working." The Worker then does almost no CPU work per request, static assets are served by the `ASSETS` binding rather than by your code, and 10 ms is plenty. You lose SSR on the public pages and gain a cacheable app shell.

Option 2 is not just a cost dodge. An app-shell SPA is what a match-day console wants anyway, because a cached shell opens instantly with no signal, whereas an SSR page with no signal does not render at all. If the console ever becomes an installable PWA, you end up here regardless.

### Recommendation: Cloudflare Workers

Reasons, in order of weight:

- It is the only candidate whose compute is physically in Indonesia. 17 ms measured to `CGK` against 54 ms for the best Singapore box.
- The v2 template documents the adapter and ships the `wrangler.jsonc` for it. Nothing to invent.
- Static assets are served by Cloudflare, not by your process, so there is no machine to keep warm and no cold start to design around.
- $0 to start, $5/mo ceiling.
- If you later want a Durable Object per live match, it is one config block away rather than a migration.

The runner-up is **Railway Singapore at $5/mo**, and it is close. `server.js` runs unchanged, you get plain Node with no CPU cap and no workerd caveats, and `pg` over TCP to Neon in the same region is a couple of milliseconds. Pick Railway if workerd's Node compatibility gaps ever start costing you more than the latency difference is worth. The two are within about 20 ms of each other on a real interaction, which the mobile link swallows.

### Estimated per-tap cost on the recommended stack

One substitution tap is one single-flight `action()` round trip, since the router folds revalidated query data into the same response.

```
phone -> Worker (CGK)            ~17 ms on broadband, budget 60-120 ms on 4G in a hall
Worker -> Neon (SIN), HTTP       ~25 ms x 1-2 round trips
query execution                  ~1-5 ms
                                 -------
                                 ~70 ms broadband, ~150-200 ms realistic on match day
```

Under 200 ms is a console that feels responsive. That is the target, and it is achievable. It is also completely irrelevant when the signal is gone, which is what section C is about.

## C. Connectivity, the part that constrains the design

### Verdict

**Optimistic UI with retry is not enough. Full offline-first with bidirectional sync is more than you need. Build a durable local outbox.**

### Why optimistic UI alone fails

This is not a hand-wave about flaky networks. It follows from what Solid v2's own primitives are documented to do.

The [`action` reference](https://v2.solidjs.com/reference/solid-js/lifecycle-actions/action) describes pairing with `createOptimistic` / `createOptimisticStore` "to apply tentative writes that auto-revert if the action fails." The [`createOptimisticStore` reference](https://v2.solidjs.com/reference/solid-js/stores/create-optimistic-store) says writes during an action "show up immediately but auto-revert (or reconcile to the action's resolved value) once the action settles."

Auto-revert on failure is exactly right for a "like" button and exactly wrong for a match console. The admin taps a substitution, sees it land, the request fails, and the app **visibly undoes the substitution he just made** while he is looking at the pitch and not the screen. The optimistic primitive is doing its job. Its job is the wrong job here.

Solid Router does give you a retry handle. [`useSubmissions`](https://v2.solidjs.com/routing/solid-router/data) exposes `.result`, `.error`, `.clear()` and `.retry()` per submission, and the router retains outcomes that have a result or an error. So a failed write is recoverable by a tap. But that list lives in the page's memory, and that is the problem.

### Walking the actual failure modes

**Signal drops for 30 seconds.** One or two submissions fail. Optimistic writes revert, the UI flickers backwards, `useSubmissions` holds the errors. The admin either notices and taps retry, or does not notice and the events are silently wrong. Recoverable, ugly, and it depends on a human watching a screen during a match. He is not watching the screen. He is watching the match.

**Signal drops for 5 minutes.** Now there is a queue of failed submissions in memory, and five minutes is long enough for the phone to lock its screen, for the admin to switch to WhatsApp, for the OS to reclaim the tab under memory pressure, or for him to pull-to-refresh out of frustration. Any of those and the entire in-memory queue is gone with no trace that it existed. This is the case that decides it. Not "retry is hard" but "there is nothing left to retry."

**Signal is gone for the whole session.** With optimistic UI only, the match produced no data at all. Every substitution, goal and clock event evaporated. The admin finds out afterwards.

Note that the running clock is the one thing that survives fine either way, as long as you derive it from a stored `startedAt` timestamp rather than incrementing a counter. Store timestamps, not elapsed values.

### Why full offline-first is overkill

The ticket asks whether single-writer-per-match simplifies the sync story. It does, and completely.

One phone writes one match. There is no second device racing it. Match events are append-only facts, not mutable state, and the writer assigns their order. So there is no merge to compute, no conflict to resolve, no last-write-wins ambiguity, no vector clocks, no CRDTs, no local-first framework. The server's job reduces to accepting a log and replaying it.

That is a much smaller problem than "offline-first" usually names, and it should not be solved with a tool built for the big version.

### What to build instead

1. **Append before you send.** Every match event goes into an IndexedDB queue first, with a client-generated `crypto.randomUUID()` idempotency key and a client timestamp. Then attempt the network. Never the other way round.

2. **Render the console from the local queue, not from server state.** This is the important inversion and it is what removes the flicker. A failed write cannot visibly revert if the UI was never reading server state to begin with. Use `createOptimisticStore` for the parts of the app that genuinely want revert-on-failure semantics, and keep the match console off it.

3. **Idempotent server endpoint.** The server upserts events by UUID, so replaying the whole queue is harmless. This is what makes the flush loop trivially safe to be dumb about. When in doubt, resend.

4. **A flush loop with backoff.** Drain pending events in order, mark them synced on acknowledgement, back off on failure. Trigger it on a timer, on the `online` event, and on `visibilitychange`.

5. **Do not gate on `navigator.onLine`.** [MDN is unusually blunt about this](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/onLine): "this property is inherently unreliable, and you should not disable features based on the online status, only provide hints when the user may seem offline." A phone attached to a venue's captive-portal wifi reports online and cannot reach you. Drive everything off actual request failure and use `onLine` only to decide when to bother trying.

6. **Show an unsynced count.** A small "3 unsynced" badge turns a silent failure into something a human can act on. This is the cheapest reliability feature in the list.

7. **Hold a screen wake lock while the clock runs.** [`navigator.wakeLock.request("screen")`](https://developer.mozilla.org/en-US/docs/Web/API/Screen_Wake_Lock_API) is Baseline as of March 2025. It removes the single most common way the console dies mid-match. It auto-releases when the document becomes invisible, so re-acquire on `visibilitychange`.

Rough cost: an event table in IndexedDB, an idempotent POST handler, a flush loop, a badge. Call it 150 to 250 lines and an afternoon. Compare that against losing one match's data once.

### What you can skip

**Background Sync API.** Tempting, and the wrong dependency. I pulled [MDN's compat data for `SyncManager`](https://github.com/mdn/browser-compat-data/blob/main/api/SyncManager.json) directly: Chrome 49+, Chrome Android, Edge and Samsung Internet support it. Firefox does not. Safari does not, desktop or iOS, tracked at [webkit.org/b/182565](https://webkit.org/b/182565). **Android WebView does not either.** MDN flags the whole API as "not Baseline because it does not work in some of the most widely-used browsers."

It is fine as progressive enhancement and useless as a foundation. The real scenario is the admin holding the phone with the console open, which the in-page flush loop already handles. Add Background Sync later if you want, and only where it exists.

**A service worker for correctness.** You want one so the app shell loads with no signal, which is a real benefit. You do not need one for the outbox. Keep the two concerns separate so a service worker bug cannot eat match data.

### One storage caveat worth designing around

Safari deletes script-created storage from origins with no user interaction in the last seven days of browser use, per [MDN's storage quotas and eviction page](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria). A weekly futsal app on an iPhone sits right on that boundary.

Two mitigations, do both. Call `navigator.storage.persist()`, which Chrome and Safari grant automatically based on interaction history without prompting. And treat the outbox as strictly same-day: it is a buffer that should be empty by the time the admin leaves the hall, never the long-term record. The server is the record. If the outbox is still full on Monday, something has already gone wrong and the badge should have said so.

### The ledger is a different problem, treat it differently

The ticket says losing minutes is annoying and losing ledger entries is not acceptable. Those two sentences describe two different write paths, and I would build them differently.

**Match events** go through the outbox. Optimistic, local-first, eventually flushed, tolerant of a bad hall.

**Ledger entries** do not. Money is entered off-match by someone sitting down with a decent connection. Write those straight through to Postgres in a transaction, block the UI on the result, and show a hard error on failure. No optimism, no queue, no local buffer. The user is in a position to retry deliberately, so let them. An unacknowledged ledger write should look like a failure, never like a success that will sort itself out later.

Keeping money out of the eventually-consistent path is the single most valuable line in this document.

### When I would change my mind

- **Two admins editing the same live match.** Conflict handling stops being free. The append-only log mostly still works, but the clock and the score need one designated owner or you get two truths.
- **The hall turns out to have solid wifi.** You could drop to optimistic plus retry. I would not bother, because the outbox is already built and costs nothing to keep.
- **Match-day moves to a laptop on a stable connection.** Then optimistic plus retry genuinely is enough and the outbox is dead weight.
- **Turso ships `ap-southeast-1`.** Revisit the database choice. Its free tier is much larger than Neon's and the embedded replica story would suit this app well.

## Recommended stack

| Layer | Choice | Cost |
|---|---|---|
| Hosting | Cloudflare Workers, Jakarta edge | $0, or $5/mo to lift the 10 ms CPU cap |
| Database | Neon Postgres, `aws-ap-southeast-1` Singapore | $0 on Free |
| Driver | `@neondatabase/serverless`, HTTP mode for one-shot queries, WebSocket for transactions | |
| Ledger backup | Scheduled `pg_dump` off Neon, because Free retains only 6 hours | $0 |
| Match console | IndexedDB outbox, UUID idempotency keys, flush loop with backoff, unsynced badge, screen wake lock | |
| Ledger writes | Straight through, transactional, no optimism | |

Runner-up if workerd gets in the way: Railway Singapore at $5/mo running `server.js` unchanged with `pg` over TCP.

## What is unresolved and needs a human decision

1. **SSR on or off.** This is the real fork. Keeping SSR means paying $5/mo for Workers CPU headroom. Turning it off makes the free tier comfortable, gives the console a cacheable shell that opens with no signal, and costs SSR on the public ledger and history pages. I lean towards `ssr: false` for a match-day-first app, but it is a product call about how the public pages should feel, not a technical one.

2. **Nobody has measured the actual venue.** All my numbers are broadband from Yogyakarta. Someone should stand in the futsal hall with the match-day phone and record RTT and packet loss. If the hall has usable wifi, the outbox stays but becomes insurance rather than a load-bearing part. If it has nothing, the shell needs to be service-worker cached before anything else gets built.

3. **Which phone, and is it an iPhone.** Decides whether Safari's 7-day storage eviction applies, and whether Background Sync exists at all as an enhancement. It does not change the recommendation, but it changes how hard the same-day-flush rule needs to be enforced.

4. **`db.ts` needs a transaction export, and its shape is undecided.** The current three-function repository has no place to put "debit, credit and audit, atomically." Whether that becomes `withTransaction(fn)`, an exported client, or a set of coarse ledger-specific functions is a design decision that belongs with whoever writes the ledger schema.

5. **Ledger backup destination.** I recommend the dump. Where it lands, private GitHub repo, object storage, someone's Drive, is unresolved, and it needs a restore drill, not just a backup job. A backup nobody has restored is a rumour.

6. **Turso is worth a diary entry.** It is the best free tier here and only loses on region. If `ap-southeast-1` appears, this decision should be reopened rather than left to inertia.
