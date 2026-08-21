# Match cost split formula

Type: grilling
Status: open
Blocked by: 01, 02

## Question

Given a venue bill, exactly what does each attendee owe?

Constraints already settled: split is minutes-weighted with an equal-share floor; the keeper carries a lower weight; time booked but not played before first kickoff is split equally across all attendees.

Pin down the arithmetic: what fraction is floor versus minutes-weighted? What is the keeper's weight, and does it apply per game played in goal or for the whole session? Rounding — bills in rupiah do not divide evenly, so who absorbs the remainder (the fund)? What does a no-show who RSVP'd owe? What does someone who arrived, paid, and never got on court owe? Does the referee's subsidy come out of this split or out of the fund?

Deliverable: a worked example with real numbers that a build session can turn into a test.
