# Futsal rotation formats for 8 to 20 players

Research for [issue #4](https://github.com/riolly/ewfc/issues/4). Written to be consumed by a format picker keyed on head count.

Every claim below is labelled as one of three things:

- **Codified** means it comes from the futsal Laws of the Game or from a published league rulebook.
- **Convention** means a pickup operator or a rec department has written down how their sessions actually run.
- **Derived** means it is my own arithmetic from the rotation rules. Nobody has measured this, and I say so where it matters.

## The one constraint that shapes everything

A futsal court holds 5 players per side, one of whom is the goalkeeper (codified, [FA Futsal Law 3](https://www.thefa.com/football-rules-governance/lawsandrules/laws/futsal/law-3---the-number-of-players)). So every minute the ball is in play spends exactly 10 player-minutes, whatever format is running.

That has a direct consequence for the billing model. **The format never changes the total bill. It only changes who pays what share.** Average court time per player is fixed at `10 x play_minutes / head_count` regardless of whether you run winner-stays, blocks, or one long match.

Only two things move the total pot:

1. **Dead time.** Changeovers, team-picking, and warm-up burn booking minutes that nobody is billed for. At 8-minute games with a 1-minute changeover, a 2-hour booking loses about 11 minutes to changeovers, roughly 10% of the court fee (derived).
2. **Playing 4v4.** Below 10 heads the pot shrinks 20% while the court fee stays the same (derived).

Working assumption for all the arithmetic below: a 120-minute booking yields about **105 minutes of clock** after arrival, changing, bibs, and an end buffer. The app should make this a setting.

Average court time per player at 105 minutes of clock, before subtracting changeovers (derived):

| Heads | 8 (4v4) | 10 | 12 | 14 | 15 | 16 | 18 | 20 |
|---|---|---|---|---|---|---|---|---|
| Avg minutes each | 105 | 105 | 88 | 75 | 70 | 66 | 58 | 53 |

## What the laws actually say

Worth pinning down, because several format choices are constrained by them and several are not.

- A match is two equal periods of 20 minutes, "unless otherwise mutually agreed between the referee and the two teams", and any change "must be made before the start of play and must comply with the competition rules" (codified, [FA Futsal Law 7](https://www.thefa.com/football-rules-governance/lawsandrules/laws/futsal/law-7---the-duration-of-the-match)). Short games are legal. There is nothing to argue about here.
- The clock stops whenever the ball is out of play (codified, [U.S. Futsal rules summary](https://futsal.com/rules-of-the-game-summary/)). 40 minutes of stopped-clock futsal runs 60 to 75 minutes of real time, which is why amateur leagues do not use it.
- Substitutions are unlimited and "flying", made with the ball in play, through the substitution zone, with the incoming player waiting until the outgoing player has fully crossed the touchline (codified, [FA Futsal Law 3](https://www.thefa.com/football-rules-governance/lawsandrules/laws/futsal/law-3---the-number-of-players)). Any player may swap with the goalkeeper.
- A match may not start with fewer than 3 players a side and is abandoned if a team drops below 3 (codified, [FA Futsal Law 3](https://www.thefa.com/football-rules-governance/lawsandrules/laws/futsal/law-3---the-number-of-players); [U.S. Futsal](https://futsal.com/rules-of-the-game-summary/) states minimum 3 to both start and finish). This is the hard floor when people leave early.
- Matchday squad is up to 14, with up to 9 named substitutes in FIFA competitions (codified, [FA Futsal Law 3](https://www.thefa.com/football-rules-governance/lawsandrules/laws/futsal/law-3---the-number-of-players), [FLM System league guide](https://www.flmsystem.com/blog/what-is-futsal-rules-and-how-to-start-a-league)). A 20-head pickup session already exceeds a legal squad, which is a hint that two fixed teams stops being the right answer somewhere around 16.
- Amateur leagues routinely shorten matches. FLM System's operator guide says organisers use "2 x 15 or even 2 x 12 minutes to fit more games into a single hall booking", and that "in a 3-hour hall slot you can comfortably fit 3 to 4 games" (convention, [FLM System](https://www.flmsystem.com/blog/what-is-futsal-rules-and-how-to-start-a-league)). Futsal NH runs 24-minute halves with a 1-minute half time (codified league rule, [Futsal NH](https://futsalnh.com/futsal-rules/)).

## Format 1: Winner stays (king of the court)

**Teams and size.** Three or more teams of 5. Requires head count to be an exact multiple of 5, so 15 or 20. Anything else leaves a spare who has to be bolted onto a team.

**Game boundary.** Three documented variants, and the choice matters more than anything else in this format.

- *First to one goal.* Tucson Soccer Center: "First team to score remains on the field; team that is scored on enters the back of rotation" (convention, [Tucson Soccer Center](https://www.tucsonsoccercenter.com/pickup)).
- *Goal target with a time cap.* Purdue's challenge-court rules for soccer: "Two Goals or 7 Minutes" wins the match (convention, [Purdue RecWell](https://purdue.edu/recwell/visit/facility/policies/challenge-rules.php)).
- *Pure clock.* Tucson's alternative "Game Clock" style sets "scoreboards to 5 to 10 minute intervals and teams are rotated" (convention, [Tucson Soccer Center](https://www.tucsonsoccercenter.com/pickup)). TOCA runs 5-minute games with no goal limit (convention, [TOCA](https://www.tocafootball.com/adult-pickup-soccer)). OpenSports' organiser guide suggests 8-minute games (convention, [OpenSports](https://opensports.net/blog/top-5-concerns-everyone-has-when-organizing-a-pickup-game)).

**Games per 2 hours** (derived, 105 minutes of clock, 1-minute changeover):

| Game length | Cycle | Games |
|---|---|---|
| 5 min | 6 min | 17 |
| 7 min | 8 min | 13 |
| 8 min | 9 min | 11 |
| 10 min | 11 min | 9 |
| First to 1 goal | unbounded | unpredictable |

The goal-only variant has no upper bound on a single game and therefore no predictable total play time. Under minutes-based billing that makes the per-minute rate unknowable until the session ends. If the community wants goal-based games, the app should force a time cap, which is exactly what Purdue's "two goals or 7 minutes" does.

**Court-time distribution.** This is where the format hurts, and it is the reason two separate rec departments cap it.

With 3 teams the loser sits exactly one game. The extreme case is one team winning everything: it plays 100% of games while the other two play 50% each, against an equal share of 66.7%. Max-to-min ratio 2.0 (derived).

With 4 teams the equal share is 50%, and an unbeaten team plays 100% while the other three play 33% each. Ratio 3.0 (derived). Winner-stays gets worse as you add teams.

In money, at 15 heads and 11 games of 8 minutes: the dominant team's 5 players play about 88 minutes each and the other 10 play about 44 each, against a mean of 59. The king's players pay 1.5x the average, the rest pay 0.75x. **Under an equal-share floor, two thirds of the roster sit on the floor and the winners absorb the whole difference.** Winner-stays turns the floor into a subsidy from winners to losers. That may be exactly what the community wants, or exactly what it does not, but it should be a deliberate choice.

Capping consecutive wins is the standard fix and both published rulebooks I found do it. BYU: winners "stay on the court for a maximum of two consecutive games, then rotate out" (convention, [BYU Student Wellness](https://studentwellness.byu.edu/open-play)). Purdue: "after a team wins three games in a row, they must rotate out" (convention, [Purdue RecWell](https://purdue.edu/recwell/visit/facility/policies/challenge-rules.php)). OpenSports' 3-team suggestion caps at two (convention, [OpenSports](https://opensports.net/blog/top-5-concerns-everyone-has-when-organizing-a-pickup-game)). TOCA's "each squad plays 2 games, then takes a breather" is the same cap applied unconditionally (convention, [TOCA](https://www.tocafootball.com/adult-pickup-soccer)).

With a cap of 2 and 3 teams, the worst case drops to roughly 75% versus 62%, a ratio near 1.2 (derived). The cap does most of the work.

**Odd head count.** Bad. Needs a multiple of 5. At 16 heads one team carries a 6th who rotates internally, and that team's players each lose a sixth of their minutes relative to the other two teams. At 17, 18, and 19 it gets progressively more arbitrary.

**Early leaver.** Worst of the four formats, because the damage compounds. A team drops to 4 and now loses every game, which under winner-stays means it also sits every other game. It loses playing time for the same reason it lost the game. The alternatives are borrowing from the resting team (which breaks team identity mid-session) or abandoning the format.

**Live tracking.** Medium difficulty but fragile. Sub taps are few, because teams are fixed within a game. But the app must record each result correctly to know who is on next, and one wrong result desyncs the rest of the session's minute ledger. That is a worse failure mode than a mistapped substitution.

## Format 2: Fixed rotation blocks

Teams rotate on a pre-set schedule. Results do not affect who plays next. This is the format the community already uses alongside winner-stays.

**Teams and size.** Three teams of 5 is the canonical shape. Sons of Pitches FC runs "games up to 8v8, rotating up to 3 teams per field (20 min on, 10 min off)" (convention, [Sons of Pitches FC listing](https://opensports.net/posts/thurs-pickup-soccer-rotating-teams-turf--Gye-OzmN)). TOCA's "teams switch every 5 mins, each squad plays 2 games, then takes a breather" is the same idea at a shorter cadence (convention, [TOCA](https://www.tocafootball.com/adult-pickup-soccer)).

**Game boundary.** A fixed clock block. Nothing else works, because the whole point is that the schedule is known in advance.

The 20-on/10-off scheme is a staggered rotation on a 10-minute segment. Every 10 minutes the team that has been on for 20 comes off and the rested team comes on. Each team is on for 2 of every 3 segments.

**Games per 2 hours.** 105 minutes divided by the segment length. At 10-minute segments that is 10 segments, at 8-minute blocks with a 1-minute changeover it is 11 (derived).

**Court-time distribution.** Exactly equal by construction, at 2/3 of play time each, provided the teams are the same size and the number of segments is a multiple of 3. A partial final cycle is the only source of unfairness and it is worth at most one segment (derived).

**This is the best format for a minutes-billing model.** The equal-share floor never binds, because everyone is already at the equal share.

**Odd head count.** Same multiple-of-5 constraint, but it degrades more gracefully than winner-stays. Make one team a 6 and let it rotate internally. Those players get 5/6 of their team's block time, so 55.6% instead of 66.7% (derived). Under minutes-billing they pay proportionally less, so the money stays fair even though the play does not. That is a genuinely good property and worth saying out loud to the community.

**Early leaver.** Better than winner-stays, because results are not in the loop. A team down to 4 keeps its scheduled minutes and just plays 4v5, or you promote the spare, or you collapse to two squads with rolling subs.

**Live tracking.** Easiest by a wide margin. The schedule is known at the start, so the app can pre-compute every player's minutes before a ball is kicked and only handle corrections. Zero taps for the rotation itself. Scorers still need tapping.

## Format 3: Round-robin across 3 or more teams

Mechanically this is fixed rotation blocks with a fixture list and a table. For 3 teams the court-time pattern is identical. What changes is that fairness now depends on completing whole rounds, and that the session produces a result somebody cares about.

**Teams and size.** Three teams gives 3 fixtures per round. Four gives 6. Five gives 10, which does not fit a single-court 2-hour booking at any sane game length.

**Game boundary.** Fixed clock, uniform across fixtures, or the table is not comparable. Standard round-robin practice adds a bye placeholder for an odd team count so one team sits each round.

**Games per 2 hours.** The fixture count must be a multiple of the round size or the last partial round breaks the fairness (derived):

| Teams | Fixtures/round | Game length | Games in 105 min | Full rounds | Minutes played per team |
|---|---|---|---|---|---|
| 3 | 3 | 7 min (+1) | 13 | 4 rounds = 12 games | 56 |
| 3 | 3 | 8 min (+1) | 11 | 3 rounds = 9 games | 72 |
| 4 | 6 | 7 min (+1) | 13 | 2 rounds = 12 games | 42 |
| 4 | 6 | 8 min (+1) | 11 | 1 round = 6 games | 48 |

Four teams at 8-minute games only fits a single round, which is thin for a competition. Seven-minute games are the better fit at 4 teams.

**Court-time distribution.** Exactly equal on whole rounds. Play one extra game and that team gets an extra fixture's worth of minutes, roughly 8 minutes of spread on a 60-minute mean (derived). Small, but the app should either refuse the partial round or fold the spare time into a longer final.

**Odd head count.** Same as fixed blocks.

**Early leaver.** The only format here where a departure corrupts the *output*, not just the minutes. A team playing the last third short makes the standings meaningless. If the community cares about the table, the app needs a rule for voiding or adjusting affected fixtures, and that is a community decision, not a technical one.

**Live tracking.** Same pre-computable minutes as fixed blocks, plus results and standings. Easy for time, a bit more UI.

## Format 4: Two fixed teams with rolling subs

League futsal applied to pickup. Two squads larger than 5, flying substitutions per Law 3.

**Teams and size.** Two squads. Absorbs any head count from 8 upward with no divisibility constraint at all. At 10 heads it is 5v5 with nobody resting.

**Game boundary.** One continuous match, or halves or quarters. FIFA's 2x20 stopped clock (codified, [FA Futsal Law 7](https://www.thefa.com/football-rules-governance/lawsandrules/laws/futsal/law-7---the-duration-of-the-match)) runs 60 to 75 real minutes. Amateur convention is 2x15 or 2x12 on a running clock (convention, [FLM System](https://www.flmsystem.com/blog/what-is-futsal-rules-and-how-to-start-a-league)). Futsal NH uses 24-minute halves with a 1-minute break (codified league rule, [Futsal NH](https://futsalnh.com/futsal-rules/)).

For a 105-minute session: four 24-minute quarters with three short breaks, or two halves of about 48 minutes, or two back-to-back 2x20 running-clock matches.

**Games per 2 hours.** One or two matches. This format spends almost nothing on changeovers, so it converts about 8% more of the booking into billable court time than any rotation format (derived).

**Court-time distribution.** Nominally perfect. Per-player share is `5 / squad_size` of the match clock:

| Heads | Split | Share each | Minutes at 95 min play |
|---|---|---|---|
| 12 | 6v6 | 83% | 79 |
| 14 | 7v7 | 71% | 68 |
| 16 | 8v8 | 62.5% | 59 |
| 20 | 10v10 | 50% | 47 |

In practice it is not perfect, and there is measured evidence of that. In a professional futsal match with seven outfield players per side and a coach managing substitutions, individual playing time was **31.71 ± 9.02 minutes out of 40** (Dos-Santos et al. 2020, [Front Psychol 11:620108](https://pmc.ncbi.nlm.nih.gov/articles/PMC7767871/)). One standard deviation is 28% of the mean. Pickup with nobody managing subs will be worse.

That study also gives usable defaults for a sub timer: stints averaged 7.15 ± 2.39 min in the first half and 9.49 ± 3.80 min in the second, each player took about two stints per half, and the in-court to out-of-court ratio was 1:1.18 ± 1:0.51.

**This is the format where the app adds the most value.** The gap between the nominal 100% fairness and the measured ±28% is precisely what a live console with running per-player minutes closes.

**Odd head count.** Best of the four. 13 heads is 7v6, giving 83% and 71%, a ratio of 1.17 (derived). Under minutes-billing the difference lands in the money automatically and nobody argues about it.

**Early leaver.** Also the best. A squad going from 7 to 6 raises everyone else's minutes and their bill in the same step, self-correcting. The only floor is Law 3's minimum of 3 players.

**Live tracking.** Highest tap volume by far, but the simplest state model: two rosters, an on/off toggle per player, no queue, no result-driven branching. A mistap costs one player's minutes instead of desyncing the whole session. For a phone console this is the format the design is aimed at.

## Comparison table

| | Winner stays | Fixed rotation blocks | Round-robin (3-4 teams) | Two teams, rolling subs |
|---|---|---|---|---|
| Teams / size | 3-4 x 5 | 3 x 5 (or 5/5/6) | 3-4 x 5 | 2 squads of 4-10 |
| Head count needed | multiple of 5 | multiple of 5, tolerates +1 | multiple of 5 | any, 8+ |
| Game boundary | first to 1-2 goals, or 5-10 min clock | fixed clock block, 5-20 min | uniform fixed clock, 7-8 min | halves or quarters, 12-24 min each |
| Games per 2h | 9-17 (unbounded if goals-only) | 10-11 segments | 6-12 fixtures | 1-2 matches |
| Court time, ideal | 66.7% (3 teams) | 66.7% | 66.7% / 50% | 5 / squad size |
| Court time, worst realistic spread | **2.0x (3 teams), 3.0x (4 teams) uncapped; ~1.2x with a 2-win cap** | 1.0x | 1.0x on whole rounds | 1.0x nominal, ~1.9x measured without management |
| Equal-share floor | binds for most of the roster; winners subsidise | never binds | never binds | binds occasionally |
| Odd head count | poor, needs a bolted-on spare | fair, one team of 6 rotates internally | poor | excellent, absorbs anything |
| Early leaver | worst, damage compounds through results | tolerable | corrupts the standings | best, self-correcting in money |
| Live tracking | medium taps, **fragile**: a wrong result desyncs the ledger | lowest, minutes are pre-computable | low, plus a standings view | highest taps, most robust state |
| Dead time cost | ~10% of booking | ~10% of booking | ~10% of booking | ~2% of booking |

## Recommendation by head count

The picker logic in one line: **below 15, two fixed teams is the only sane option. At 15 and above, multi-team formats open up, and the choice is between even minutes (blocks) and a competition (round-robin), with winner-stays acceptable only if capped.**

| Heads | Recommended | Also viable | Avoid | Why |
|---|---|---|---|---|
| 8 | 4v4, two fixed teams, quarters | | anything multi-team | Everyone plays 100%, but no rest. Run quarters. Billable pot is 20% smaller than 5v5 while the court fee is not. |
| 9 | 5v4 two fixed teams | 4v4 plus one shared floating sub | multi-team | 5-squad 80%, 4-squad 100%. Minutes-billing absorbs it. A floating sub keeps play even but muddies the ledger. |
| 10 | 5v5 two fixed teams, quarters | | multi-team | Full court, everyone 100%, zero rest. Two hours at 5v5 without subs is punishing, so break it into quarters. |
| 11 | 6v5 two fixed teams | | multi-team | 83% / 100%. Fine. |
| 12 | **6v6 two fixed teams** | | multi-team | The sweet spot. 83% court time each with a real rest cycle, near-zero dead time, and the sub-tracking load is light. |
| 13 | 7v6 two fixed teams | | multi-team | 71% / 83%, ratio 1.17, handled by the billing. |
| 14 | 7v7 two fixed teams | | multi-team | 71% each. Matches a legal 14-player matchday squad on each side of a normal futsal fixture. |
| 15 | **3 x 5 fixed rotation blocks** | 3-team round-robin; 8v7 two teams; winner-stays *with a 2-win cap* | uncapped winner-stays | First count where three fives divide exactly. Blocks give exactly 66.7% each with pre-computable minutes. Uncapped winner-stays risks a 2:1 spread. |
| 16 | 8v8 two fixed teams | 3 teams of 5/5/6 with internal rotation | winner-stays | 62.5% each with no spare-player problem. If the group wants a competition, the 6-team rotates internally at 55.6%. |
| 17 | 9v8 two fixed teams | 3 teams of 6/6/5 | winner-stays | Same call as 16. Two teams stays simpler. |
| 18 | **3 x 6 fixed rotation blocks** | 9v9 two teams | uncapped winner-stays | Clean split. Each team keeps one sub, so 55.6% each, identical to 9v9. Blocks win on tracking cost. |
| 19 | 10v9 two fixed teams | 3 teams of 7/6/6 | winner-stays | Two teams is simpler at this count. |
| 20 | **4 x 5 round-robin, 7-min fixtures, 2 full rounds** | 3 teams of 7/7/6 blocks; 10v10 | winner-stays (3:1 spread with 4 teams) | Exactly 50% each and a real table. 10v10 gives the same 50% but half the squad sits and the sub-tracking load is heavy. |

Two things worth flagging to the community from this table:

At 18 and 20 heads, average court time is the same whichever format you pick. The choice is entirely about even distribution and tracking cost, not about getting more football. People often assume more teams means less playing time. It does not. The court is the constraint, not the format.

Winner-stays never wins a row outright. It is a good format for competitiveness and a bad one for a minutes-based bill. If the community keeps it, cap the reign at 2 or 3 consecutive wins and the billing spread mostly goes away.

## What is unresolved

These need a decision from the community. Research will not settle them.

1. **Cap on consecutive wins under winner-stays.** BYU says 2, Purdue says 3, TOCA effectively enforces 2. This single number moves the billing spread more than anything else in this document, from a 2:1 ratio down to roughly 1.2:1. Pick one.
2. **Game boundary for winner-stays.** First-to-one-goal, as Tucson runs it, has no bound on a single game and therefore no predictable total play time, which means no predictable per-minute rate. Purdue's "two goals or 7 minutes" is the standard fix but the community has to accept a clock.
3. **Draws.** No rulebook I found says what happens when a *timed* king-of-the-court game finishes level. The options are king stays, both teams off, or next goal wins. Community Kickabout reportedly ends every game on next goal wins, but their rules page returned 403 so I could not verify it and I am not citing it as established.
4. **Whether changeover and warm-up minutes are billed.** About 10% of the court fee covers time when nobody is on court, under any rotation format. Either the per-minute rate absorbs it or the equal-share floor does. Two fixed teams cuts this to about 2%, which is a real and underappreciated argument for that format.
5. **Keeper weight, and whether the keeper rotates within a game.** A volunteer keeper who plays the whole session is one discounted line item. Footy Addicts' standard host instruction is "Everyone goes in goal. Rotation every (number) minutes" (convention, [Footy Addicts host guide](https://blog.footyaddicts.com/organise-football-games/)), which turns keeper time into many short intervals nested inside each player's on-court interval. That is a second timer per player and a lot more taps. Which model applies here changes the data design.
6. **Lending players.** When someone leaves, does the resting team lend a player, and do the lent minutes bill to the lender or to the borrowing team's account? The team-based formats need an answer, two fixed teams does not.
7. **Mid-session reshuffles.** Some operators rearrange teams when one side is dominating. That kills a round-robin table and complicates per-team minute attribution. Decide whether the app supports it at all.
8. **Whether 4v4 is acceptable at 8 or 9 heads**, given it shrinks the billable pot by 20% while the court fee stays flat.

## Where the sources are thin

Being straight about this rather than smoothing it over.

- **No Indonesian-language documentation of amateur futsal rotation convention exists that I could find.** I searched in Indonesian for community rotation systems, three-team setups, and losing-team-off conventions. Results were tournament regulations, coaching articles, and court rental pricing. Nothing describes how a *main bareng* group actually rotates. This is undocumented oral practice. **The app should ask this community what it does rather than inherit a convention from US and UK pickup operators, whose sessions are shorter, refereed by a paid host, and run on booked turf rather than a 2-hour futsal court.**
- **The community-practice side rests on operator documentation and university rec rulebooks**, not on player conversation. Reddit and similar forums were not reachable from my tooling. TOCA, Tucson Soccer Center, Sons of Pitches, Footy Addicts, OpenSports, Purdue and BYU are all credible about what they run, but they are all describing organised drop-in with a paid host, which is not quite the same animal as a fixed friend group renting a court.
- **Nobody has measured court-time distribution under winner-stays.** Every ratio in the winner-stays section is my own arithmetic from the rotation rules. The fact that two independent rec departments felt the need to cap consecutive wins is the strongest empirical evidence I have that the imbalance is real.
- **The one measured distribution I found is from professional two-team futsal** with a coach managing substitutions (Dos-Santos et al. 2020). Treat its ±9 minute spread as a floor on how uneven pickup rolling subs will be, not an estimate.
- **The FIFA Futsal Laws of the Game 2024/25 PDF** is at [digitalhub.fifa.com](https://digitalhub.fifa.com/m/7b1da24ec7a25f67/original/Futsal-Laws-of-the-Game-2024-2025.pdf). I read Laws 3 and 7 through the FA's published text of the same Laws, which matches. If an exact clause ever needs quoting in the app, go to the FIFA PDF.

## Sources

Codified rules:

- [FA Futsal Law 3: The Number of Players](https://www.thefa.com/football-rules-governance/lawsandrules/laws/futsal/law-3---the-number-of-players)
- [FA Futsal Law 7: The Duration of the Match](https://www.thefa.com/football-rules-governance/lawsandrules/laws/futsal/law-7---the-duration-of-the-match)
- [FIFA Futsal Laws of the Game 2024/25 (PDF)](https://digitalhub.fifa.com/m/7b1da24ec7a25f67/original/Futsal-Laws-of-the-Game-2024-2025.pdf)
- [U.S. Futsal, Rules of the Game Summary](https://futsal.com/rules-of-the-game-summary/)
- [FIFA, A crash course in futsal rules](https://inside.fifa.com/news/a-crash-course-in-futsal-rules)

League and operator rulebooks:

- [Futsal NH league rules](https://futsalnh.com/futsal-rules/)
- [FLM System, What is futsal: rules, format and how to run a league](https://www.flmsystem.com/blog/what-is-futsal-rules-and-how-to-start-a-league)
- [Purdue RecWell challenge court rules](https://purdue.edu/recwell/visit/facility/policies/challenge-rules.php)
- [BYU Student Wellness open play rules](https://studentwellness.byu.edu/open-play)
- [Tucson Soccer Center pickup rules](https://www.tucsonsoccercenter.com/pickup)
- [TOCA adult pickup soccer format](https://www.tocafootball.com/adult-pickup-soccer)
- [Sons of Pitches FC pickup listing (OpenSports)](https://opensports.net/posts/thurs-pickup-soccer-rotating-teams-turf--Gye-OzmN)
- [OpenSports, Top 5 concerns when organizing a pickup game](https://opensports.net/blog/top-5-concerns-everyone-has-when-organizing-a-pickup-game)
- [Footy Addicts host guide](https://blog.footyaddicts.com/organise-football-games/)
- [Socceroof, The unwritten rules of pickup soccer games](https://www.socceroof.com/en/blog/unwritten-rules-pickup-soccer/)

Measured data:

- Dos-Santos JW et al. (2020), Physiology Responses and Players' Stay on the Court During a Futsal Match: A Case Study With Professional Players. *Front Psychol* 11:620108. [doi:10.3389/fpsyg.2020.620108](https://pmc.ncbi.nlm.nih.gov/articles/PMC7767871/)
