# Match-day console domain model

Type: grilling
Status: open
Blocked by: 03

## Question

What is the state machine and event model of a session in progress?

Pin down: the session lifecycle (planned → attendance taken → teams formed → games running → finished → settled), and what a Game is inside a session — its teams, its clock, its end condition. What events does the console record: substitution in/out, goal (scorer, team, minute), game start/end, player arrives, player leaves? Is per-player minutes a derived fold over sub events, or a stored number the console maintains? Can the admin correct history after the fact — undo a sub, fix a scorer, adjust a game's length?

Deliverable: the event list and the state each event mutates.
