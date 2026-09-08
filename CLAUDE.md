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
responder's recent-acceptance component, which lowers their priority on future callouts until
it recovers. There is no written spec of this logic — Priya flagged writing one as unfinished
business. The actual implementation lives in [code/dispatch-routing/](00-rook/code/dispatch-routing).

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
