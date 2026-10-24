# CLAUDE.md

A small Express REST API (users + health check) backed by an in-memory store.

## Commands

- `npm run dev` — start the API with auto-reload on http://localhost:3000 (`PORT` overrides)
- `npm test` — run all tests with Node's built-in runner (`node --test`)
- `npm run lint` — ESLint over the whole repo; CI runs lint then tests on every push/PR, so both must pass

## Conventions

- Use CommonJS (`require` / `module.exports`), not ES modules — ESLint is configured with `sourceType: "script"`.
- Route handlers access data only through functions exported from `db/store.js`; never import or mutate the `users` array directly. Add a new store function when a route needs a new operation.
- Error responses are JSON shaped `{ error: "<message>" }` with a matching status code (400 for invalid input, 404 for missing resources).
- Tests use `node:test` + `node:assert` + `supertest` against the exported `app`; never call `app.listen()` in tests. Add tests to `tests/<resource>.test.js`.
- Store state is shared across tests in a run and resets only on restart, so tests must not depend on exact user counts or IDs created by other tests.

## Architecture

- `server.js` — builds the Express app, mounts routers, exports `app`; only listens when run directly.
- `routes/` — one file per resource exporting an `express.Router()`, mounted in `server.js` at `/<resource>`.
- `db/store.js` — the in-memory data layer standing in for a real database; data is not persisted.
- Config comes from environment variables; `.env` holds real values and is git-ignored — see `.env.example` for the keys.
