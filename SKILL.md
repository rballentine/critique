---
name: critique
description: "Adversarial self-review loop: produce an answer, spawn subagent critic(s) to attack it, revise in response, then surface a three-part summary (initial answer -> what was criticized & changed -> final answer). Use for a red-team / devil's-advocate pass to harden a response before trusting it - works on a fresh prompt, a plan, an analysis, or the previous answer in the session. Also reviews a written document or analysis with claim-testing, fact-checking and newcomer reviewers (see 'Reviewing a document or analysis'). Invoked as /critique (Claude Code) or $critique (Codex), followed by [rounds=N] [task]."
argument-hint: "[rounds=N] [task]"
---

# critique - adversarial review & revise

Harden an answer by attacking it with an isolated critic, then revising. The
critic runs as a **subagent** with a clean context, so its critique isn't
anchored to the reasoning that produced the answer - that independence is the
whole point. Do **not** simulate the critique in your own head and call it done;
spawn a real subagent. If the session has no way to spawn one, say so plainly
instead.

## Arguments

Parse from the invocation text:

- `rounds=N` - maximum critique->revise cycles. **Default 1.** The loop stops
  early if a round surfaces nothing material, so `rounds=3` means "up to 3."
- Everything else is the **task**. If the task is empty, the target is the most
  recent substantive answer you gave earlier in this conversation. If there's no
  task and no prior answer to critique, ask what to critique and stop.

## Procedure

### 1. Initial answer (v0)
- If given a task, actually do it now and write the complete answer.
- If critiquing a prior answer, treat that answer as v0.
- Snapshot v0 verbatim - you'll quote it in the summary. Keep all intermediate
  versions internal and surface them only as step 5 says.

### 2. Critique (one round)
Spawn an adversarial reviewer as a subagent: in Claude Code, the Agent tool
with `subagent_type: general-purpose`; in Codex, its multi-agent tools. The
subagent has **no memory of this conversation**, so the prompt must be fully
self-contained. Give it:

- The original task / goal. When critiquing a prior answer with no task
  supplied, reconstruct this from the conversation - the request that answer was
  responding to - since the subagent cannot see the transcript.
- The current answer (vN) **in full**.
- This mandate: *Be adversarial. Find what is wrong or weak - factual errors,
  unsupported claims, faulty logic, missed edge cases, scope creep, simpler or
  better alternatives. Verify load-bearing claims against the actual
  code, data, files, or primary sources rather than trusting the prose. Cite
  specific locations. Rank findings by severity and separate material flaws
  from nitpicks. End with an explicit verdict: are there material problems,
  yes or no?*
- A hard constraint: **read-only**. Do not edit files, write to a database,
  send anything, place orders, or change anything else; the critic only reads.
  This rests on the prompt alone unless your client offers a read-only agent
  type that can still read and search everything the critic needs to check.

For a large or high-stakes artifact you may spawn 2-3 critics in parallel with
different lenses (e.g. correctness / completeness / alternative approach), so
they run concurrently. Otherwise default to one critic per round to control
cost. Reviewing a document or analysis (below) uses its own set of reviewers.

### 3. Revise
- Fold the valid critiques into a revised answer (vN+1).
- **Push back on bad critiques.** The critic is often wrong, pedantic, or
  inventing problems. Where you reject a point, keep it and note why - don't
  cargo-cult every suggestion.
- Track what changed and what you rejected; you'll report both.

### 4. Loop
- If rounds remain **and** this round raised material issues, return to step 2
  with vN+1 as the new current answer.
- Otherwise stop. Convergence (the critic found nothing material) ends the loop
  even when rounds remain.

### 5. Surface the summary
When you run this as a step of other work rather than on request (for example,
when the project's instructions ask for a critique before hard work is final),
skip this format: the revised work is the deliverable; say in a line what the
critique changed. A calling skill or process that keeps its own record of the
work may ask for what the review changed to be written there instead.

Otherwise output exactly these three sections, in order:

**1. Initial answer** - a tight summary of v0's approach and key claims (the
starting point, before any critique).

**2. Critique & changes** - per round, the material points the critic raised,
and for each one how the answer changed, or why you rejected it. Be concrete; a
reader should see what the critique actually bought.

**3. Final answer** - the fully revised answer (vFinal), in full. This is the
deliverable the user acts on, so do not abbreviate it.

## Reviewing a document or analysis

For a written piece someone will rely on - a research write-up, a memo, a
report, a proposal - run the same loop with the reviewers below instead of a
single critic. A skill or process that calls this one can add its own sources
and rules.

**Whose document.** If you wrote it in this session, revise it as in step 3.
If the user handed you someone else's document, or a file of theirs, don't
rewrite it unless asked: v0 is the document as given, the critique section
lists the findings, and the final section gives the changes you recommend (or
the revised text, if they asked for one). Either way the output is the
three-part summary of step 5.

- **Reviewers are fresh-context subagents.** Give them the draft and its
  evidence (the sources, data and notes it draws on), never your reasoning, so
  they attack what is on the page with no stake in it. Make sure they can reach
  the same sources you used: files, data, the web, any search or data tools.
  They stay read-only.
- **Each round runs three reviewers in parallel:**
  - **Claims test.** It extracts the load-bearing claims, at most 12 so it
    goes deep on the ones the conclusion rests on, including claims taken from
    the author's own notes and from interested parties (a vendor, a company's
    management, the other side of a deal). For each, it looks for
    counter-evidence and rates the claim: holds, weakened, refuted or
    untested, with sources.
  - **Fact-check.** It checks every figure, quote, date and attribution
    against its source. Quotes must be verbatim and correctly attributed;
    nothing may be presented as someone's view that isn't in their own words.
    An illustrative worked example has no source: it checks that the example
    is labelled and that its arithmetic holds.
  - **Newcomer read.** It gets only the document, no evidence or tools, and
    reads as someone new to the field. It lists every term, acronym or metric
    used undefined, even ones it knows; every mechanism stated without how it
    works (how a business earns its money, how a process actually runs); every
    event missing what happened, why, and its effect; any missing worked
    example, size of an effect or dated timeline; anything else a newcomer
    could not follow. Address everything it lists.
- **Later rounds:** a new fresh reviewer critiques the revised version and
  triages it as converged, refine or fatal. Stop at converged.
- **A fatal flaw is reported as fatal,** with the flaw stated, and changes the
  conclusion where you own it. Never drop it quietly.
- **Rounds:** `rounds=N` caps them when given. Otherwise scale to the stakes:
  one round for a short piece, up to three for a full analysis, and for the
  largest add a fourth reviewer focused on completeness (missing alternatives,
  comparables, the opposing case).
