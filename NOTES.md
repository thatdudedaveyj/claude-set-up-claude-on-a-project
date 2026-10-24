# Notes

## CLAUDE.md: what's in, what's out

**In:** the three commands I actually run, plus the fact that CI runs lint and tests, so Claude knows both have to pass. Conventions are rules Claude can't safely guess: use CommonJS, go through `db/store.js` for all data access, use a consistent `{ error }` response shape, never call `app.listen()` in tests, and don't let tests depend on each other's state in the shared in-memory store. The architecture section is four lines about where things live and how a new resource gets wired in.

**Out:** endpoint lists and per-route details (Claude can read those in `routes/` faster than it can read a stale summary), install instructions (`npm install` is obvious), quote and semicolon style (ESLint enforces it), anything from `.env`, and course instructions from the README. Those are one-off tasks, not project knowledge.

## Permission rules

- **Allow:** `npm test`, `npm run lint`, `git status`, `git diff`. They're read-only or safe, and I run them constantly, so prompting for them only adds friction.
- **Ask:** `git push` and `npm install`. Pushing is visible to others, and installing pulls third-party code and changes the lockfile, so I want to confirm both each time.
- **Deny:** reading `.env` / `.env.*`, `git push --force` / `-f`, and `git reset --hard`. Without the `.env` rule, Claude could read real secrets into the conversation (and from there into logs or output) just by trying to "check the config". Without the force-push and hard-reset rules, one bad command could overwrite remote history or throw away uncommitted work, with no undo.

## Fixes along the way

The original `.gitignore` had every line indented with spaces. Git treats leading spaces as part of the pattern, so `.env` and `.claude/settings.local.json` were **not** actually ignored (I confirmed this with `git check-ignore`). I removed the indentation. Without this fix, the `.env` deny rule would protect against Claude reading the file but not against anyone committing it.

## Verification

- `/memory`: TODO, confirm `CLAUDE.md` shows as loaded
- `/permissions`: TODO, confirm the allow / ask / deny rules above are listed
- Asked "How do I run the tests here?": TODO, record the answer
