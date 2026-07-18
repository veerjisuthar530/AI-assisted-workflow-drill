# WORKFLOW.md — Vague Prompt vs. Precise Prompt

**Task:** a settings form (name, email, password, confirm password, notifications toggle).
Round 1 = one-sentence prompt, accepted as-is (`round1-vague-prompt`). Round 2 = a prompt
with file references, explicit constraints, an example behavior, and a verification step, run
through a plan-then-code loop (`round2-precise-prompt`). Full prompts are in each branch's
`PROMPT.md`. Diff: `git diff round1-vague-prompt round2-precise-prompt` (150 lines changed in
`SettingsForm.tsx` alone, plus a new schema file and a 5-test suite in round 2, none in round 1).

## Correctness

Round 1 has a real bug: the confirm-password field is collected but never checked against
`password`. I caught this by re-reading the submit handler — it only checks `password.length`
and console-logs+alerts on submit, so a user can save `abc12345` / `xyz00000` with no warning.
Round 1 also validates password length in the handler but leaves email format to the browser's
native `type="email"` behavior only, so there's no consistent single source of truth for what
"valid" means. Round 2's zod schema is that single source, is unit-testable independent of the
DOM, and the mismatch case is covered by `SettingsForm.test.tsx` (`blocks submit and flags
confirm-password when passwords do not match`), which fails loudly if the bug ever regresses.

## Accessibility

Round 1's labels aren't associated with inputs (no `htmlFor`/`id`), so a screen reader announces
inputs as unlabeled. Errors surface via `alert()`, which yanks focus and is easy to miss for a
screen-reader user. Round 2 pairs every input with a real `<label htmlFor>`, sets `aria-invalid`
and `aria-describedby` on error, and renders inline `role="alert"` text plus an
`aria-live="polite"` save-status region. The "every input has an accessible name" test enforces
this instead of relying on me remembering to check it visually.

## Edge cases

Round 1: no whitespace trimming on name, no minimum password complexity beyond length, double
submit is possible (button is never disabled), nothing stops an empty form from partially
succeeding client-side. Round 2 handles all of these because I specified them as constraints
up front rather than discovering them after the fact.

## Review effort and time

Round 1 took ~4 minutes to generate and looked done at a glance — the real cost showed up when
I actually tried to break it and found the password bug, which would take another prompt round
and a re-review to fix "for real." Round 2 took longer up front (plan review, schema, tests,
one dependency install pass) but needed zero manual review of runtime behavior — the test suite
was the review. End-to-end, round 2 was faster once you count time spent finding round 1's bug
instead of just writing code, which is the whole point of writing tests as part of the prompt
instead of after.
