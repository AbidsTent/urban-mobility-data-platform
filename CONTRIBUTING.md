# Development and GitHub workflow

## Principles

1. Build one testable unit at a time, following [the phase roadmap](docs/roadmap.md).
2. Before coding, open a GitHub Issue with an ID (for example, `P1-FR02`) and acceptance criteria.
3. Work on a short-lived branch from `main`. Recommended examples: `docs/p0-data-profile`, `feat/p1-downloader`, `test/p4-idempotency`, `fix/p3-date-validation`.
4. Make small, descriptive commits, for example `feat(ingestion): record SHA-256 and source bytes` or `docs(p0): record January 2025 schema observations`.
5. Open a PR to `main` with rationale, test commands and results, and evidence paths. Avoid merging unverified implementation claims.
6. When done, update `README.md` status and `docs/progress.md`; close the issue with supporting evidence.

## Branching, code and data safety

- Keep `main` as the last verified stable state. For a solo project, self-review and merge PRs are fine; do not imply external code reviews.
- Never commit downloaded monthly Parquet files, output Delta tables, API keys, personal credentials or giant logs.
- Tiny manually created test fixtures can be considered later if licensing and repository size permit; store provenance and expected outcomes.
- Data contracts and architecture decisions live in versioned documentation, not only in chat messages.
- Avoid force pushing to `main` unless recovery absolutely requires it.

## Definition of done for an Issue

- [ ] Implementation or documentation meets issue acceptance criteria.
- [ ] Relevant tests/validation executed with actual outcomes recorded (when applicable).
- [ ] Failure modes/limitations documented.
- [ ] README, architecture, data contract, or ADR updated if affected.
- [ ] GitHub PR links requirement IDs and evidence.
