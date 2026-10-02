## What it does

`writer-reviewer-protocol` gets one piece of work written and then polished by independent critics until they stop finding real flaws. A cheap suborchestrator [subagent](https://www.aihero.dev/ai-coding-dictionary/subagent) runs the loop: a **writer** produces the work, one or more **reviewers** score it from 1 to 10, and the writer revises against their feedback until every reviewer scores 9 or higher, or a round cap runs out.

The bar is hidden from everyone doing the work. Neither the writer nor the reviewers are ever told that 9 is the pass mark. A reviewer who knows the bar grades against the bar ("is this a pass?") instead of against the work, and a writer who knows it can leak it into what the reviewers read. Only you, the main [agent](https://www.aihero.dev/ai-coding-dictionary/agent) and the suborchestrator know the number.

## When to reach for it

You invoke this by typing `/writer-reviewer-protocol`, and the agent won't reach for it on its own. Pass the task, and optionally an effort level for the writer and reviewers and how many reviewers you want.

Reach for it when a piece of work matters enough to want more than one pass of independent scrutiny, and you'd rather not be the reviewer on every draft: a doc, a README section, a design write-up, a self-contained piece of code. For a single review of changes you already have, use [code-review](https://aihero.dev/skills-code-review) instead. For a whole spec broken into tickets, use [implement-spec](https://aihero.dev/skills-implement-spec).

## The scale and the hidden bar

Reviewers score on one rubric, and see nothing else about scoring:

| Score | Meaning | What the reviewer owes |
| --- | --- | --- |
| 1-6 | Unacceptable | A detailed account of every problem, why it matters, and what a fix looks like |
| 7 | Acceptable | The flaws worth fixing |
| 8-10 | No major flaws | Any minor issues or polish, anyway |

The pass mark sits above "acceptable" on purpose. A 7 or an 8 still sends the writer back for another round, which is why reviewers list minor issues even when they find no major ones: the writer needs something to act on. With several reviewers, the strictest one decides.

## Three tiers, three efforts

The main agent parses your request and hands off. The suborchestrator only relays and decides, so it runs on the cheapest model. The writer and reviewers run at the effort you ask for:

| You say | Model |
| --- | --- |
| low | haiku |
| medium (the default) | sonnet |
| high | opus |

The writer and the reviewers can differ ("writer medium, reviewers high"). Reviewer effort is the one that matters most: weak reviewers are where the loop breaks down (see below).

## Common questions

**Why did it stop without passing?**
It hit the round cap (5 by default). Scores of 8 mean the reviewers found only minor issues, and a hidden bar of 9 makes that a "not yet" rather than a pass. The report names the best draft so far, which is not always the last one, plus the concerns still open, so you can accept it or run more rounds.

**Which effort should I pick?**
Spend it on the reviewers first. In testing, low-effort reviewers estimated a word count instead of counting it, and missed a wrong example while marking style. A medium writer with high reviewers passed in two rounds on the same task where an all-low run ran out of rounds.

**Why do the scores sometimes go down between rounds?**
Reviewers are fresh each round and never see earlier scores, so each one judges the draft on its own merits. A dip usually means the revision broke something, or a new reviewer caught what the last one missed. Every round's draft is snapshotted, so a worse revision never destroys a better version.

## It's working if

- Scores climb across rounds instead of swinging between extremes.
- Reviews point at specific problems with specific fixes, not general impressions.
- Hard requirements in your task (a length limit, a required section, an exact command) are met in the final draft, and you can confirm each one yourself.
- Nothing the writer or reviewers saw mentions the pass mark or how close the work is to it.
- The report points at a draft that matches the scores it quotes.

## Where it fits

`writer-reviewer-protocol` is a **reach-for-it-anytime standalone**, off the main build flow. Its neighbours are [code-review](https://aihero.dev/skills-code-review), because that is one review pass rather than a write-and-revise loop, and [implement-spec](https://aihero.dev/skills-implement-spec), the other skill that orchestrates subagents, but over a task graph of tickets rather than one piece of work. [ask-matt](https://aihero.dev/skills-ask-matt) routes you over the rest of the set.
