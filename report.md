# Postmortem Maker — Frantic #83 delivery report

## What was shipped

`postmortem-maker` v0.3.0, published to the runx registry as
`alexanderjiazx/postmortem-maker@sha-62ac1d14a135`
(public listing: https://runx.ai/x/alexanderjiazx/postmortem-maker), with the
source package opened as a public PR against runxhq/runx:
https://github.com/runxhq/runx/pull/484.

## What the skill does

The graph is one sealed read -> reason -> publish run:

1. `read-source` (agent-task, allowed_tools: `web.fetch`,
   `data.read_projection`) reads the incident record from a real source at run
   time. `incident_source.kind` selects `web_fetch` (live HTTPS fetch of an
   incident thread), `read_projection` (incident/ticket event-stream
   projection), or `inline` (supplied fragments — the deterministic harness
   path).
2. `digest-source` binds the exact fragment set with `data.digest`.
3. `draft` (agent-task) separates facts from hypotheses into the
   `runx.postmortem.v2` shape: summary, impact, source-cited timeline,
   root_cause (known/suspected/unknown), unknowns, owned action_items.
4. `finalize` (javascript, deterministic) enforces citations verbatim: every
   timeline entry and any non-unknown root cause must name a fragment that
   exists in the read and a quote that appears verbatim in that fragment's
   text. Fabrication refuses the whole run.
5. When the postmortem is publishable and `postmortem_policy.allow_publish`
   is true, `send_plan.status` is `ready` and the delivery branch executes:
   `prepare-delivery` binds the payload, `deliver` performs the sealed outbox
   write (`fs.write`), and `record-delivery` seals `publish_result` with
   `send_plan_status: executed`. Conflicting or incomplete evidence withholds
   the plan and nothing is published.

## Verification

- `runx --version` -> runx-cli 0.9.1 (required >= 0.6.14).
- `runx harness ./skills/postmortem-maker` -> passed, 4 cases, 0 assertion
  errors, before publish.
- `runx add alexanderjiazx/postmortem-maker@sha-62ac1d14a135` -> clean install.
- `runx registry read ...@sha-62ac1d14a135 --json` resolves published metadata
  and digests.
- Dogfood run `run_make_f4356b399b76ec9d` read the live GitHub Status incident
  feed (incident qwdwmtqghpk5, resolved 2026-09-15T11:17Z) via web_fetch,
  sealed a publishable postmortem, executed the sealed-outbox delivery to
  `outbox/postmortems/gh-status-qwdwmtqghpk5.postmortem.json`.
- Receipt `sha256:7a6b0e07ddeb135f4059248be494a4a25dd7198280e2969fb49d31a18b598cf2`
  passes `runx verify` (digest, content address, signature all valid).

## Maintainer-facing gaps worth noting

- The `web_fetch` read path trusts the agent's fragmentation of the fetched
  body; a typed fetch tool with structured incident records would tighten the
  evidence chain further.
- The sealed-outbox transport is a filesystem write — real deployments will
  want to compose a hosted comms transport (send-as or equivalent) for actual
  publication channels.
- `read_projection` was exercised structurally but not against a live
  data-store backend in the dogfood; that path needs a populated incident
  projection to prove end to end.
