# EWFC App Map

Label: wayfinder:map

## Destination

A written spec + domain model for the EWFC futsal community app — ledger/payments, people, places, match planning, live match-day ops, and announcements — sharp enough that build sessions can implement it without asking another product question.

## Notes

Domain: amateur futsal community "EWFC", Indonesia, currency IDR (Rupiah, written Rp 25.000).

Stack: SolidJS v2 (v2.solidjs.com) — see repo CLAUDE.md, always SolidJS v2 and the /unslop skill. pnpm, vite, vitest. Repo is currently the SolidJS v2 starter: `src/server/db.ts` is an in-memory Map placeholder and the only module that knows the data source.

Every session: consult `/grilling` and `/domain-modeling`. Use `/research` for facts outside the repo, `/prototype` when the question is "how should this look or behave".

### Established constraints (settled while charting, not up for re-litigation)

- **Destination is a spec**, not shipped code. Plan, don't build.
- **Accounts are admin-created.** No public signup. Player gets an initial password/code and can change it. Phone number is the contact detail on a Person.
- **The fund is a real account**, not just the sum of player balances: it takes non-match income and pays non-match expenses.
- **Deposits are wallet top-ups.** Rp 25.000 per match is an expected-top-up nudge in the UI, not a domain rule. The ledger only records real amounts.
- **Bill splits by minutes played, with an equal-share floor.** Keeper carries a lower weight. Booked-but-unplayed time before first kickoff is split equally across all attendees.
- **Rotation format is configurable.** Both winner-stays and fixed rotation blocks are used; the app must help pick fast based on head count and conditions. Deciding the format is itself a thing the community currently wastes time on.
- **Teams are randomized, then hand-editable.** Admin can swap players after the shuffle. Keepers spread across teams.
- **Match-day is a live console**: tap substitutions off the formed teams, track score and goal scorers, surface who has played too long or too little.
- **Referee/timekeeper is compensated** through the ledger (subsidy or balance credit).
- **Announcements are copy-out.** App generates the text, a human pastes it into the existing WhatsApp group.

## Decisions so far

<!-- one line per resolved ticket -->

## Not yet specified

- Fairness indicator: how "played too long / not enough" is computed and shown during a session.
- Pairing history — randomizing to favour players who have never played together. User: "if complicated, maybe later."
- Player stats surfacing: goal scorers, appearances, leaderboards.
- The referee/timekeeper subsidy amount, and whether it is a fund expense or a discount on their own share.
- Player-facing views: my balance, my minutes, fund transparency.
- Settlement: what "this match is settled" means, and how a mistake gets corrected after settlement.
- Importing whatever spreadsheet or WhatsApp history the community uses today.
- Overall app IA and navigation.

## Out of scope

- Auto-posting announcements to WhatsApp via the Business API — real future opportunity, but it needs API approval and per-message cost. Manual copy-out ships now.
- Public or self-serve player signup — ruled out while charting; admin creates every account.
- Skill-rating-based team balancing — manual post-shuffle swapping was chosen instead; ratings go stale and cause arguments.
- Online payment gateway for top-ups. Money changes hands in person; the app records it.
