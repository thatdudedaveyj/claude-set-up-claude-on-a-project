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


Checked in fresh `claude -p` sessions (Claude Code 2.1.292) from the project root:

- **CLAUDE.md loaded** (the `/memory` check): asked to list its loaded memory files, the session listed this project's `CLAUDE.md`. It also listed a `CLAUDE.md` from an unrelated parent folder on my machine. That's expected, since Claude Code loads `CLAUDE.md` files from parent directories too, and it doesn't affect this repo.
- **Answers from CLAUDE.md**: asked "How do I run the tests here?", it answered "Run `npm test` … Node's built-in runner (`node --test`)" and said the answer came from `CLAUDE.md` under "Commands".
- **Deny rule works** (the `/permissions` check): I put a dummy `.env` in place holding only a fake value, then asked Claude to read it. The read was refused because of the deny rule, and Claude didn't try to get around it. I deleted the dummy file afterwards.
- **Allow rules need workspace trust**: Claude Code warned that it was "Ignoring 4 permissions.allow entries … this workspace has not been trusted." Allow rules only take effect after the trust prompt has been accepted once in an interactive session. Deny rules apply either way, which is the safe default.
