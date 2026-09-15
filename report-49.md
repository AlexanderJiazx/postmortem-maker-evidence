# Frantic #49 — Give runx some love

Support action: filed runxhq/runx#485, a concrete usability issue found while
publishing a real skill with runx-cli 0.9.1.

- `runx login --provider github --for publish --from-gh` fails when the `gh`
  binary is missing, even though a usable GitHub token already exists in
  `GH_TOKEN`/`GITHUB_TOKEN` or the git credential helper.
- The issue records the exact error output, why it blocks minimal agent and
  CI environments, and a concrete fallback order: env vars, then the git
  credential store, then the gh binary as a last resort.
- It was authored from real usage — the failure was reproduced locally and
  worked around with a two-line shim — so a maintainer can act on it directly.

Public artifact: https://github.com/runxhq/runx/issues/485
