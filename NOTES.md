1. I do not want Claude to access any sensitive information, so I added:
   "deny": [
   "Read(./.env)",
   "Read(**/.env)",
   "Edit(./.env)",
   "Bash(cat .env)",
   "Bash(git push --force*)",
   "Bash(git push -f *)",
   "Bash(git push * --force*)",
   "Bash(git push * -f)",
   "Bash(git push * -f *)",
   "Bash(git push * +*)"
   ]
2. I do not want claude to git add or git commit without asking me, so I added:
   "ask": [
   "Bash(git commit *)",
   "Bash(git push *)"
   ],
3. I have added these rules, because they are not obvious and yet important for the project:
If I haven't added the deny rules, the sensitive info could have leaked as well as pushed into repo (both are undesired)

## Commands

- `npm run dev`: start the API with auto-reload on http://localhost:3000 (`PORT` overrides)
- `npm test`: run all tests (Node's built-in `node:test` runner + supertest)
- `node --test tests/users.test.js`: run one test file; add `--test-name-pattern="404"` to run matching tests only
- `npm run lint`: ESLint (`eslint:recommended`, ESLint 8 with `.eslintrc.json`)

CI (`.github/workflows/ci.yml`, Node 22) runs `npm run lint` then `npm test`, so both must pass.

## Conventions

- Use CommonJS (`require` / `module.exports`), not ES modules: ESLint is set to `sourceType: "script"`.
- Write tests with `node:test` and `node:assert`, not Jest or Mocha. Exercise routes through `supertest` against the exported `app`, never a running server.
- Route handlers never touch data directly. All reads and writes go through functions exported from `db/store.js`; add a store function when you need a new one.
- Error responses are JSON shaped `{ "error": "<message>" }` with the right status (400 for validation, 404 for not found). Successful creates return 201.
- Precede each route handler with a one-line comment in the form `// METHOD /path — what it does`.

## Architecture

- `server.js` builds the Express app, mounts each router under its path, and exports `app`. It only calls `listen()` when run directly (`require.main === module`), which lets tests import `app` without opening a port. Keep that guard.
- `routes/` holds one `express.Router()` file per resource. A new resource gets a new file and a matching `app.use("/<resource>", ...)` in `server.js`.
- `db/store.js` stands in for a database: module-level arrays with an incrementing `nextId`. Data resets on restart, and state persists between tests in the same process, so tests must not assume exact counts or IDs.
- Config comes from environment variables (see `.env.example`). Real values go in `.env`, which must never be read or committed.
