# Money model and ledger primitives

Type: grilling
Status: open
Blocked by: —

## Question

What are the money entities, and what exactly can move money?

Pin down: is a balance stored or derived from a transaction log? What entry types exist — top-up, match charge, referee subsidy, external income (sponsor, fines), external expense (ball, jersey), manual adjustment? Does every entry hit exactly one player balance or the fund, or can one entry span both? How is a negative player balance represented and what does the app do about it? Is the fund's cash position derived from the same log, and does it reconcile against physical cash someone is holding?

Output: the ledger's ubiquitous language and the invariant that keeps it honest.
