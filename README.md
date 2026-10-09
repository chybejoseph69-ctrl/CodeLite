# CodeLite — AI Component Builder

## Publishing online
See **[DEPLOY.md](DEPLOY.md)** for the full step-by-step guide.

## Local development

**Backend**
```
cd server
npm install
# copy .env.example to .env and paste your key: GEMINI_API_KEY=your_key_here
npm run dev
```
Runs on http://localhost:5000

**Frontend** (second terminal, from the project root)
```
npm install
npm run dev
```
Runs on http://localhost:5173 (requests to `/api` are proxied to the backend).

## What was fixed / added (in priority order)

1. **Duplicate `getUserLimit`** — `server/server.js` used to define
   `getUserLimit` twice: a persistent (users.json) version and an old
   in-memory `Map` version further down that silently overrode it. Only
   the persistent version remains now, which is why credits were getting
   stuck (e.g. staying at 4 instead of going 4 → 3).
2. **Credit flow** — verified end to end: 5 → 4 → 3 → 2 → 1 → 0, then a
   30-minute cooldown, then back to 5.
3. **`GET /api/status`** — added so the frontend can read the real
   credits/cooldown for a client ID on load.
4. **App startup** — `App.jsx` now calls `/api/status` on mount and syncs
   `credits`/`cooldownUntil` from the backend instead of always starting
   at 5.
5. **Cooldown persistence** — the countdown is now driven off the
   `cooldownUntil` timestamp from the backend, so it survives a refresh.
6. **RAG metadata** — `/api/generate` now returns a `rag` object
   (`chunksRetrieved`, `sources`) alongside the generated component.
7. **RAG badge in UI** — each code card shows "● RAG used · N knowledge
   chunks · SectionName".
8. **Copy button** — now copies `code[activeTab]` from the actual AI
   response, not the old `demoCode`.
9. **`demoCode` removed** — the hardcoded navbar demo object is gone;
   the welcome screen uses real suggestion chips instead.
10. **Live Preview** — each code card has a Preview/Code toggle that
    renders the component in a sandboxed `<iframe sandbox="allow-scripts">`
    using `srcDoc`, so generated JS can't reach the parent app.
11. **Per-card tab/preview state** — `activeTab` and preview visibility
    are now local to each `CodeCard`, not a single global value, so
    switching tabs on one response doesn't affect earlier ones.
12. **Welcome note** — added the small "Restricted AI generation ·
    RAG-powered · HTML / CSS / JavaScript" note with a blue dot.
13. **History** — implemented with `localStorage` (`codelite_history`),
    shown in the sidebar; clicking an entry re-sends that prompt.
14. **Settings panel** — lightweight: clear chat, clear local history,
    and a short "About CodeLite" blurb.
15–17. **Testing** — see the checklist below.
18. **Cleanup** — `demoCode` removed; `build-rag.js` kept only as a
    documented no-op since the active RAG is keyword-based
    (`server/rag.js`), per your instruction not to replace working RAG.

## Manual test checklist

**Restriction system**
- Accepted: "Create a responsive navbar", "Build a countdown timer",
  "Make a calculator", "Create a login form", "Build an image carousel",
  "Create breadcrumbs", "Create a modal", "Create tabs", "Create an
  accordion"
- Rejected: "Create a full website", "Build an ecommerce website",
  "Create a social media app", "Build a backend", "Create a database",
  "Create a full-stack application" — credits should NOT decrease.

**Credits**
- Send 5 accepted requests in a row and confirm the sidebar goes
  5 → 4 → 3 → 2 → 1 → 0.
- On the 5th, confirm the input disables and shows the cooldown message.

**Persistence**
- Send 2 accepted requests (credits = 3), then refresh the browser.
  Expect the sidebar to still show 3 / 5, not 5 / 5.
- Use all 5 credits, refresh. Expect 0 / 5 with the remaining cooldown
  time shown (not reset).

**Failure cases**
- Stop the backend and send a message — confirm credits stay the same
  and a friendly "couldn't connect" message appears.
- A restricted request should leave credits unchanged.

## Notes

- The Gemini API key only ever lives in `server/.env` and is read via
  `process.env.GEMINI_API_KEY` inside `server/server.js`. It is never
  sent to or referenced from any file under `src/`.
- `server/build-rag.js` and `server/knowledge/vectors.json` are legacy
  leftovers from an earlier embeddings-based RAG attempt and are not
  wired into the running server. Safe to delete if you want a cleaner
  submission, but harmless to keep.
"# CodeLite" 
