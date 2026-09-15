# Frantic #49 — Give runx some love

Support action: filed runxhq/runx#485, a concrete usability issue found while
publishing a real skill with runx-cli 0.9.1.

`runx login --provider github --for publish --from-gh` fails when the `gh`
binary is missing, even though a usable GitHub token already exists in
`GH_TOKEN`/`GITHUB_TOKEN` or the git credential helper. The issue records the
exact error, why it blocks minimal agent/CI environments, and a concrete
fallback order (env vars -> git credential store -> gh binary). It is the kind
of report a maintainer can act on directly.
