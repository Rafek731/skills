---
name: writer-reviewer-protocol
description: Run a task through a writer-reviewer loop. A low-effort suborchestrator has a writer produce the work and blind reviewers score it 1-10 until it clears a hidden bar.
argument-hint: "The task, plus optional effort (low/medium/high) and reviewer count"
disable-model-invocation: true
---

Get a piece of work done by a **writer** and polished until independent **reviewers** stop finding real flaws. Three tiers:

- **Main agent** (you): parses the request, spawns the suborchestrator, reports the result.
- **Suborchestrator**: a low-effort model that runs the loop. It never writes or reviews; it relays and decides.
- **Writer** and **reviewers**: spawned by the suborchestrator at the effort the user asked for.

## The hidden threshold

The work is done when every reviewer scores it **9 or higher**. Only the main agent and the suborchestrator know this number. Never put it, or any hint of it ("aim for a 9", "almost there"), in a writer or reviewer prompt.

The reason: a reviewer who knows the bar grades against the bar instead of against the work, drifting toward "pass" or "fail" rather than an honest number. The writer is kept blind too, because anything the writer knows can leak into the work or its notes, and the reviewers read both.

## The scale

Reviewers get this rubric verbatim, and nothing else about scoring:

- **1-6: unacceptable.** Explain in detail: every problem, where it is, why it matters, and what a fix would look like.
- **7: acceptable.** Usable as-is, but with flaws worth fixing. List them.
- **8-10: no major flaws found.** List any minor issues or polish opportunities anyway; an empty list is fine only if there truly is nothing.

## Step 1: Parse the request (main agent)

From the user's arguments, extract:

- **Task**: what the writer must produce, including any files, constraints, and where the output should live.
- **Effort**: map to a model for the writer and reviewers. low = `haiku`, medium = `sonnet`, high = `opus`. The user may name a model directly, or give the writer and reviewers different efforts. Default: `sonnet` for both.
- **Reviewer count**: default 1.
- **Round cap**: default 5. A round is one write (or revision) plus one review pass.

If the task itself is unclear, ask before spawning anything; a vague task wastes every round downstream. Effort and reviewer count have defaults, so don't ask about those.

Create a **work directory** outside the repo (a temp dir) for drafts and reviews, unless the task names an output location.

## Step 2: Spawn the suborchestrator (main agent)

Spawn one subagent on the cheapest model (`haiku` in Claude Code) with the **suborchestrator brief** below, filled in. Then wait for its report.

If your harness doesn't let subagents spawn subagents, play the suborchestrator yourself instead, following the same brief and keeping the same information barriers.

### Suborchestrator brief

> You run a writer-reviewer loop. You never write or review the work yourself; you relay between agents and decide when to stop.
>
> Task: `<task>`. Work directory: `<dir>`. Writer model: `<model>`. Reviewer model: `<model>`, count `<n>`. Round cap: `<cap>`.
>
> **Threshold: the work passes when every reviewer scores 9 or higher.** This number is for you only. Never pass it, or any hint about it, to the writer or reviewers.
>
> Before round 1, fix the **work path**: one file such as `<dir>/work.md` (pick the extension the task implies), or a folder `<dir>/work/` if the work spans several files. The writer always writes there and nowhere else, so you never have to guess which file is the latest.
>
> Each round:
>
> 1. **Write.** Round 1: spawn the writer with the writer brief. Later rounds: send the same writer the revision brief (resume it if your harness supports messaging an agent; otherwise spawn a fresh writer pointed at the work path and the feedback).
> 2. **Snapshot.** Copy the work path to `<dir>/draft-r<round>` (same extension, or a folder), after the writer replies and before review. Confirm the copy differs from the previous round's snapshot; if it is identical, the writer saved somewhere else, so ask it to fix that before going on.
> 3. **Review.** Spawn `<n>` fresh reviewers in parallel with the reviewer brief, pointed at the **snapshot**, not the work path. Scores then belong to a draft that can't change underneath them. They get no earlier scores or reviews, so each judges the work with fresh eyes. Each saves its review to `<dir>/review-r<round>-<i>.md` and replies with it, first line `SCORE: <n>`.
> 4. **Decide.** If every score is 9 or above, stop. Otherwise, if the round cap is reached, stop. Otherwise, merge all reviewers' feedback (deduplicated, but never softened or dropped) and start the next round. Pass the feedback to the writer without scores and without telling it how close it is.
>
> When you stop, report: the scores per round, the number of rounds, whether the threshold was met, and the path to the **best** draft (highest lowest-reviewer score; the latest on a tie), which is not always the last. Include any concerns its reviewers raised that remain unaddressed.

### Writer brief

> Accomplish this task: `<task>`. Save your work to exactly `<work path>`, nowhere else, and reply with a one-paragraph summary. It will be reviewed independently; you may receive feedback to address.

### Revision brief

> Reviewers left this feedback on your work: `<merged feedback>`. Revise the work in place at `<work path>`.
>
> The task's own requirements (`<task>`) outrank any suggestion: if a suggestion would break one (a length limit, a required count, a format), or two suggestions contradict each other, follow the task and say which suggestion you declined and why. Fix what the feedback identifies without rewriting what nobody criticised. Swinging from one extreme to the other to satisfy the last review is the usual way revisions get worse. Before replying, re-check every hard requirement yourself (count, run, or trace it), since edits often break something that used to be right.

### Reviewer brief

> Review the work at `<path>`, produced for this task: `<task>`. Read it critically and in full.
>
> Check correctness first: trace every example, run any code or command you can, and verify factual claims. A wrong example or a broken requirement matters more than style. Measure hard requirements (word counts, item counts, formats) with a tool rather than estimating; an estimate presented as a fact is a review error.
>
> Score it on this scale: `<rubric verbatim>`. Reply with `SCORE: <n>` on the first line, then your review.

## Step 3: Report (main agent)

Relay the suborchestrator's report to the user: where the best draft is (copy it to the task's output location if one was named), how many rounds it took, the scores, and whether it cleared the bar. If the round cap was hit first, say so plainly and surface the outstanding concerns, so the user can decide whether to run more rounds.
