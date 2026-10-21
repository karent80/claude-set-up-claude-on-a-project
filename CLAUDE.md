# CLAUDE.md

A small Express REST API (users + health check) backed by an in-memory store, used as the base project for a Claude Code course.

## Commands

- `npm run dev` — start the API with `node --watch` on http://localhost:3000 (`PORT` env var overrides)
- `npm test` — run all tests with the built-in Node test runner (`node --test`)
- `node --test --test-name-pattern="404" tests/users.test.js` — run a single test by name
- `npm run lint` — ESLint (`eslint:recommended`, CommonJS, ES2022)

CI (`.github/workflows/ci.yml`) runs `npm run lint` then `npm test` on Node 22 for every push and PR.

## Conventions

- CommonJS only: use `require` / `module.exports`, not `import` / `export` (ESLint is configured with `sourceType: "script"`).
- Tests use `node:test` + `node:assert` + `supertest`, not Jest or Mocha. Import `app` from `../server` and call `request(app)` — never start a real server in tests.
- Route handlers never touch data directly; all reads/writes go through functions exported from `db/store.js`.
- Errors are returned as JSON `{ "error": "<message>" }` with the matching status (400 for invalid input, 404 for missing records); successful creates return 201.
- Each route has a one-line comment above it in the form `// METHOD /path — what it does`.

## Architecture

- `server.js` builds the Express app, mounts each router under its resource path (`/users`, `/health`), and exports `app`. It only calls `listen()` when run directly (`require.main === module`), which is what lets tests import it.
- `routes/` holds one `express.Router()` file per resource; a new resource means a new file here plus an `app.use()` line in `server.js`.
- `db/store.js` is an in-memory array with an incrementing `nextId`. Data resets on restart, and state is shared across tests in the same process (e.g. a test that POSTs a user changes what later `GET /users` calls return).
- There is no `.env` loader (no `dotenv`); `.env.example` is documentation only, and `.env` is git-ignored.
