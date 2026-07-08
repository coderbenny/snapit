<div align="center">

<img src="https://snapit.ink/icon.png" alt="Snapit" width="80" />

# Snapit

**Universal clipboard sync across all your devices**

Copy on your phone. Paste on your Mac. Instantly.

[![Backend](https://img.shields.io/badge/backend-Flask-black?logo=python&logoColor=white)](https://github.com/coderbenny/snap_BE)
[![Frontend](https://img.shields.io/badge/frontend-Next.js%2014-black?logo=next.js&logoColor=white)](https://github.com/coderbenny/snap_FE)
[![Mobile](https://img.shields.io/badge/mobile-Flutter-black?logo=flutter&logoColor=white)](https://github.com/coderbenny/snap_mobile)
[![Desktop](https://img.shields.io/badge/desktop-Flutter-black?logo=flutter&logoColor=white)](https://github.com/coderbenny/snap_PC)
[![License](https://img.shields.io/badge/license-MIT-black)](LICENSE)
[![Platform](https://img.shields.io/badge/platforms-macOS%20·%20Android%20·%20Web-black)](#)

[**snapit.ink**](https://snapit.ink) · [API Docs](#api) · [Self-Hosting](#self-hosting) · [Contributing](#contributing)

</div>

---

## What is Snapit?

Snapit is a cross-platform clipboard sync system that keeps your clipboard in sync across your Mac, Android phone, and browser in real time. Beyond plain text, it handles rich snippets, team-shared libraries, and direct file transfers between any two of your devices — all end-to-end through a relay server you can self-host.

| | |
|---|---|
| **Copy anywhere, paste everywhere** | Clipboard syncs the moment you copy — no button to press |
| **File transfer** | Send a file from Android to Mac (or vice versa) over a secure WebSocket relay |
| **Team libraries** | Share snippet collections across a team with role-based access |
| **Works offline** | Local SQLite cache on every client; syncs deltas when reconnected |
| **Self-hostable** | The entire backend is open source and ships as a single Docker Compose stack |

---

## Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                            Clients                                    │
│                                                                      │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐  │
│   │  Web App     │  │  Desktop     │  │  Mobile                  │  │
│   │  Next.js 14  │  │  Flutter     │  │  Flutter + Android       │  │
│   │  (Vercel)    │  │  macOS/Win   │  │  Background sync via     │  │
│   └──────┬───────┘  └──────┬───────┘  │  WorkManager + FCM       │  │
│          │                 │          └───────────┬──────────────┘  │
└──────────┼─────────────────┼────────────────────┼─────────────────┘
           │                 │                    │
           └────────────────┬┘                    │
                            │   HTTPS / WSS / SSE │
                            │                     │
              ┌─────────────▼─────────────────────▼──┐
              │           nginx + Cloudflare           │
              │    TLS termination · Real-IP headers   │
              └─────────────────┬──────────────────────┘
                                │
              ┌─────────────────▼──────────────────────┐
              │          Flask API  (Gunicorn)           │
              │                                          │
              │   REST endpoints   /snap/*               │
              │   SSE stream       /events               │
              │   WebSocket relay  /snap/transfer/*      │
              └────────┬──────────────────┬─────────────┘
                       │                  │
            ┌──────────▼──┐     ┌─────────▼──────────┐
            │    MySQL    │     │       Redis          │
            │  (primary   │     │  SSE pub/sub ·      │
            │   store)    │     │  rate-limit ·       │
            └─────────────┘     │  Celery broker      │
                                └─────────┬──────────┘
                                          │
                              ┌───────────▼──────────┐
                              │    Celery Workers     │
                              │  email dispatch ·     │
                              │  subscription jobs    │
                              └─────────┬─────────────┘
                                        │
                         ┌──────────────▼──────────────┐
                         │  Firebase FCM  ·  Resend     │
                         │  push notifications · email  │
                         └─────────────────────────────┘
```

### How a clipboard sync works

1. Client copies text → POST `/snap/sync/push` with the new item
2. Server stores it, then publishes a `sync` event on Redis
3. SSE connections on all other devices receive the event instantly
4. Receiving client pulls the delta → item appears in clipboard history
5. Mobile clients that are backgrounded wake up via **FCM push** then pull on open

### How a file transfer works

1. Sender POSTs `/snap/transfer/start` → server creates a session and broadcasts `transfer_incoming` via SSE to the target device
2. Target opens a WebSocket to `/snap/transfer/<id>/recv`
3. Sender opens a WebSocket to `/snap/transfer/<id>/send` and streams binary chunks
4. Server relays chunks in real time; never writes to disk
5. On completion, server signals EOF sentinel; receiver closes and saves the file

---

## Repositories

| Repo | Role | Language / Framework |
|------|------|----------------------|
| [**snap_BE**](https://github.com/coderbenny/snap_BE) | REST API, SSE event stream, WebSocket file-transfer relay, Celery workers | Python · Flask |
| [**snap_FE**](https://github.com/coderbenny/snap_FE) | Web dashboard, marketing site, billing portal | TypeScript · Next.js 14 |
| [**snap_PC**](https://github.com/coderbenny/snap_PC) | Desktop app — macOS (Homebrew) and Windows | Dart · Flutter |
| [**snap_mobile**](https://github.com/coderbenny/snap_mobile) | Android app with background sync and share-sheet integration | Dart · Flutter |
| [**homebrew-tap**](https://github.com/coderbenny/homebrew-tap) | Homebrew Cask for `brew install --cask snapit` | Ruby · Homebrew |

---

## Tech Stack

### Backend — `snap_BE`

| Layer | Technology |
|-------|-----------|
| Framework | Flask 3 · Gunicorn |
| Database | MySQL · SQLAlchemy 2 · Alembic (migrations) |
| Real-time | Server-Sent Events (SSE) · Redis pub/sub |
| File transfer | WebSocket relay via flask-sock |
| Background jobs | Celery + Redis broker |
| Auth | JWT (PyJWT) · refresh-token rotation |
| Billing | Paystack (subscriptions + webhooks) |
| Push notifications | Firebase Admin SDK (FCM) |
| Email | Resend API · SMTP fallback |
| Rate limiting | Flask-Limiter + Redis |
| Observability | Sentry · structlog |
| Containerisation | Docker · Docker Compose · GHCR |
| Reverse proxy | nginx · Cloudflare |
| CI/CD | GitHub Actions — build → push to GHCR → SSH deploy → migration → health check |

### Frontend — `snap_FE`

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 14 (App Router) · React 18 |
| Language | TypeScript |
| Styling | Tailwind CSS v4 · shadcn/ui · Base UI |
| Auth | iron-session (server-side sessions) |
| HTTP | Axios |
| Real-time | SSE (EventSource) · WebSocket (file transfer) |
| Billing UI | Paystack inline checkout |
| Deployment | Vercel |

### Desktop — `snap_PC`

| Layer | Technology |
|-------|-----------|
| Framework | Flutter (macOS · Windows) |
| Language | Dart |
| State | Riverpod + code generation |
| Navigation | GoRouter |
| HTTP / SSE | Dio |
| Storage | flutter_secure_storage · SharedPreferences |
| Encryption | pointycastle (AES-256-GCM + PBKDF2) |
| Distribution | DMG (macOS, ad-hoc signed) · Homebrew tap · GitHub Releases |
| CI/CD | GitHub Actions — tag push → build DMG → GitHub Release → tap formula auto-update |

### Mobile — `snap_mobile`

| Layer | Technology |
|-------|-----------|
| Framework | Flutter (Android) |
| Language | Dart |
| State | Riverpod + code generation |
| Navigation | GoRouter |
| HTTP / SSE | Dio |
| Local DB | SQLite via sqflite |
| Encryption | pointycastle (AES-256-GCM + PBKDF2) |
| Background sync | WorkManager |
| Push notifications | Firebase Cloud Messaging (FCM) |
| Share sheet | receive_sharing_intent |
| File picking | file_picker |
| Storage | flutter_secure_storage |

---

## Features

### Core
- **Real-time clipboard sync** across all signed-in devices over SSE
- **Clipboard history** with full-text search, local SQLite cache
- **Soft-delete** with delta-pull — bandwidth-efficient, conflict-free
- **Device management** — name, track, and revoke individual devices

### File Transfer
- Peer-to-peer relay over server WebSocket — no file stored server-side
- Progress indicator and timeout handling on both sender and receiver
- Automatic SSE notification to the target device
- Works across all platform combinations (Android ↔ Mac, Mac ↔ Web, etc.)

### Teams
- Create a team and invite members
- Shared snippet libraries with role-based access
- Real-time push to all team members on update

### Billing & Plans
| Plan | Sync | History | Transfer | Teams |
|------|------|---------|----------|-------|
| Free | ✓ | 100 items | ✗ | ✗ |
| Pro | ✓ | Unlimited | ✓ | ✗ |
| Pro AI | ✓ | Unlimited | ✓ | ✗ |
| Team | ✓ | Unlimited | ✓ | ✓ |

### Admin
- User management, plan overrides, broadcast messaging
- Coupon system with percentage and flat discounts
- System health dashboard

---

## Getting Started

### Use the hosted version

Visit **[snapit.ink](https://snapit.ink)**, create an account, then install the client for your platform:

```bash
# macOS
brew tap coderbenny/tap
brew install --cask snapit
```

Android: download the APK from the [latest release](https://github.com/coderbenny/snap_mobile/releases)

---

## Self-Hosting

The backend runs as a single Docker Compose stack. You need:
- A server with Docker and Docker Compose installed
- MySQL and Redis running on the host
- A domain with an SSL certificate (Certbot recommended)

```bash
git clone https://github.com/coderbenny/snap_BE.git
cd snap_BE
cp .env.example .env   # fill in your values
docker compose up -d
```

See the [**snap_BE README**](https://github.com/coderbenny/snap_BE#self-hosting) for the full environment variable reference, nginx config, and migration guide.

---

## Contributing

Contributions are welcome across all parts of the system. Here's how to get oriented:

```
snap_BE      →  Python/Flask — start with app/routes/ and app/services/
snap_FE      →  Next.js — start with the app/ directory
snap_PC      →  Flutter desktop — start with lib/features/
snap_mobile  →  Flutter Android — start with lib/features/
```

**Before you open a PR:**

1. Check the open issues in the relevant repo — your bug/feature may already be tracked
2. For significant changes, open an issue first to discuss the approach
3. Follow the existing code style — each repo has its own linting config
4. Write or update tests where applicable

**Good first issues** are labelled across repos. If you're new to the codebase, the backend is the best place to start — the API is fully documented in [`openapi.yaml`](https://github.com/coderbenny/snap_BE/blob/main/openapi.yaml).

---

## License

MIT — see [LICENSE](LICENSE). Each component repository carries the same license.

---

<div align="center">
  <sub>Built with care · <a href="https://snapit.ink">snapit.ink</a></sub>
</div>
