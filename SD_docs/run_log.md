# Run Log — L-02

**Student:** Yael Juliard  **Date:** 2026-10-21
**Fork:** https://github.com/Yavigail/event-photo-capstone (fork of Binah-CS/event-photo-capstone)
**Environment:** Windows 10 (build 22621), cmd + PowerShell, VS Code
**Node version:** v26.8.1   **npm version:** 11.19.0

## 1. Setup and installation (slide 8)

| # | Command | What it does | Result | Worked? |
|---|---|---|---|---|
| 1 | `git clone https://github.com/Yavigail/event-photo-capstone.git` | Local working copy of my fork | 37 objects received | ✅ |
| 2 | `node --version` | Checks Node is installed (README requires 20+) | `v26.8.1` (meets README requirement of 20+) | ✅ |
| 3 | `npm install` | Installs dependencies from package.json / package-lock.json | 160 packages added | ✅ with warnings |
| 4 | `type package.json` | Shows available npm scripts | `dev`, `start`, `test`, `test:watch`, `typecheck` | ✅ |

## 2. Tests and server (slide 9)

| # | Command | What it does | Result | Worked? |
|---|---|---|---|---|
| 5 | `npm test` | Runs vitest once on `tests/` | 5/5 tests passed (vitest 3.2.7) | ✅ |
| 6 | `npm run typecheck` | Runs `tsc --noEmit` — TypeScript type check, no build output | No output, no errors — type check passed | ✅ |
| 7 | `npm run dev` | Starts server with `tsx watch` (restarts on change) | `Event Photo API listening on http://localhost:3000` | ✅ |

## 3. Manual API checks from README (second terminal, cmd, server running)

| # | Command | Response | Worked? |
|---|---|---|---|
| 8 | `curl http://localhost:3000/health` | `{"status":"ok"}` | ✅ |
| 9 | `curl -H "x-photographer-id: studio-amber" http://localhost:3000/api/events` | `[{"id":"event-100","title":"Cohen Wedding","galleryPublished":true}]` — only studio-amber's event | ✅ |
| 10 | `curl -H "x-photographer-id: studio-amber" http://localhost:3000/api/events/event-100/gallery` | `{"id":"event-100","title":"Cohen Wedding","photos":["photo-1","photo-2","photo-3"]}` | ✅ |
| 11 | `curl -H "x-client-id: client-100" http://localhost:3000/client/events/event-100/gallery` | `{"eventId":"event-100","title":"Cohen Wedding",...,"availableServices":[retouching, album-premium]}` — **client gallery shows the title** | ✅ |
| 12 | POST selection (see note) | `{"id":"selection-1",...,"requestedServices":["retouching","print-10x15"],"status":"submitted"}` | ✅ but see finding |

Note: the README's multi-line curl uses `\` and single quotes (bash). In Windows cmd it must be one line with escaped double quotes:
`curl -X POST http://localhost:3000/api/events/event-100/selections -H "content-type: application/json" -H "x-client-id: client-100" -d "{\"photoIds\":[\"photo-1\"],\"requestedServices\":[\"retouching\",\"print-10x15\"]}"`

**Finding:** the POST accepted `print-10x15` although it is `enabled:false` for event-100 (`memory-store.ts` line 15) and does not appear in the client's `availableServices` (check 11). Matches README limitation #4.

## 4. What didn't work / what had to change

- `npm install` reported 3 vulnerabilities (1 moderate, 2 critical). Not fixed — `npm audit fix --force` may cause breaking changes.
- `npm warn install-scripts`: esbuild postinstall not approved. Tests and server still ran.
- README curl examples are bash syntax — had to rewrite the POST for cmd (see note above).
- Running `npm run dev` a second time failed: `Error: listen EADDRINUSE: address already in use :::3000` — the server was already running in the first terminal. Fix: run the server in one terminal only; use the second terminal for curl.
- VS Code opened the new terminal as PowerShell; switched to cmd (`cmd`) so the cmd curl syntax works.
- Stopping the dev server in cmd: Ctrl+C → `Terminate batch job (Y/N)?` → `y`.
- I emptied `SD_docs/run_log.md` by mistake with `type nul >`; restored it with `git checkout -- SD_docs`.
- `.gitignore` contains `SD_docs/`, so `git add .` skipped new files in SD_docs. Fix: `git add -f`.

## 5. Verified vs. not verified

- Verified by running: node version, install, tests (5/5), typecheck, dev server, all 5 README curl checks (each ran twice; second POST returned `selection-2`, the in-memory store keeps selections until restart).
- Not verified: behavior after server restart; cross-tenant checks for every endpoint (README limitation #5) beyond the existing tests.