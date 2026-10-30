# SecureID Shield: front end

Open `index.html` (or run `python -m http.server` here). No build step.

## Structure
- `index.html`: page shell only
- `css/styles.css`: all styling and the light/dark theme tokens
- `js/config.js`: **the one switch**: `SOURCE` ('demo' or 'http'), `BASE_URL`, `getToken`
- `js/api.js`: **all backend calls** plus field mapping (`MAP`)
- `js/core.js`: demo data, state (`S`), helpers, charts, phishing scanner logic
- `js/pages.js`: the 8 page templates (`P.overview`, `P.alerts` ...)
- `js/app.js`: router, drawers, click/input handlers, live stream

Scripts are plain (non-module) and load in the order above.

## Connecting a backend
1. Set `SOURCE:'http'` and `BASE_URL` in `js/config.js`.
2. Allow CORS for this page's origin on the backend.
3. Implement these routes (JSON). If yours differ, edit `js/api.js` only.

| Route | Used by |
|---|---|
| GET /api/alerts, /users, /logs, /keys, /sessions | initial load |
| GET /api/overview -> `{riskTrend:[0-100 x40]}` | Overview chart |
| GET /api/stream (SSE) -> `{type:'alert'|'log'|'risk', data}` | live updates |
| PATCH /api/alerts/:id `{status}` | acknowledge, escalate, resolve |
| PATCH /api/users/:email `{locked}` or `{mfa}` | lock, unlock, reset MFA |
| POST /api/keys/rotate `{name}` | key rotation |
| POST /api/sessions/revoke `{user,device}` | revoke session |
| PUT /api/settings/:key (access, pol, nt, profile) | permissions, policies, settings |
| POST /api/scans | saves phishing scan results |

Field names the backend should return are in `MAP` (e.g. alert: id, title, severity, category, user, ip, createdAt, status).

## Notes
- The UI updates first and saves in the background; a failed save shows a toast but does not roll back.
- Browser `EventSource` cannot send an Authorization header. Use a cookie, a short-lived token in the URL, or swap to a fetch stream.
- Once live, delete the demo data block in `core.js`.
- The phishing scan runs in the browser; `saveScan` only stores the result.
