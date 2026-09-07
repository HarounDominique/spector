---
name: spec-corpus-hygiene
description: Audits and repairs a spec corpus for internal truth — citations that still resolve, derived figures that agree with their source, and claims about built state that match the code. Use when asked to "audit the specs", "check the docs are still true", or "clean up the spec corpus"; and after any change that moved specs, decision records, or a nexus status table, since that is exactly when citations and derived counts go stale.
---

# Spec Corpus Hygiene

## Overview

A spec corpus rots the same way code does, except nothing fails to compile when it does. A citation can point at a heading that moved, a count in prose can drift from the table it summarizes, a decision can declare a feature "not built yet" months after it shipped — and every one of these reads as normal prose. Nobody notices until someone acts on the false claim.

Manual review does not converge on this problem. Repeated hygiene passes over the same corpus find real defects and introduce new ones of the same class — a figure declared in two places gets corrected in one, a note about the rot pattern itself lands in a commit that then commits the same rot. The fix is not "review more carefully"; it is running cheap automated checks before any human or agent judgment is spent, then giving the semantic review a closed loop that audits itself.

Complements [documentation-and-adrs](../documentation-and-adrs/SKILL.md), which records a decision once, and the sync protocol in [spec-driven-development](../spec-driven-development/SKILL.md#keeping-a-multi-spec-project-in-sync), which propagates a single module's changes to its citers. This skill is the periodic pass that re-verifies the whole corpus against itself and against the code, catching drift that accumulated across many edits rather than one.

## When to Use

- Asked to audit, review, or "sanity check" a spec corpus
- After any edit that moved a spec, a decision record, or a nexus status table — citations and derived counts may now be stale even though nothing looks wrong
- Before treating a spec's claims about the current implementation as authoritative for a new decision
- Periodically on a long-lived, continuously developed spec corpus, independent of any specific edit

**When NOT to use:** A single spec being written for the first time (nothing to audit yet — see spec-driven-development). A one-line correction to a single known-wrong fact (just fix it; this process is for corpus-wide sweeps).

## Process

### Phase 0 — automated checks first, before spending any judgment

Anything a script can catch, a script should catch. Running semantic review before mechanical checks pass wastes the review on defects a regex would have found, and buries real findings in noise.

1. **Build or confirm the mechanical checks exist**, scoped to what the corpus can drift on:
   - Citation resolution — every internal reference (module id + heading, `file:line`, requirement id, decision id) still resolves to real content.
   - Derived counts — any total, status tally, or "N open items" figure in prose still matches the table or list it summarizes.
   - Structural conventions — the corpus's own formatting rules (heading numbering, table shape, a single strikethrough/status convention) hold everywhere, not just where someone last touched.
2. **Run the checks' own test suite before the checks themselves**, if one exists. A checker with a broken test is a checker whose verdicts are worthless — confirm it still catches the defects it claims to catch before trusting a clean run.
3. **Fix everything the checks report**, one by one, before moving to Phase 1. Never silence a red result by relabeling or excluding — either the citation is genuinely wrong (fix it) or the check is (fix the check).
4. Re-run until green. **Only then** proceed.

If no such checks exist yet for this corpus, build the minimal version before the first pass — even a single script that greps for internal id patterns and confirms each resolves pays for itself immediately. Treat "we don't have that check yet" as a Phase 0 task, not a reason to skip to manual review.

### Phase 1 — parallel read-only semantic audits

Run independent audit passes concurrently; none of them edit anything. Each targets a different class of drift the mechanical checks in Phase 0 cannot catch, because it requires judgment:

- **Self-audit of the last corpus change.** Take the most recent hygiene commit (or the current uncommitted diff) and try to refute it: for every factual claim in its message — every figure, every "fixed", every "now consistent" — verify it against the current tree with a file:line citation, and mark it true or false. This is the highest-yield lane, because it is the only one that audits the auditor, and defect density right after a hygiene pass is measured to be highest in exactly what that pass touched.
- **Referential integrity.** Search for citations, ids, or cross-references that do not resolve: an id referenced but never defined, a heading reference where the target section has no such subheading, a `file:line`-style citation embedded in prose that no automated check watches, two independently-numbered series that collide.
- **Cross-document coherence and built-state accuracy.** Look for two specs (or a spec and a decision record) that order incompatible things about the same fact, even in different vocabulary. Look for prose that describes the product in future tense or as absent when the code and tests already show it built — and the inverse: a spec declaring a requirement satisfied with no passing test as evidence. Look for a decision that excludes or defers something without checking whether it was already built — compare the decision's date against the code's date, not its prose, since a decision that says "deferred" can postdate the code it describes.

Every finding needs its evidence inline: the location of the claim, the location of what contradicts it, literal quotes for both. **A finding without evidence is discarded at triage, no exceptions.** Each lane must verify its own claims directly rather than delegating to further sub-audits — a claim reported as verified before it was actually checked contaminates every downstream decision built on it, and is worse than no claim at all; an unverifiable claim should be reported as such, not filled in with an assumption.

### Phase 2 — triage: fix or register, never guess

Every finding goes to exactly one of two places:

> **Fix what has a verifiable answer in the corpus or the code. Register as an open gap anything that requires a decision that isn't yours to make.**

| Fix it | Register it as a gap |
|---|---|
| A figure that disagrees with the table or count it claims to summarize | A genuine contradiction between two rules, where both are currently in force |
| A citation that's stale, ambiguous, or dangling | A figure whose correct value depends on a decision not yet made |
| A status, version, or level that disagrees with its authoritative record | Anything that requires reinterpreting an already-approved decision |
| Prose describing as future or absent something already built and verified | A change that would break citations from documents you don't have authority to edit |
| A formatting or convention violation | — |

The operative test: *can I write the correction and point at the file that proves it's correct?* If yes, fix it. If the answer starts with "the sensible thing would be", it's a gap — register it wherever this corpus tracks open questions (a backlog file, an issue, a nexus "Open Questions" section), at its correct severity, cross-referenced from the exact clause it affects, and — if the corpus has an authoritative document on the topic — from that document specifically, not only from a draft nobody reads first.

Never resolve a registered gap unilaterally, never edit a rule to match a deviation instead of correcting the deviation, and never downgrade a finding's severity to shrink a count.

### Phase 3 — apply fixes with five points of discipline

Every one of these exists because skipping it produced a real regression in a past pass.

1. **Never bulk-edit a structured element with a pattern-matching tool.** A table row, or an id that shares a prefix with others, is not safely touched by `sed`/`awk`/find-and-replace/whole-file rewrite — a pattern match that hits one row often hits several, and a miscalculated range deletes rows outright. Edit one row or one id at a time, by its exact full text, then re-run the Phase 0 structural and count checks.
2. **A fact stated in N places is corrected in all N, or in none.** Before editing: grep for the old value and for the concept in prose (numbers get spelled out too), write down every site you found, edit them all, then re-grep the old value and confirm zero hits outside historical notes. If one of the N sites can't be corrected yet because it depends on an unresolved decision, correct none of them — register a gap instead. A corpus with the old figure everywhere is auditable; one with two values coexisting is not.
3. **A correction note describes the state after the fix, written after the fix.** Never describe a correction not yet applied. Write it last, and make it specific enough that the next pass can falsify it — how many sites, which ones.
4. **Every figure is verified by counting at the moment of writing it, never inherited** from a previous note, a commit message, or memory.
5. **Editing a cited document is editing every citation that points into it.** If the corpus cites by line number anywhere, adding or removing lines shifts every citation after that point — re-run the citation check after each batch of edits to a heavily-cited file, not once at the end, or you lose track of which edit broke what. If a suggested fix comes from a checker's fuzzy relocation, read the destination before accepting it — it found the right file, not necessarily the right span. Prefer the citation form from [spec-driven-development's Citation Convention](../spec-driven-development/SKILL.md#citation-convention) (module id + heading) specifically because it survives edits that don't rename the heading; if this corpus still cites by line number, that's itself a structural fix worth making (see below).

### Phase 4 — close the pass

1. Automated checks green again, including the citation check even if no citation was touched (point 5 above).
2. Before committing, audit your own diff the same way the self-audit lane in Phase 1 audits a past commit — it's cheaper to refute yourself now than to be the subject of the next pass's self-audit.
3. Fix what that self-audit finds in this same pass. Deferring it to "next time" is how passes chain indefinitely.
4. Track defects this pass itself introduced, from the first one, not reconstructed from memory at the end — a clean automated check run afterward proves the pass didn't break anything the checks look for, not that it broke nothing.
5. Write the commit message last, with counts verified in this pass, and state the stopping metric explicitly: per class, how many were closed, how many were introduced.

## Structural Fixes Beat New Checks

When the same class of defect recurs, prefer removing the possibility over watching for it harder:

- Don't restate a derived figure in prose at all — reference the table or count that holds it. A figure that exists in exactly one place cannot disagree with itself.
- Pick one convention for any recurring notation (strikethrough, status markers) and verify it corpus-wide once, rather than re-deciding it per document.
- Forbid a sub-reference into a section that has no subheadings — it's ambiguous by construction.
- Migrate any citation still made by line number or ad-hoc prose reference to the stable id+heading form, so a checker can watch it at all.
- Require every decision record to declare, next to itself, which derived figures it moves.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "The corpus looks clean, this pass is done" | A clean pass proves this pass, with the angles it covered, found nothing — not that nothing remains. A prior pass reporting clean and a later pass finding a real defect with nothing else changed in between is the expected failure mode, not a fluke. |
| "It's a small fix, I'll skip the re-grep" | The missing re-grep is exactly what leaves a stale value coexisting with the corrected one — it's the cheapest step and the one most often skipped. |
| "I'll note the other N-1 sites for next time" | Noting instead of fixing is how a single pass turns into three chained ones, each introducing what it didn't finish. |
| "This finding is tiny, but I'll open a full audit for it anyway" | Not every finding earns a full re-audit — see Red Flags for the severity split. Chasing every cosmetic finding with a dedicated pass is the same non-convergence problem as never triaging at all. |
| "The checker's suggested citation fix looks right" | It found the right file by a fuzzy match, not necessarily the right span — read the destination before accepting a suggested relocation. |
| "I already know this figure from the last pass" | That's inheriting, not verifying. Count it again, now, against its source. |

## Red Flags

- A hygiene commit whose message asserts a figure or "now consistent" claim that a fresh grep contradicts
- The same fact appearing with two different values anywhere in the corpus
- A citation that resolves to the right file but the wrong section, accepted without reading the destination
- An id reused for two different things because a bulk rename touched more than the one row it was meant for
- A decision record declaring something deferred or excluded when the code implementing it already exists and predates the decision
- A registered gap resolved unilaterally, or a finding's severity downgraded to shrink a count
- Semantic audit lanes run before the mechanical checks are green, or before confirming the mechanical checks' own tests pass
- A finding reported as verified that was actually delegated to a sub-agent whose answer hadn't come back yet

## Verification

Before declaring a pass complete:

- [ ] All Phase 0 automated checks pass, including the citation check, even if nothing citation-related was touched this pass
- [ ] Every fix applied in Phase 3 was verified by counting/grepping at the moment of writing, not inherited from a prior note
- [ ] Every finding fixed or registered carries a `file:location` and literal evidence — nothing was fixed or filed on the strength of a paraphrase
- [ ] Every fact that was corrected was corrected everywhere it appears in the corpus, confirmed by a re-grep showing zero remaining old values outside historical notes
- [ ] The pass audited its own diff (Phase 4) and fixed what that self-audit found, in the same pass
- [ ] Findings were split into blocking (a reader who trusts only this claim reaches a wrong conclusion) versus cosmetic (only visible once you already located the defect) before triage, and only blocking findings reopened the loop
- [ ] The corpus is declared closed only if the pass immediately following the last blocking fix reports zero blocking findings across every lane — not merely that some earlier pass was once clean
- [ ] The commit message states the stopping metric: per defect class, how many closed, how many introduced this pass
