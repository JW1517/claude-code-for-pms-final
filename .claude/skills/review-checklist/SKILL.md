---
name: review-checklist
description: Runs a fixed 4-point check on a brief, proposal, or write-up before it goes further — owner named, success measure stated, scope consistent start-to-end, problem explained before the fix. Use this whenever the user asks to review, check, sanity-check, or "run the checklist on" a brief or doc (e.g. "/review-checklist brief.md", "does this brief pass the checklist", "check this against my usual review"), even if they don't name the four items — they're fixed and always run together, never a subset and never with extra criteria added.
---

# Review checklist

This captures one person's habitual first-pass review of a brief before it's allowed to move forward. The value of the skill is its *consistency* — the same four things, checked the same way, every time — so resist the urge to add a fifth check just because something else looks off in the document. If something else looks wrong, mention it separately, after the checklist, clearly labeled as a side note rather than folded into the four.

## The four checks

Run all four, in this order, against the target document:

1. **Names who owns it.** Look for a single, identifiable owner or approver — a person's name, not "the team," "stakeholders," or a role left blank. A named team is not a pass unless one person within it is clearly accountable.
2. **Says how we'll know it worked.** Look for a stated success metric or signal — something that could actually be measured or observed later. An action item, a to-do, or "we'll monitor it" without a specific signal does not count.
3. **Scope at the end matches scope at the start.** Compare what the document frames itself as addressing in its opening (problem statement, framing paragraph) against what it actually asks for or commits to in its closing (recommendation, next steps, ask). Flag it if the close is narrower, broader, or otherwise different from the open — that drift is the failure mode this check exists to catch, not just "is scope mentioned."
4. **Explains the problem before proposing the fix.** Check ordering, not just presence: evidence/problem description should come before the proposed solution appears. A document that opens with the fix and backfills justification afterward fails this even if all the right content is present somewhere.

## How to review

1. Read the whole target document first — don't grade from a partial read, since check 3 specifically requires comparing the beginning against the end.
2. Go through the four checks in order. For each one, decide Met / Not met, and back it up with a specific quote or pointer from the document — not a general impression. "No owner named" is weaker than "closing section says 'the team will decide'; no individual named anywhere."
3. If a check is partially met (e.g., a metric is implied but never stated outright), call that out explicitly rather than forcing a binary — but still give a working Met/Not met call so the review is actionable.
4. Keep the whole review short. This is a fast pre-flight check, not a rewrite or a full edit of the document — don't rewrite passages, don't propose alternative structures, don't review prose quality. If a check fails, one line pointing at the gap is enough for the author to go fix it themselves.

## Output format

Use this structure:

```
## Review: <document name>

1. Owner named — Met / Not met
   <one-line evidence or gap>

2. Success signal stated — Met / Not met
   <one-line evidence or gap>

3. Scope consistent (start vs. end) — Met / Not met
   <one-line evidence or gap>

4. Problem explained before fix — Met / Not met
   <one-line evidence or gap>

Overall: <N>/4 met
```

If there's something worth flagging outside the four checks, add it after `Overall:` under a clearly separate `Note:` line — never mixed into the four items above.
