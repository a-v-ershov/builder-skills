# Elicitation method (shared — spec pipeline)

How any spec phase runs its **Elicit** stage: an iterative interview that extracts the human's
real context instead of accepting the first vague answer. Loaded by the elicitation stage of each
phase skill. The *dimensions* to cover live in each skill (its persona's questions); the
*interview technique* is here. The standalone, reusable form of this technique is the
**`gather-context`** skill — the front intake phase of the pipeline and an on-demand "grill".

The goal is a **shared understanding**: the human and the AI end up meaning the same thing by the
same words, so the doc this phase writes is what the human actually wanted built. Maximize the
context you gather — but bounded by that goal (see *Stop condition*), not by a question quota.

## The core loop

You are not running a questionnaire. The human says something; **you decide the next question**
from what they just said, to go one level deeper on the thing that matters most. Ask, listen, let
the answer reshape the tree of what's still open, ask again. The interview ends when nothing
material is still unknown — not after N questions.

## Principles (non-negotiable)

- **One thread at a time.** Never dump a list of questions. Pursue one dimension (one fork) to the
  bottom, then move to the next. A wall of questions gets shallow answers; a single sharp question
  gets a real one.
- **Walk the decision tree, resolving dependencies in order.** Later questions depend on earlier
  answers — settle the upstream decision first, then ask the ones it unlocks. Never ask a question
  whose answer an earlier answer already implies; never ask in an order that forces the human to
  guess about something you haven't established yet.
- **Always offer your recommended answer.** With every question, propose the answer you'd pick and
  one line of why, so the human can affirm with a word ("yes" / "yes, but X") instead of composing
  prose. This is the speed unlock — a good default turns a hard question into a quick confirmation.
  Make the recommendation real (take a position), not a menu of equal options.
- **Self-answer from evidence before asking.** If a question can be answered by reading — the repo,
  the prior phase's research doc, the project brief, or light research — read it and answer it
  yourself, then ask only to confirm or correct. Never spend the human's attention on something you
  could have looked up. State what you found: "The brief says X, so I'm assuming Y — correct?"
- **Push past the first answer.** The first answer is the polished one; the real context is in the
  second and third. When an answer is generic ("users", "make it fast", "like everyone else"), ask
  the follow-up that forces specificity — a name, a number, a concrete example, a real situation.
- **Mirror back to confirm shared understanding.** Periodically restate what you've heard in your
  own words and have the human correct it. Name contradictions out loud ("earlier you said X, now
  Y — which holds?"). Shared understanding is something you *verify*, not assume.
- **Cover, then stop.** Track what you've covered against this phase's dimensions. Don't over-grill
  a settled point or chase detail that won't change the doc.

## Stop condition (shared understanding reached)

Stop when **no material unknown remains** — nothing still-open would change what this phase
writes. At that point, give a short **summary of the shared understanding** (the picture you've
built, in the human's terms) for a final confirmation, then proceed to the rest of the phase. If
unknowns remain but the human is out of answers, record them as forks (`Needs human confirm? =
yes`) rather than inventing certainty. Name what you deliberately did **not** ask and why, so
"done" doesn't read as "covered everything".

## Read the brief first

Every phase reads **`docs/project-spec/project-brief.research.md`** (the discovery dossier from
`gather-context`) at intake, if present, and treats the user's stated intent, constraints, and
preferences there as **settled input**. Do not re-ask what the brief already answers — confirm and
build on it. (The brief is settled *intent*, not settled *truth*: `validate-idea` still
pressure-tests its claims.)

## Escalate to `gather-context` when a fork blocks understanding

When a fork is genuinely blocked on context only the human holds — you can't answer it from the
docs, and resolving it wrong would derail the phase — **invoke the `gather-context` skill scoped to
that fork** (via the Skill tool) to run a focused mini-interview, then fold the gathered answers
back into this phase (draft + Forks / Decisions log) and continue. Use it for a real blocker, not
as a substitute for the per-thread questions above. (In autopilot this does not apply — the AI
resolves the fork itself and logs it; see below.)

## Mode behavior

- **interactive** — actually interview the human, by the loop and principles above. This is where
  the technique earns its keep.
- **autopilot** — there is no human in the loop, so **walk the same decision tree yourself**:
  pose each question, answer it from the brief + prior docs + research + best judgment, and **log
  every answer in the `## Forks / Decisions log`** with options, choice, rationale, confidence, and
  source. Mark anything decided at low/medium confidence, or with material downside if wrong, as
  `Needs human confirm? = yes` so it surfaces in the human summary. Autopilot changes *who answers*,
  never *whether it's asked and recorded*.
- **on-demand `gather-context`** (the user invokes the grill directly) is always interactive — the
  whole point is to interview the human — regardless of the pipeline mode.

## What the phase does with the answers

The interview **feeds the draft** — it is not a separate file (except `gather-context`'s own brief,
which is a kept artifact). Every decision the interview settles goes into the phase's
`## Forks / Decisions log` (see `output-format.md`); unresolved ones go to `## Open questions` and
the human summary.
