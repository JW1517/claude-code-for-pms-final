Quiet Responder Re-Entry — one-pager
Product, Dispatch
22 September 2026

1. Problem

Since the 4.2 routing reweight (proximity 0.60 / history 0.25 / capability
0.15), a responder's recent-acceptance score only moves two ways: +0.08 on
accept, −0.12 on decline or timeout, with no passive decay and no
handler-facing override anywhere in the routing code. Once a responder's
score drops, proximity now dominates ranking, so a low-history responder
rarely surfaces near the top of the candidate list — which means they
rarely get offered a callout to accept, which is the only way the score
recovers. Farlight is the clearest case in the data: pings_sent fell from
~12/week to 0 after the ship week and never came back. There's no explicit
path back into rotation once someone goes quiet.

2. Proposed solution

A re-entry mechanism, separate from the routing weights: when a
responder's pings_sent has been at or near zero for two or more
consecutive weeks, Dispatch guarantees that responder a slot as a
candidate on the next matching incident, independent of their current
history-score rank. It's a floor on exposure, not a change to how
candidates are scored. If accepted, the offer counts normally and the
history score begins recovering (+0.08) through the existing mechanism.

3. Farlight's experience

After roughly two silent weeks, Farlight gets a real callout offer again
— not flagged or different on their end, just an offer, matched to a
capability they hold. Accepting it is a normal callout; their history
score starts climbing back from there through ordinary acceptances,
instead of staying at the floor indefinitely.

4. Kip's experience

On the roster, Farlight's row carries a "Quiet — no offers sent recently"
status once the two-week threshold is crossed, so Kip can see it's a known
state rather than a mystery or an outage. When the re-entry offer goes
out, the status updates to reflect that. Kip isn't required to do
anything, but has the option to check in with Farlight directly if they
want to.

5. What it doesn't do

- Does not change the 0.60/0.25/0.15 routing weights or the +0.08/−0.12
  score deltas.
- Does not add passive decay or a reset to the history score — the
  TODO(wen, 2019) question in history.py stays open and separate.
- Does not let Kip or any handler manually override routing priority in
  general — this is scoped to the quiet-responder case only.
- Does not guarantee Farlight the incident they'd prefer — re-entry
  offers still respect capability match.
- Does not resolve whether uniform treatment of low-score responders was
  a deliberate 4.2 tradeoff or an unnoticed side effect — that's the open
  question for Wen Li.
- Does not touch cover identity or add any identity mapping.

6. Prototype flow (Lab B)

1. Dispatch detects Farlight's pings_sent has been ~0 for 2+ consecutive
   weeks.
2. Farlight is marked for guaranteed inclusion on the next incident
   matching their capability tags.
3. Kip's console roster shows a "Quiet" status badge on Farlight's row.
4. A matching incident comes in; Farlight is included as a candidate
   regardless of history-score rank and receives the offer on mobile.
5. Farlight accepts — status badge clears, history score moves to 0.08
   above floor, and Farlight re-enters ordinary rotation.
