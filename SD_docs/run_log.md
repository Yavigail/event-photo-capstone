# Run Log

## 1. Clone
Command: git clone https://github.com/Yavigail/event-photo-capstone.git
Result: Success, 37 objects received.

## 2. Install
Command: npm install
Result: Added 160 packages.
Warnings: 3 vulnerabilities (1 moderate, 2 critical); esbuild install-script warning; new npm version available. Did not run audit fix.

## 3. Tests
Command: npm test (vitest run)
Result: 1 test file, 5 tests passed.
Tests covered: demo photographer identity required, events listed per photographer,
other photographer's event not returned, client can submit selection for own gallery,
client cannot access another client's gallery.

## 4. Run
Command: npm run dev (tsx watch src/server.ts)
Result: "Event Photo API listening on http://localhost:3000"

## Problems
None blocking. Only the npm warnings above.
