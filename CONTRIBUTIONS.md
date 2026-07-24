# Open-source contributions — NousResearch/hermes-agent

Contribution record for [@falkoro](https://github.com/falkoro) on
[NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent),
an actively-maintained OSS agent framework (~50 issues/PRs per day, maintainer
`teknium1`). Every fix ships with regression tests; every review claim is backed
by something actually executed (local repro, test run, or wire test) — not by
reading the diff.

_Last updated 2026-07-24._

## Fixes landed on `main` (2)

Both applied to `main` by the maintainer with authorship preserved. Note for
anyone verifying: the maintainer applied these outside GitHub's merge button, so
the PRs read as `CLOSED` with `mergedAt: null` and will **not** appear in a
merged-PR count. The commits themselves are on `main` and are the ground truth —
verify with `git log origin/main --author=falkoro`:

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

## Open fixes (8, all mergeable; maintainer-bot reviews answered same-day)

Kept rebased against a fast-moving `main` (~990 commits in the three days to
2026-07-24). Mergeability is checked locally with
`git merge-tree --write-tree --merge-base` rather than the API's `mergeable`
field, which is computed lazily and reports `UNKNOWN` rather than a conflict.

- **[#67023](https://github.com/NousResearch/hermes-agent/pull/67023)** —
  `fix(tui)`: surface image-routing downgrades instead of silently sending a
  text description (#66829). An `auto`-mode session that fell back to text —
  including a user who had explicitly forced `native` on a Codex app-server
  transport — got no signal at all; the only evidence was a gateway stderr
  line. Adds a decision helper returning `(mode, downgrade_reason)`, a
  `status.update` on every downgrade path, and a per-turn routing trace.
  Rebase note: while this sat, upstream added a `requested_provider=` argument
  to the exact `decide_image_input_mode()` call the PR replaces. Resolving the
  conflict by dropping the superseded block — the mechanically clean
  resolution — would have silently reverted that fix with no test failing.
  The argument was ported into the helper and verified present in the final
  diff.
- **[#65452](https://github.com/NousResearch/hermes-agent/pull/65452)** —
  `fix(toolsets)`: string-typed `disabled_toolsets` config (YAML-quoted or
  bare scalar) was iterated per *character*, silently disabling nothing —
  the full tool schema set went out on every model call (#61264, and the
  schema-blowup contributor to #61265's multi-minute local-model stalls).
  One shared coercion helper at the `get_tool_definitions()` funnel (which
  also fixes memo-cache-key pollution) and the cron denylist merge, with a
  loud misconfiguration warning and 12 regression tests.
- **[#64525](https://github.com/NousResearch/hermes-agent/pull/64525)** —
  `fix(toolsets)`: `disabled_toolsets: [browser]` silently stripped
  `web_search` from every session (it was a member of both `browser` and
  `web`, and disabled toolsets are a strict subtraction). Removed the
  taxonomy overlap (#64503). When the maintainer's review bot spotted that
  independent distribution rolls dropped browser+web_search co-occurrence
  from 97% to 94.09%, restored the exact coupling same-day with grouped
  `"+"`-compound sampler entries plus deterministic co-occurrence tests.
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
  (#59890). Rated salvageability=high by the maintainer's review bot; its
  one request (poller-level wiring tests) was added the same day.

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

## Community reviews (49 threads on other contributors' work)

Non-maintainer reviews and triage across 49 threads authored by other people
(38 PRs + 9 issues; count from GraphQL `commenter:falkoro`, which unlike
`gh search issues --commenter` does not under-report PR comments). Method:
git-protocol diffs (never the REST budget), duplicate sweeps before endorsing,
and an empirical check of every load-bearing claim. Representative:

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
- **[#69895](https://github.com/NousResearch/hermes-agent/pull/69895)** —
  A/B'd the branch against `main` with the models.dev catalog stubbed for
  determinism, confirming `decide_image_input_mode` moves `"text"` → `"native"`
  for a Nous vendor-qualified id while a text-only vendor still resolves
  `False`. Adversarially probed the containment the author claimed (a non-Nous
  provider whose model name merely contains a slash must not be reinterpreted
  as a provider route) and confirmed it holds. Flagged one non-blocking
  diagnostic defect: the new block calls a name bound by an import inside an
  earlier `try:`, so an `ImportError` there produces a misleading "cannot
  access local variable" log rather than the real cause — demonstrated with a
  simulated `ImportError`.
- **[#70091](https://github.com/NousResearch/hermes-agent/pull/70091)** —
  a 744-line kanban-notifier ownership rewrite touching the opposite end of an
  open PR of mine. Rather than claim a full logic review, built the actual
  combined tree and ran both sides on it: 469 tests pass across the kanban
  surface, no merge-order constraint. The comment states explicitly that this
  is a composition check, not an endorsement of the ownership logic not
  verified.
- Multiple duplicate-arbitration comments (e.g. #62844 vs the canonical #43645)
  comparing PR heads programmatically and crediting the earlier author.

A community member independently cherry-picked #60085 onto current `main` as
**[#66435](https://github.com/NousResearch/hermes-agent/pull/66435)**,
preserving the original `falkoro` authorship on both commits. Verified the
salvage was faithful (byte-identical bar one blank line) and thanked them
rather than rebasing #60085 to compete with it — a duplicate pair of mergeable
PRs just pushes arbitration work onto the maintainer. #60085 is retained only
as a fallback.

Two authors returned to request re-review after addressing feedback
([#57849](https://github.com/NousResearch/hermes-agent/pull/57849),
[#60897](https://github.com/NousResearch/hermes-agent/pull/60897)); the
maintainer's automated sweeper independently reached the same close/keep verdicts
as several of these reviews.
