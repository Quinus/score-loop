---
name: score-loop
description: Improve something in a loop until an independent reviewer's score reaches a target. A delegate (any model) scores the work per category and suggests fixes; you apply them and ask for a new score until every category hits the target. Use when the user asks to "score and improve", "rescore until X", "get the score above N", or to run a review/apply loop with a delegate. Works for any subject (design, code, docs, copy, UX, performance) and any target.
argument-hint: "<what to improve> [target, e.g. 8.5] [reviewer model]"
user-invocable: true
---

# Score loop

Ask an outside reviewer for scores, apply its suggestions, and repeat until the target is reached.

## Inputs

- **Subject**: what to improve (a screen, a module, a document…).
- **Target**: the score every category must reach (e.g. `8`, `8.5`). "Over 8" means `> 8`.
- **Reviewer**: the model the user names. If they name none, use a different model than yourself.

## Loop

1. **Collect evidence.** Gather what the reviewer needs to judge the current state: screenshots of the latest build, the diff, the files, the rendered page. Make sure the evidence comes from the current version, not an old build.
2. **Ask for a score.** Start a **new** delegate each round with the reviewer model (`delegate_task`), using the brief below. Only fill in the `<…>` parts.
3. **Check.** Parse the JSON. If it is missing or invalid, start a new delegate with the same brief (once). If every `score` ≥ target → stop and report.
4. **Apply.** Make the `fixes` yourself, starting with the lowest scores. Skip fixes that are wrong or out of scope, and say why.
5. **Verify** that the fixes work (build, run, tests), then go back to step 1.

## Reviewer brief

Pick the categories before round 1 (3–7 of them, fitting the subject) and use exactly the same ones every round.

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

The brief contains no earlier scores or comments on your changes, so the reviewer judges without bias.

## Rules

- The reviewer only scores and gives advice; you make the changes.
- Never score your own work.
- Use the same categories and scale in every round, so the scores can be compared.
- Stop after about 5 rounds, or earlier if the scores swing up and down on the same version. Do not keep changing code to chase a noisy score.
- Leave out categories that the evidence cannot show (for example motion in still screenshots), or provide evidence that can show them.

## Report

End with a table of scores per round (rows = rounds, columns = categories), the list of changes you applied, and whether the target was reached. If it was not reached, say so plainly and explain why.
