# Open-source contributions — NousResearch/hermes-agent

Contribution record for [@falkoro](https://github.com/falkoro) on
[NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent),
an actively-maintained OSS agent framework (~50 issues/PRs per day, maintainer
`teknium1`). Every fix ships with regression tests; every review claim is backed
by something actually executed (local repro, test run, or wire test) — not by
reading the diff.

_Last updated 2026-07-14._

## Merged fixes (2/2 authored PRs merged)

Both merged by the maintainer with authorship preserved:

- **[#58519](https://github.com/NousResearch/hermes-agent/pull/58519)** —
  `fix(config)`: env-ref snapshot invalidates `load_config()` cache when a
  referenced `${VAR}` changes (fixes #58514). Reporter had misdiagnosed the
  root cause; maintainer: _"Great diagnosis on the real root cause."_
  Commit `e3203e4d8` on `main`.
- **[#58520](https://github.com/NousResearch/hermes-agent/pull/58520)** —
  `fix(agent)`: `_resolve_task_provider_model` adopts
  `auxiliary.<task>.base_url/api_key` on explicit provider arg (fixes #58515).
  Maintainer: _"Clean, conservative fix with a thorough test matrix."_
  Commit `9ad912a79` on `main`.

## Open fixes (all mergeable; sweeper salvageability=high)

- **[#60797](https://github.com/NousResearch/hermes-agent/pull/60797)** —
  `fix(lazy_deps)`: unpin `huggingface-hub` to a range so `hermes update`
  stops downgrading the shared dep under transformers/Hindsight (#60783).
  Automated review: _"clean cherry-pick candidate."_
- **[#61611](https://github.com/NousResearch/hermes-agent/pull/61611)** —
  `fix(hindsight)`: surface the exception chain when the local runtime is
  unavailable, so misconfiguration is diagnosable.
- **[#61613](https://github.com/NousResearch/hermes-agent/pull/61613)** —
  `fix(agent)`: log api_kwargs keys + `llm_request` middleware trace on
  unexpected-kwarg `TypeError` at dispatch (hardening from the #60821 triage).
- **[#61615](https://github.com/NousResearch/hermes-agent/pull/61615)** —
  `fix(gateway)`: Windows planned stops report `PLANNED_STOP` in shutdown
  forensics instead of `signal=UNKNOWN` (#61596).
- **[#60085](https://github.com/NousResearch/hermes-agent/pull/60085)** —
  `fix(tui)`: deliver kanban notify subscriptions to TUI/desktop sessions
  (#59890).

## Notable triage & root-cause work

- **[#60821](https://github.com/NousResearch/hermes-agent/issues/60821)** —
  root-caused an intermittent `TypeError: unexpected keyword 'system'` across
  chat-completion providers to a user-installed plugin injecting an
  Anthropic-shaped `system` field via `llm_request` middleware. Eliminated
  config/overrides/transport from attached logs, predicted the injector class
  from the middleware's replace-whole-request capability; reporter confirmed by
  code + disable-test. Supplied a shape-aware fix. Reporter: _"love you man."_
- **#60774**, **#63129** — close-recommendation triage: reproduced (or
  disproved) the issue's premise empirically before recommending action,
  including a minimal s6-overlay container to disprove a claimed
  restart-persistence mechanism.

## Community reviews (40+ threads)

Non-maintainer reviews across 40+ PR/issue threads. Method: git-protocol diffs
(never the REST budget), duplicate sweeps before endorsing, and an empirical
check of every load-bearing claim. Representative:

- **[#56927](https://github.com/NousResearch/hermes-agent/pull/56927)** — proved
  the SSRF redirect guard on `main` was **dead code** (httpx `response.next_request`
  is `None` inside a response hook, verified with a local 302 server) — the PR
  fixes a live bypass, not an edge case.
- **[#62264](https://github.com/NousResearch/hermes-agent/pull/62264)** — proved
  a proposed `ThreadPoolExecutor` timeout fix didn't actually unblock the caller
  (the `with`-block joins the worker on exit), supplied the `asyncio.wait_for`
  alternative with timing proof; author adopted it and credited the review.
- **[#62241](https://github.com/NousResearch/hermes-agent/pull/62241)** — caught
  a PR bundling three unrelated changes (one duplicating the author's own open
  PR); recommended unbundling.
- Multiple duplicate-arbitration comments (e.g. #62844 vs the canonical #43645)
  comparing PR heads programmatically and crediting the earlier author.

Two authors returned to request re-review after addressing feedback
([#57849](https://github.com/NousResearch/hermes-agent/pull/57849),
[#60897](https://github.com/NousResearch/hermes-agent/pull/60897)); the
maintainer's automated sweeper independently reached the same close/keep verdicts
as several of these reviews.
