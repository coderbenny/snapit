# Contributing to Snapit

Thank you for considering a contribution. This document covers everything you need to get started.

## Project Structure

Snapit is split across five repositories. Each has its own README with local setup instructions.

| Repo | What it covers |
|------|---------------|
| [snap_BE](https://github.com/coderbenny/snap_BE) | API, SSE, WebSocket, Celery workers |
| [snap_FE](https://github.com/coderbenny/snap_FE) | Web dashboard and marketing site |
| [snap_PC](https://github.com/coderbenny/snap_PC) | Desktop app (macOS, Windows) |
| [snap_mobile](https://github.com/coderbenny/snap_mobile) | Android app |
| [homebrew-tap](https://github.com/coderbenny/homebrew-tap) | macOS distribution via Homebrew |

## How to Contribute

### Reporting Bugs

Open an issue in the repository where the bug occurs. Include:
- Steps to reproduce
- Expected vs actual behaviour
- Platform and version (e.g. macOS 14.4, app v1.0.0)
- Relevant logs or screenshots

### Suggesting Features

Open an issue with the `enhancement` label and describe:
- The problem you're trying to solve
- Your proposed solution
- Any alternatives you considered

### Submitting a Pull Request

1. Fork the relevant repository
2. Create a branch: `git checkout -b feat/your-feature` or `fix/your-bug`
3. Make your changes and commit with a clear message
4. Open a pull request against `main`

Keep PRs focused — one feature or fix per PR makes review faster.

## Development Setup

### Backend (snap_BE)

```bash
git clone https://github.com/coderbenny/snap_BE.git
cd snap_BE
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements-dev.txt
cp .env.example .env   # configure your local values
flask --app wsgi:app db upgrade
flask --app wsgi:app run --port 5559
```

Tests:
```bash
pytest
```

### Frontend (snap_FE)

```bash
git clone https://github.com/coderbenny/snap_FE.git
cd snap_FE
npm install
cp .env.example .env.local
npm run dev
```

### Desktop (snap_PC)

```bash
git clone https://github.com/coderbenny/snap_PC.git
cd snap_PC
flutter pub get
flutter run -d macos
```

### Mobile (snap_mobile)

```bash
git clone https://github.com/coderbenny/snap_mobile.git
cd snap_mobile
flutter pub get
flutter run
```

## Code Style

- **Backend**: PEP 8, enforced by Ruff (run `ruff check .`)
- **Frontend**: ESLint + Prettier
- **Flutter**: `flutter analyze` must pass with no warnings

## Commit Messages

Use the conventional commit format:

```
feat(scope): short description
fix(scope): short description
chore(scope): short description
```

## Questions?

Open a discussion or file an issue — we're happy to help you find the right place to start.
