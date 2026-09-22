# Lab 5 — Implementation & Double-Check Report

## Code-side implementation completed

Implemented directly from the supplied Day 3 Lab Guide:

- `.github/workflows/ci.yml`
  - `pull_request`, `push` to `main`, and `workflow_dispatch`
  - least-privilege workflow permissions
  - concurrency cancellation
  - `lint`, `test`, `image-smoke`, and `publish` jobs
  - pip caching keyed by `requirements.lock`
  - Ruff, import-linter, and mypy checks
  - fast pytest coverage gate at 80%
  - real behavioural-model gate
  - coverage artifact upload with `if: always()`
  - Docker BuildKit/GHA layer cache
  - readiness polling instead of fixed startup sleep
  - immutable SHA image tag for smoke tests
  - GHCR publish restricted to push-to-main
  - SHA + `main` tags on published images
- `payloads/sample.json` added with a valid `PredictRequest` payload.
- `[tool.importlinter]` clean-architecture contract added to `pyproject.toml`.
- Repository branch renamed from `master` to `main` to match the Lab 5 guide.
- `README.md` updated to reflect the completed code-side Lab 5 implementation.
- `LAB5_GITHUB_CHECKLIST.md` added for the GitHub-only verification steps.

## Important project-specific correction

The starter's Lab 4 behavioural test module is marked both `behavioural` and `slow`.
Therefore the literal selector from the Lab 5 guide:

```text
behavioural and not slow
```

selects zero tests in this repository.

The workflow therefore runs:

```text
pytest -m "behavioural" -q --no-cov
```

This is intentional: it makes the behavioural job a real gate against the model
instead of allowing a zero-test selection to pass. The fast `not slow` command
remains the coverage-enforced per-commit gate.

## Local verification performed

- YAML workflow parsed successfully.
- Workflow contains all four required jobs.
- Required triggers, dependencies, GHCR permissions, SHA tags, GHA cache,
  readiness polling, and coverage artifact were checked.
- Python source compilation passed.
- Manual AST architecture check found **0 upward layer violations**.
- `pytest -m "not slow" --cov-fail-under=80` passed:
  **all fast tests passed; branch coverage 98.66%**.
- `pytest -m "behavioural" -q --no-cov` passed:
  **3 behavioural tests passed**.

## Checks that require GitHub/runner state

These cannot be truthfully completed inside a local starter archive without the
user's actual GitHub repository:

- GitHub Actions green run on `main`
- cold/warm GHA cache timings
- GHCR published image and digest
- branch-protection settings
- real `bad-pr` blocked by required checks/review
- final measured timings in `BENCHMARKS.md`

The repository includes the exact verification sequence for these in
`LAB5_GITHUB_CHECKLIST.md`. No course reference timing was fabricated as a
measured result.
