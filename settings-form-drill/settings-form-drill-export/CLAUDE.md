# CLAUDE.md

Project rules for this app, learned from the round-1/round-2 comparison in `WORKFLOW.md`.

## Rules

1. **Forms use `react-hook-form` + `zod` via `@hookform/resolvers/zod`. Never uncontrolled
   `useState`-per-field or manual `if` chains in a submit handler.** Round 1's hand-rolled
   validation silently skipped the confirm-password check; a schema makes "what's valid" one
   testable object instead of scattered conditionals a reviewer has to reconstruct by hand.

2. **Every form input must have a programmatically associated label (`htmlFor`/`id`),
   `aria-invalid` when it has an error, and an `aria-describedby` pointing at the error
   element.** This is a lint-able, checkable fact (`getByLabelText` either finds the input or
   it doesn't) — not a vibe. A PR that adds an `<input>` without a matching `<label htmlFor>`
   should fail review.

3. **A feature isn't done until it has a test that would fail if the bug from round 1 came
   back.** Concretely: any field with a cross-field constraint (password === confirmPassword,
   date ranges, etc.) needs an explicit test for the mismatch case, not just a happy-path test.
   "I tried it in the browser and it looked right" is not verification.

4. **Never call `alert()` for form errors or success states.** Use inline, field-level error
   text tied to the field via `aria-describedby`, and a `role="status" aria-live="polite"`
   region for success messages. `alert()` blocks the main thread, is unstyleable, and is a
   known screen-reader trap.

5. **Submit buttons must be disabled while `isSubmitting` is true.** Round 1's button had no
   pending state, so rapid double-clicks fire the submit handler twice; this is a checkable
   condition (`expect(button).toBeDisabled()` during an async submit) not a nice-to-have.
