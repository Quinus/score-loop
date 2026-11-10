---
name: score-loop
description: Improve something in a loop until an independent reviewer's score reaches a target. You orchestrate; fresh implementer delegates apply fixes each round (in parallel when the fixes are independent) and a fresh scorer delegate scores the work per category through a fixed contract, until every category hits the target. Use when the user asks to "score and improve", "rescore until X", "get the score above N", or to run a review/apply loop with delegates. Works for any subject (design, code, docs, copy, UX, performance) and any target.
argument-hint: "<what to improve> [target, e.g. 8.5] [implementer model] [scorer model]"
user-invocable: true
---

# Score loop

Run a loop with three separate roles until an independent score reaches the target:

`Orchestrator → Implementers (in parallel where possible) → verify → Scorer → Orchestrator → …`

| Role         | Who                          | Job                                                          |
| ------------ | ---------------------------- | ------------------------------------------------------------ |
| Orchestrator | You, in this conversation    | Keeps the loop state, picks the next step, decides when to stop |
| Implementer  | New delegates every round, one per independent batch, in parallel | Applies the fixes it is given, nothing else |
| Scorer       | A new delegate every round   | Scores the current version through the fixed contract below  |

The point of the split: you never hold the implementation context, and you never improvise how scoring works. After a compaction you only need the latest **Loop state** block and this skill to continue.

## Inputs

- **Subject**: what to improve (a screen, a module, a document…).
- **Target**: the score every category must reach (e.g. `8`, `8.5`). "Over 8" means `> 8`.
- **Implementer model**: the model the user names. If they name none, use your own provider's default model.
- **Scorer model**: the model the user names. If they name none, pick a model different from the implementer, so the scorer can find other weak spots.
- **Max rounds**: what the user says, otherwise 5.

Use each model's normal mode. Only use a fast mode if the user asks for it explicitly. Keep the same implementer model, scorer model and categories for the whole loop, so scores stay comparable.

## Setup (once)

1. Pick 3–7 categories that fit the subject. They are fixed for the loop.
2. Decide how to collect evidence (screenshots of the latest build, the diff, files, the rendered page) and how to verify a change (build, tests, run).
3. Post the first **Loop state** block (see below) with round 0.
4. Run the scorer once on the starting version to get a baseline. That is round 0.

## Loop (each round)

1. **Choose fixes.** Take the `fixes` from the latest score, lowest categories first. Drop fixes that are wrong or out of scope, and note why in the state.
2. **Split into batches.** Group the fixes into batches that can be done in parallel: no two batches touch the same file or area, and no batch depends on another's result. Fixes that overlap or depend on each other go in the same batch, in order. One batch is fine when nothing can run in parallel.
3. **Implement.** Start one **new** implementer delegate per batch, all at the same time, each with the implementer brief and only its own batch. Never reuse an earlier implementer; its context should not keep growing. Wait until all of them are done.
4. **Verify.** Check every implementer's report, then run the verification yourself once on the combined result (build, tests). If it fails, start a new implementer with the failure output as its only fix. Do not score a broken version.
5. **Collect evidence** from the current version, not an old build.
6. **Score.** Start a **new** scorer delegate with the scorer contract. Fill in only the `<…>` slots.
7. **Check.** Parse the JSON. If it is missing or invalid, start one new scorer with the same brief. Then check the stop conditions.
8. **Update state.** Post a new **Loop state** block. Then start the next round.

You do not edit the subject yourself, and you never score it yourself.

## Stop conditions

Stop as soon as one of these is true:

- Every `score` ≥ target → target reached.
- Max rounds reached.
- Three rounds in a row without meaningful progress: the lowest score did not rise by at least 0.5 and no category below the target improved.
- Scores swing up and down on the same version (noise). Do not keep changing the subject to chase a noisy score.

## Loop state

The state lives only in this conversation, not on disk. Post this block after setup and after every round. The latest block is the single source of truth. After a compaction, find the latest block and continue from there; do not rebuild state from memory.

````
## Loop state — round <n>
Subject: <what, and where it lives>
Target: <target> in every category · Max rounds: <max>
Implementer: <model> · Scorer: <model>
Categories: <cat1>, <cat2>, <cat3>
Evidence: <how to collect it>
Verify: <command or check>

| Round | <cat1> | <cat2> | <cat3> | Note |
| ----- | ------ | ------ | ------ | ---- |
| 0     | 6      | 7      | 5      | baseline |
| 1     | 7      | 7      | 6.5    | <what changed> |

Open fixes: <the fixes for the next round, grouped per batch, one line each>
Skipped fixes: <fix — reason>
Rounds without progress: <0–3>
Next step: <implement round n+1 | stop: reason>
````

## Implementer brief

````
You are the implementer in a score loop. Apply only the fixes below, nothing else.
Other implementers work on other parts at the same time; only touch the files and areas of your fixes.

Subject: <what it is, and where it lives>
Fixes (in this order):
- <where> — <change>
- <where> — <change>
Constraints: <scope limits, style rules, files not to touch, including those of the other batches>
Verify with: <command or check>

Rules:
- Do not score the work and do not add your own improvements.
- If a fix is wrong or impossible, skip it and say why.
- Run the verification before you finish.

Reply with:
- Applied: <fix — what you changed, file:line>
- Skipped: <fix — reason>
- Verification: <command, pass/fail, short output if it failed>
````

## Scorer contract

This is a fixed interface: same brief, same categories, same scale every round. Only fill in the slots; never change the wording, add context, or tell the scorer about earlier scores or your changes, so it judges without bias.

````
You are a strict reviewer. Score only, do not change anything.

Subject: <what it is, and what it is for>
Evidence (read all of it): <file paths, screenshot paths, diff range>
Categories: <cat1>, <cat2>, <cat3>
Scale: 1–10, where 10 is excellent and best in class. Target: <target> in every category.

Judge only what the evidence shows. If a category cannot be judged from the evidence, give score null.

Reply with only this JSON, no other text:
{
  "scores": [
    {
      "category": "<cat>",
      "score": <1-10 or null>,
      "reason": "<one sentence>",
      "fixes": [
        { "where": "<file:line or screen>", "change": "<one concrete change>" }
      ]
    }
  ]
}
Give "fixes" only for categories below the target. List every fix you find for those, highest impact first.
````

Valid result: every category from the list appears exactly once, `score` is a number from 1 to 10 or `null`, and every category below the target has at least one fix. Anything else counts as invalid.

## Rules

- Three roles, kept apart: you orchestrate, the implementer changes, the scorer scores.
- Every implementer and every scorer is a new delegate.
- Run implementers in parallel when their batches are independent; never let two of them touch the same file or area.
- The scorer contract does not change during a loop.
- Leave out categories that the evidence cannot show (for example motion in still screenshots), or provide evidence that can show them.

## Report

End with the score table per round (rows = rounds, columns = categories), the list of changes that were applied, the skipped fixes with reasons, and why the loop stopped. If the target was not reached, say so plainly and explain why.
