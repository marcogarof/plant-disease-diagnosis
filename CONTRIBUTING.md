# Contributing / Git workflow

This project follows **GitHub Flow** (see the course slides on code versioning):

1. Create a branch from `main` for every issue (feature, fix, docs).
2. Add small, related commits with meaningful messages.
3. Open a pull request and request review.
4. Discuss and review; keep the branch up to date with `main`.
5. Deploy/verify, then merge into `main`.

Branch naming: `feat/<short-name>`, `fix/<short-name>`, `docs/<short-name>`.

Commit messages: imperative mood, short summary, details in the body when needed
(e.g. `feat: add confidence-gated abstention to /predict`).

## Before committing

```bash
make lint
make test
```

Never commit raw data, models or secrets: they are handled by DVC and `.env`
respectively.
