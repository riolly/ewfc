# Persistence, hosting and venue connectivity

Type: research
Status: open
Blocked by: —

## Question

What data layer and hosting fit this app — a SolidJS v2 app, a handful of concurrent users, Indonesian latency, and a live console running on a phone inside a futsal hall?

Cover: database options that work with SolidJS v2 server modules (the repo's `src/server/db.ts` is the single seam to swap); hosting with acceptable latency from Indonesia on a free or cheap tier; and how badly the live match console breaks when the venue's mobile signal drops — does this force offline-first with local writes and later sync, or is optimistic UI with retry enough?

Deliverable: a recommendation with the connectivity trade-off stated plainly, since it constrains the match-day console design.
