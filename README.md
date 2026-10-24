# AbFlow

English · [Français](README.fr.md)

**Self-hosted** file and photo manager, designed to run on a Raspberry Pi (or any small server) at home. Upload, timeline organisation, temporary sharing, and a read-only API to plug in other apps (screensaver, dashboard…) without exposing everything else.

Personal use: no multi-account support for now (see [Roadmap](#roadmap)).

## Features

- **Upload** of several files at once with a real progress bar; every file type is accepted (images, videos, audio, documents, archives…)
- **Timeline** grouped by day, search by name, sorting (date/name/size), filters by type, multi-select with bulk delete
- **Built-in viewer**: image lightbox (mobile swipe, keyboard ← → Esc), video/audio player, PDF preview
- **Duplicate detection** on upload (sha256 fingerprint): a file that already exists is rejected before it is written to disk
- **Metadata**: original file name, MIME type, image dimensions, EXIF on photos (camera model, real shooting date used for day grouping, never the GPS data, even if present in the file)
- **Sharing**: temporary link (24 h by default) to a single file, without giving access to anything else
- **Storage quota**, configurable, with a usage bar in the app
- **Named API keys**, generated and revocable from the interface, to connect an external app in read-only mode, restricted to images
- **PWA**: installable (iOS/Android/desktop), dark theme, mobile-first
- **Storage**: local or remote SFTP, chosen in `.env`
- **Security**: hardened headers (helmet, CSP, HSTS), fixed JWT algorithm, rate limiting on login/upload/sharing, non-root containers

## Stack

| | |
|---|---|
| Frontend | Vue 3 + TypeScript (`<script setup>`), Vite |
| Backend | Node.js + Express |
| Auth | bcrypt + JWT (24 h) + rate limiting on login |
| Deployment | Docker Compose, nginx image for the frontend |
| Tests | `node --test` + supertest (backend) · Vitest + Testing Library (frontend) · Playwright (e2e) |

## Quick start

Requirements: Docker, Docker Compose, Node.js 20+ (for dev mode with hot reload).

```bash
cp .env.example .env   # fill in AUTH_PASSWORD, JWT_SECRET (see comments in the file)
make dev                # backend in Docker + Vite frontend with hot reload
```

Other useful commands (see `make help`):

```bash
make up       # build + start everything in Docker (prod-like, HTTP)
make https    # same, with a self-signed certificate
make status   # services state + URL
make e2e      # isolated stack + Playwright suite
```

The app starts on `http://localhost:8080`. The first account is configured entirely through `.env` (`AUTH_USERNAME` / `AUTH_PASSWORD`): there is no sign-up, it is a single personal account.

## Configuration

Everything is configured through `.env` (see `.env.example` for the full, commented list): credentials, JWT secret, allowed CORS origins, storage type (local or SFTP), optional quota, displayed app name.

API keys for external integrations are no longer set through an environment variable: they are generated from the interface (key icon in the header once logged in).

## Deployment

In production the app runs behind [Traefik](https://traefik.io/traefik/) (routing + Let's Encrypt HTTPS) and updates itself through [Watchtower](https://containrrr.dev/watchtower/): every push to `main` that passes CI builds and publishes images to GHCR, which Watchtower detects and redeploys with no manual step. See `docker-compose.prod.yml`.

## Tests

```bash
cd backend  && npm test              # 102 tests (node --test + supertest), ~99% coverage
cd frontend && npm test              # 154 unit tests (Vitest), ~99.8% line coverage
cd frontend && npm run test:coverage # detailed coverage report
cd frontend && npm run typecheck
make e2e                             # critical end-to-end path (Playwright)
```

## Project layout

```
backend/            Express API: auth, upload, storage, sharing, API keys
  storage/          Storage adapters (local, SFTP)
  tests/
frontend/
  src/
    views/           Login, Timeline (files), Upload, Settings (API keys)
    components/      Header, bottom navigation, footer
    composables/     Reusable logic (auth, API calls, stats)
  e2e/               Playwright suite
docker-compose.yml            Development / local use
docker-compose.prod.yml       Production (GHCR images, Traefik, Watchtower)
```

## Roadmap

Not implemented yet: multi-account with storage spaces separated per user, a backup system for maintenance.

## Licence

AGPL-3.0-or-later: see [LICENSE](LICENSE).
