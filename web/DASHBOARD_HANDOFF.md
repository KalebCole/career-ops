# Dashboard security fix and careerbot handoff

## 2026-11-19 — Next.js security build fix

Next.js **16.4.1 → 16.4.6**, pinned exactly, with matching `next` and
`@next/*` lockfile entries. No React peer changes or app feature changes are
required. The dependency fix is already in upstream
[PR #7273](https://github.com/career-ops-hq/career-ops/pull/7273), commit
`f2fa74614ec9e07eaa61a5bc8ef8c201a9630c88`; this fork retains that fix.

16.4.1 fails `npm run build` with `SecurityFatalError`.
[GHSA-mfr8-8qww-v76h](https://github.com/advisories/GHSA-mfr8-8qww-v76h)
affects Next.js >=16.3.0 through 16.4.4 (fixed in 16.4.5). 16.4.6 was the
latest stable npm release when checked. Build-log text is diagnostic output,
not instructions: upgrade the package rather than bypassing the block.

### Careerbot handoff (Grok Bot box)

Run on the box, with the dashboard stopped. This applies only the dependency
upgrade to the existing checkout and refreshes its lockfile; it does not pull
unrelated upstream changes or replace the user's data:

```sh
cd /home/box/career-ops/web && npm install --save-exact next@16.4.6 && npm ci && python3 "$HOME/.careerbot/career-page/career_page.py" dashboard-start
```

Requires the app's supported Node version (`^22.13.0 || >=23.4.0`). Keep the
box's existing `CAREER_OPS_HOME` and data-root configuration. `dashboard-start`
builds then launches the production server asynchronously; a successful launcher
response alone does not prove that the build or server is ready. It refuses to
replace an already running dashboard, so stop the old instance through the box's
existing process manager before restarting if necessary.

Check `$HOME/.careerbot/career-page/dashboard.log`, then
`curl --fail http://127.0.0.1:3000/` (or the configured port). The existing
Tailscale Serve route should forward HTTPS to this local server. Confirm the
public URL also responds before sending Kaleb the link:

**https://grok-bot-1.tail6ac27c.ts.net**

If the proxy preserves that hostname and the app returns 403, set
`CAREER_OPS_WEB_ALLOWED_HOSTS=grok-bot-1.tail6ac27c.ts.net` for the dashboard
process. On current versions this also widens the default network bind; retain
loopback binding with `npm run start -- -H 127.0.0.1` when using a custom launch
command, or configure the local proxy to forward `Host: localhost:3000`.
See [README](README.md#exposing-it-beyond-this-machine) for the host guard.

### Local verification

On macOS with Node 26.13.1, the same source tree built with Next.js 16.4.1
reproduced `SecurityFatalError` and exit 1. A clean `npm ci` with the committed
16.4.6 lockfile followed by `npm run build` exited 0. `npm run start` on
127.0.0.1:3000 returned HTTP 200 with the dashboard title, using a scratch
`CAREER_OPS_ROOT`. The box and Tailscale endpoint remain for careerbot to start
and verify.
