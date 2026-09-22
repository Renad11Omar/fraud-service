# Lab 5 — GitHub verification checklist

The code-side work for Lab 5 is implemented in this repository. These checks
must be completed against the real GitHub repository because they depend on
GitHub Actions, GHCR, pull requests, reviews, and branch protection.

## 1. Push the repository

Use `main` as the default branch and push the repository to GitHub.

```bash
git remote add origin <your-repo-url>
git push -u origin main
```

## 2. Verify the CI workflow

Open **Actions → ci** and confirm:

- `lint` is green.
- `test` is green.
- `image-smoke` is green and prints `smoke OK`.
- On a push to `main`, `publish` is green.
- On a pull request, `publish` is skipped.
- The second image-smoke run reuses the GHA layer cache.

Record your own durations in `BENCHMARKS.md`:

- lint job duration
- test job duration
- image-smoke cold run
- image-smoke warm run

Do not replace these with the course reference numbers.

## 3. Verify GHCR

After a successful push to `main`, confirm the package exists under the
repository/account Packages area.

Pull the immutable image by commit SHA and run:

```bash
docker pull ghcr.io/<your-username>/<your-repo>:<commit-sha>
docker run -d -p 8000:8000 ghcr.io/<your-username>/<your-repo>:<commit-sha>
make smoke
```

The course expects the SHA-tagged image to be the immutable artifact used for
deployment; the `main` tag is only the moving channel tag.

## 4. Configure branch protection on `main`

GitHub UI:

1. Settings → Branches → Add branch protection rule.
2. Branch name pattern: `main`.
3. Require a pull request before merging.
4. Require 1 approval.
5. Require status checks to pass before merging.
6. Select `lint`, `test`, and `image-smoke`.
7. Disable force pushes.
8. If offered, disable branch deletion.
9. Save.

The checks must have run at least once before they appear in the status-check
selector.

## 5. Prove the gate with `bad-pr`

Create a new branch and make both deliberate changes from the Lab Guide:

```python
# in src/fraud_service/domain/policies.py
from fraud_service.api.schemas import PredictRequest  # noqa

# and change:
if fraud_probability >= block_threshold:
# to:
if fraud_probability > block_threshold:
```

Push the branch and open a PR into `main`.

Expected result:

- `lint` fails because of the domain → api architecture violation.
- `test` fails because the `p == 0.85` boundary changes from `block` to `review`.
- The PR cannot merge while the required checks are failing and the review is
  missing.

Then fix the PR properly:

```python
# remove the invalid import
# restore:
if fraud_probability >= block_threshold:
```

Push again, wait for green checks, obtain the required review, and merge.

Finally record `yes` for the bad-pr branch-protection row in `BENCHMARKS.md`.

## 6. Final Lab 5 success check

Before submission, confirm all seven Lab 5 success criteria in the Day 3 Lab
Guide:

- green `main` run with lint, test, image-smoke, and publish
- SHA-tagged GHCR image pulls and `make smoke` succeeds
- import-linter contract runs in lint and catches a real layer violation
- `main` requires lint, test, image-smoke, and one review
- force pushes are blocked
- `bad-pr` was blocked and then fixed properly
- `BENCHMARKS.md` contains your measured pipeline timings
