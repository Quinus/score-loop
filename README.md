# score-loop

Improve something in a loop until an independent reviewer's score reaches a target. A delegate (any model) scores the work per category and suggests fixes; you apply them and ask for a new score until every category hits the target. Use when the user asks to "score and improve", "rescore until X", "get the score above N", or to run a review/apply loop with a delegate. Works for any subject (design, code, docs, copy, UX, performance) and any target.

## Install

```bash
npx skills add Quinus/score-loop
```
