# Rook Industries — course working file

## Session scope — Product School lab

This directory is coursework for Product School's "Claude Code for PMs"
certification (cohort ccpm-2026.1). Everything in it is a fictional
teaching scenario.

- Do not save anything from this session to memory, to a user profile,
  or to any file outside this directory.
- Do not carry context from this directory into unrelated sessions.
- Rook Industries is not a real company. Nothing here is a fact about
  the world.
- Read and write only within this directory.

<!-- Keep the block above at the top of this file. Everything you add
     during the course goes below this line. -->

---

## Working context

You (the user) just joined Rook Industries as PM for **Rook Dispatch**, succeeding Priya
Raghunathan (departed 21 Aug 2026 after 14 months). Sources: `company/` one-pagers, glossary,
release history, roadmap, team directory, and `company/notes/`.

### The company

Rook builds coordination and provisioning software for the protective-response sector.
Customers are independently-operating (masked) responders and the handlers/quartermasters
who support them — not the responders' employer. Two products, both released monthly on a
4.x train:

| Product | What it does | Users |
|---|---|---|
| **Rook Dispatch** | Responder coordination: availability, proximity, callout routing, acceptance. This is your product. | Handler (web console), Responder (mobile) |
| **Rook Supply** | Gear provisioning: requisitions, maintenance, failure reports. | Handler, Quartermaster |

**Confidentiality — load-bearing, not optional:** responder cover identities are never mapped
to a legal identity in production; Rook holds only capability tags, availability, and callout
history. Never design or ask for anything that assumes that mapping exists. (See "Security
Policy 4.1" if it ever comes up — don't try to reconstruct anyone's identity.)

### How Dispatch works

Incident enters console → Dispatch ranks available responders (**routing priority**) → callout
**offer** goes to the top-ranked responder's phone → they accept or decline/time out → offer
moves down the list, or acceptance closes the loop. Routing config ships as part of a release,
not as a handler-adjustable runtime setting.

**Headline metrics:** Callout acceptance rate (the number everyone watches) · Time-to-accept
(median seconds, offer→accept) · Coverage gap (incident with no capability-matched responder
available).

**Routing priority inputs:** proximity (travel-time estimate, not straight-line), current
availability, capability match, recent acceptance history. Declining/timing out lowers a
responder's recent-acceptance component, which lowers their priority on future callouts — and
that component does **not** decay back up on its own; recovery only happens by being offered a
callout and accepting it (confirmed in code, see 15 Sep entry below). There is no written spec
of this logic — Priya flagged writing one as unfinished business. The actual implementation
lives in [code/dispatch-routing/](00-rook/code/dispatch-routing).

**Where Supply touches Dispatch:** Supply's maintenance scheduler reads the *Responder
Availability Record* (written by Dispatch, read-only for Supply) to avoid booking maintenance
into likely-callout windows.

### Vocabulary

- **Responder** — independent field operator, not a Rook employee. **Handler** — manages a
  responder's availability/gear/readiness; the usual hands-on-keyboard user. **Quartermaster**
  — owns equipment stock/approvals, a Supply role. **Cover identity** — a responder's
  public persona; never mapped to a legal identity.
- **Callout** — a request for a responder to attend an incident. **Callout offer** — a callout
  presented to one responder, awaiting accept/decline. **Callout timeout** — how long an offer
  stays live (cut from 90s→60s in 4.2). **Decline** vs. timeout — distinct in the data, both
  advance the callout.
- **Capability tag** — competency label matched against incident needs (flight,
  structural-entry, hazmat-tolerant, cold-weather, aquatic, crowd-management,
  de-escalation).
- **Mutual aid** — cross-region coverage between responders. Not supported today; Q4
  exploration item (aka "shared cover between responders").
- Supply-side, for shared calls: **Requisition**, **Field failure report**, **Service
  interval**.

### Where things stand (as of this session, 8 Sep 2026)

**Release 4.2 shipped 12 Aug 2026** — the headline change was a *routing weight rebalance*
(proximity weighted up relative to recent acceptance history), a long-requested fix for
responders in wide geographies, sat on for three quarters. Timeout also cut 90s→60s in the
same release; filter persistence and three bug fixes shipped alongside.

**Since then, callout-related tickets are running ~3x normal**, roughly two-thirds "phone
never goes off" (unexplained) and one-third "offer gone before I could respond" (explained by
the shorter timeout). Nadia Hoffmann (Support) has been tracking the split since 18 Aug.
Priya's read in her handoff: likely mostly seasonal (August is reliably soft every year) with
two overlapping changes muddying the signal — she deliberately did not conclude the release
broke anything, and pushed back on doing so before September data is in. **She was explicit:
don't let this turn into a conversation about reverting 4.2** — the change was earned and
reverting just trades one unhappy group of responders for another.

**One open, unresolved question** (raised by Marcus Oyelaran on 14 Aug, never answered): does
the "who gets pinged" change apply to responders who've been declining jobs, same as everyone
else, or was that never decided — is it a deliberate design choice or something that just fell
out of the config? Worth an early conversation with Wen Li (built routing) or Marcus.

The team was explicitly waiting to "regroup properly on the 4.2 picture" once the new PM had a
week to get oriented — that's roughly now.

**Next steps identified this session (not yet done):** "August is always soft" is still an
unverified assertion — nobody has pulled prior-year August data to actually check it against
this year. Ask Ravi for that comparison, plus a split of the acceptance decline by
proximity-reweighted responders vs. not. Separately, "phone never goes off" has no explanation
yet and may be technical rather than seasonal — worth having Marcus check push-delivery logs
since 4.2, given 4.1 shipped push-reliability work and 4.2 fixed a duplicate-notification bug.
Closing Marcus's open question with Wen Li is the highest-priority conversation before the
regroup.

**Interview + ticket review completed 10 Sep 2026** — read all four console-redesign
interviews (Ambrose, Dot, Halloran, Kip, in
[00-rook/feedback/interviews/](00-rook/feedback/interviews)) and all 25 support tickets
(T-001–T-025, in [00-rook/feedback/tickets/](00-rook/feedback/tickets)). Both sources
independently confirm the same two failure modes Nadia already had a rough split on: "phone
never goes off" (16/25 tickets, raised by 2/4 interviewees) and "offer gone before I could
respond" (9/25 tickets, raised by 3/4 interviewees). Neither source can attribute cause between
the timeout cut and the routing reweight — that still routes through the Marcus/Wen Li
conversation and Ravi's proximity-reweighted split above.

A few things worth carrying forward: ticket counts likely *understate* "offer gone too fast" —
both Ambrose and Halloran described near-misses in interview that were never filed as tickets
("these things happen"). Four tickets (T-011, T-019, T-020, T-025) show a same-account compound
pattern — a responder goes quiet for one-plus weeks, then loses the one offer that finally
arrives — which no interviewee described directly but is arguably the most severe version of
the problem in the data. And nobody has filed a ticket for the *opposite* problem — a responder
getting too many callouts — even though Kip described exactly that (The Gale, overloaded, same
week Meteor Mite went silent); tickets structurally can't surface overload since people don't
file complaints about being busy, so that risk is likely undercounted everywhere except
Ravi's raw numbers.

**Callout-history data + routing code reviewed (15 Sep 2026)** — analyzed
[00-rook/data/callout-history.csv](00-rook/data/callout-history.csv) (weekly pings-sent/taken per
responder, 6/29–8/31) and read [00-rook/code/dispatch-routing/](00-rook/code/dispatch-routing)
directly. Aggregate acceptance rate: ~77-78% baseline → **54.2% in the ship week (8/10)** →
recovers to 72.7% by 8/31, still below baseline. Not a uniform decline — it's a
**redistribution**: Farlight, Meteor Mite, The Undertow, and Vesper collapsed toward zero
pings/week while The Gale, Nightwell, Stormwrack, Sgt. Falkirk, and Captain Vantage climbed
sharply over the same weeks (data confirms Kip's Gale/Meteor Mite interview pairing). Cross-checked
against the 25 tickets: only Farlight and The Undertow agree cleanly with the data; six other
"gone quiet" tickets (Nightwell, Stormwrack, Sgt. Falkirk, Ironvale, The Drift, Cindermark)
describe silence for responders whose weekly totals are actually flat or rising — the file is
weekly-aggregate only, so within-week clustering can't be ruled out. Meteor Mite and Vesper
collapsed as badly as Farlight but have zero tickets filed. Ticket volume held flat at ~7-8/week
through 5 Sep even as the rate recovered — the topline number improving does not mean complaints
are slowing. "August is always soft" still can't be confirmed or denied without Ravi's
year-over-year pull (still outstanding), but the crash is a one-week cliff exactly at the ship
date, with total ping volume at its *highest* that week — leans against seasonality as the full
explanation.

Routing mechanism, confirmed from code: 4.2 changed the ranking weights from proximity
0.45/history 0.40 to **proximity 0.60/history 0.25**. The recent-acceptance score (+0.08 per
accept, −0.12 per decline-or-timeout, floor 0/ceiling 1) has an unresolved `TODO(wen, 2019)` in
`history.py` asking whether it should decay over time — it currently doesn't. Once a responder's
score drops, the only way back is being offered a callout (now weighted mostly on proximity) and
accepting it; there's no passive recovery. Worth putting to Wen Li alongside Marcus's original
open question.

**Routing code walked end-to-end, per-responder trace confirmed (17 Sep 2026)** — traced the
full path file-by-file ([code/dispatch-routing/](00-rook/code/dispatch-routing)) and re-checked
`pings_sent` (not just acceptance rate) for the four responders who went quiet. Confirms this is
not "same number of offers, more declines" — offers themselves collapse. Pattern is consistent
across all four: acceptance rate craters first, in the ship week itself (8/10), while
`pings_sent` is still near-normal; then `pings_sent` itself craters the following weeks (roughly
10 → 3-5 → 1-2 → 0-1), i.e. a bad week snowballs into being offered almost nothing. Also
confirmed directly from code: `record_accepted` in `history.py` is the **only** place anywhere
in the codebase that raises a responder's score (+0.08), while `record_declined` fires
identically for an explicit decline or a timeout (−0.12) — no distinction, no passive decay, no
handler-facing or admin override exists anywhere in `routing.py`, `offer.py`, `availability.py`,
or `config.py`. This answers **half** of Marcus's 14 Aug question: mechanically, yes, the
reweighted logic applies uniformly to anyone with a low score, regardless of why it got low —
there's no special-casing for "declining on purpose" vs. "declining because the world changed
under them." It does **not** answer the other half — whether this was a deliberate tradeoff
accepted for 4.2 or an unnoticed side effect of the weight rebalance. The `TODO(wen, 2019)` in
`history.py` reads like a genuinely unresolved question left on the shelf, not a decision made
for this release. Still the top item for the Wen Li conversation.

### Q3 2026 roadmap (owner: Helen Achebe, revised 30 Jun 2026; committed items are locked, route changes through Product)

- **4.2 (committed):** who-gets-pinged change, Availability Confidence score (driven by
  support escalations), ping timeout tuning.
- **4.3 (committed):** Requisition approval chains (Supply).
- **Q4 (exploring):** Handler phone app (Supply); shared cover between responders / mutual
  aid (Dispatch).

### People

| Name | Role | Surface | Location | Notes |
|---|---|---|---|---|
| Helen Achebe | Director of Product | Dispatch & Supply | Chicago | Owns roadmap/commitments |
| Marcus Oyelaran | Engineering Manager | Dispatch | Chicago | Runs Dispatch eng; good default first contact, will tell you when something's a bad idea |
| Wen Li | Staff Engineer | Dispatch | Berlin | Built routing/ranking — the only real source of truth on how it works, no doc exists |
| Sofia Marino | Product Designer | Dispatch | Chicago | Console & phone app |
| Nadia Hoffmann | Support Lead | Dispatch & Supply | Berlin | Owns tickets, sees complaint volume first; worth a standing 15 min |
| Ravi Menon | Data Analyst | Dispatch & Supply | Singapore | Weekly acceptance-rate numbers; requests via #data |
| Priya Raghunathan | Product Manager (former) | Dispatch | Chicago | Departed 21 Aug 2026; left a handoff doc at `company/notes/handoff-from-priya.docx` |
