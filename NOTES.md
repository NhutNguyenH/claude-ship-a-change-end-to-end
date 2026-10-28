# Notes — update a user (`PUT /users/:id`)

## Plan

I ran `npm test` first to see the target: the three `PUT /users/:id` tests (200, 404, 400) were red. Claude read `routes/users.js`, `db/store.js` and the test file, then proposed a three-commit plan: add a store helper, add the route following the existing pattern, then write these notes. I approved it without edits. The only open question was how strict validation should be; I kept it the same as `POST /users` (both `name` and `email` required) because the tests only cover the missing-field case and consistency with the existing route seemed more valuable than inventing new rules.

## Model

This work ran on Claude Sonnet 5.5. The change is small and fully specified by existing tests, and it follows a pattern already present in the codebase, so a larger model would not have added much. The tests act as the safety net if the model gets something wrong.

## Commits

1. `Add updateUser helper to the in-memory store` — data access only. It returns `undefined` for an unknown id so the store stays free of HTTP concerns.
2. `Add PUT /users/:id endpoint to update a user` — the route: 400 for missing fields, 404 for an unknown user, 200 with the updated user otherwise.
3. `Add NOTES.md` — the write-up.

I split it this way so each commit does one thing and can be understood (and reverted) on its own: storage first, then the HTTP layer that depends on it.

## Review

Claude reviewed the diff before pushing. It confirmed that an empty body does not crash (`express.json()` sets `req.body` to `{}`), that an `id` in the request body is ignored so it cannot overwrite a user's id, and that non-numeric ids become `NaN` and fall through to 404. It also pointed out two things I chose not to change: `name`/`email` are not type-checked (a number would be accepted, same as `POST`), and `Number("0x1")` equals 1 so `/users/0x1` matches user 1 (same as `GET /users/:id`). Both are pre-existing behaviours outside this task. `npm test` and `npm run lint` pass.
