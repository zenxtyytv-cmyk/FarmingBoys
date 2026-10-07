# Farming Boys — working notes

## What this repo is
A plain static website: hand-written `.html` files in the repo root, no build step, no
package manager, no backend. Bulma is loaded from a CDN at runtime, so pages need internet
access to look right. All page text is Dutch, except `fortnite-launcher.html` (English).

## Running it in the sandbox
`docker-compose.base44.yml` serves the repo root with `python -m http.server` on host port
3000, bind-mounted read-only at `/site`. Start with:

```
docker compose -f docker-compose.base44.yml up -d
```

The plain `python:3.12-alpine` image is used directly and reads files from disk on every
request, so edits are picked up by a normal page refresh. Do not swap it for nginx: the
sandbox gives the repo root `700 root:root` permissions, which the nginx worker user cannot
read (it answers 403 for every page).

The healthcheck probes `http://127.0.0.1:3000/`, not `localhost`: inside the container
`localhost` resolves to `::1` first and the IPv4-only server refuses that connection, which
reports a perfectly healthy site as unhealthy. There is no live-reload dev server, so call
`reload_preview` (or refresh) after changing a page. No environment variables or secrets are
required, and nothing in the stack reads `BASE44_PREVIEW_MODE`, so no sandbox-specific code
overrides exist.

## Things to know before editing
- Filenames contain spaces and semicolons (`Farming Boys 16;9 - Farming.png`). Keep the exact
  spelling when linking, and URL-encode spaces if you build URLs in JavaScript.
- Every page repeats the same nav bar (green `#A0C213` header) — add new pages to the
  `buttons-container` of `index.html`, `farming-simulator.html`, `minecraft.html` and
  `euro-truck-simulator.html` as well, or the page is unreachable from the site.
- "Login" is client-side decoration only: `login.html` compares passwords hardcoded in the
  page and sets `sessionStorage.loggedIn`. It is not real authentication; don't rely on it to
  protect anything.
- `trein.html` is just a redirect to an external game server.

## Verifying a change
```
curl -s http://localhost:3000/fortnite-launcher.html | head
```
should return the page source. `docker compose -f docker-compose.base44.yml ps` shows the
`web` health status.

## Deploying
GitHub Pages already serves this repo (see `CNAME` → `farmingboys.nl.eu.org`); changes land
on the live site when the branch is merged to the default branch. There is no other
deployment configuration in the repo.
