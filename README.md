# score-loop

Improve something in a loop until an independent reviewer's score reaches a target. You orchestrate; fresh implementer delegates apply fixes each round (in parallel when the fixes are independent) and a fresh scorer delegate scores the work per category through a fixed contract, until every category hits the target. Use when the user asks to "score and improve", "rescore until X", "get the score above N", or to run a review/apply loop with delegates. Works for any subject (design, code, docs, copy, UX, performance) and any target.

## Install

```bash
npx skills add Quinus/score-loop
```
