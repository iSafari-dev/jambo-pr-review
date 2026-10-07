# IMPROVEMENTS — Jambo Devs PR review

Practical roadmap for a small Kenyan / African dev team turning an MIT PR-Agent snapshot into a branded product. Prefer **config + prompts + ops** before deep refactors.

## Principles

- Ship a **working reviewer on real PRs** before rewriting architecture.
- Keep Python package name `pr_agent` until branding is stable (avoids breaking imports, Docker targets, tests).
- One **self-hosted** service for many repos beats per-repo Actions sprawl.
- You still pay for **inference** (API or local GPU) — software is free; tokens are not.
- Measure: useful comments / noise, time-to-first-comment, cost per PR.

---

## Phase 0 — Baseline (week 1)

- [ ] Deploy GitHub App (or GitLab webhook) per [DEPLOY.md](DEPLOY.md) on the existing server.
- [ ] Start small: `GUNICORN_WORKERS=2` if RAM is tight (~1Gi guidance for 4 workers upstream).
- [ ] Install the app on 1–2 active repos only; run `/review` and `/improve` on real PRs.
- [ ] Log: model used, approx token cost, which comments the team kept vs ignored.
- [ ] Document secrets location (env / mounted volume) — never commit `.secrets.toml`.

## Phase 1 — Team rules without code changes (weeks 1–2)

- [ ] Add org-wide `.pr_agent.toml` (upstream: `config.global_settings_repo` + settings repo, or mount/extra config URL).
- [ ] Tune review focus: tests, security, style noise off if it wastes review time.
- [ ] Add `pr_compliance_checklist.yaml` (or project checklist) for must-check items.
- [ ] Point `repo_context_files` at `AGENTS.md` / team coding standards so the model sees local norms.
- [ ] Pick a default model for cost (e.g. a fast/cheap LiteLLM route) + a stronger fallback.

## Phase 2 — Brand & product surface (weeks 2–4)

- [ ] Bot display name / help copy: “Jambo Devs PR review” in docs and any user-facing strings you safely edit.
- [ ] Custom welcome / help comment text for first-time PR authors.
- [ ] Public README + DEPLOY already branded; keep CHANGELOG of *our* deltas vs upstream.
- [ ] Optional: logo, status page, internal Notion/Drive “how we use it” one-pager for the team.

## Phase 3 — Prompt productization (month 1–2)

- [ ] Fork prompt TOMLs under `pr_agent/settings/` (reviewer, improve, describe) — version them in git.
- [ ] Add Kenya / team-specific rules: naming, auth patterns, mobile money / M-Pesa pitfalls if relevant, Swahili-friendly PR summaries if the team wants `response_language`.
- [ ] A/B: run old vs new prompts on the same historical PRs; keep the quieter, more actionable set.
- [ ] Document “how to change a prompt” so any teammate can improve without reading all of LiteLLM.

## Phase 4 — Ops & multi-repo scale (month 2+)

- [ ] Install app across org; optional global settings repo for one policy everywhere.
- [ ] Pin Docker image to a **version tag or digest** (not floating `latest`) for production.
- [ ] Alerts: webhook 403 (missing secret), OOM, LLM timeouts; basic uptime check on `/` or health.
- [ ] Cost cap: monthly API budget; consider Ollama / local models for non-critical repos.
- [ ] Sync plan: merge upstream releases carefully ([UPSTREAM.md](UPSTREAM.md)).

## Phase 5 — Deeper fork (only when needed)

- [ ] New tools (custom `/jambo_*` commands) via upstream extension guide.
- [ ] Extra git provider if you outgrow GitHub/GitLab.
- [ ] Hard rename `pr_agent` → `jambo_pr_review` (big bang: imports, Docker CMD, tests, docs) — schedule deliberately.
- [ ] SaaS wrapper (auth, billing, multi-tenant) only after the self-hosted product is loved internally.

---

## What “improving the fork” usually means (cheat sheet)

| Goal | Where to change |
|------|-----------------|
| Review tone / categories | `pr_agent/settings/pr_reviewer_prompts.toml`, `[pr_reviewer]` in config |
| Code suggestions | improve prompts + `[pr_code_suggestions]` |
| Cheaper / different model | `[config] model`, `fallback_models`, provider API keys |
| Ignore noise paths | `ignore.toml`, `generated_code_ignore.toml` |
| Org defaults | `.pr_agent.toml` / global settings repo |
| New slash command | `pr_agent/tools/`, register in `pr_agent/agent/pr_agent.py` |
| Deploy reliability | gunicorn workers, webhook secret, pinned image tag |

Upstream docs: https://docs.pr-agent.ai/usage-guide/configuration_options/  
Extension: `docs/docs/usage-guide/extending_pr_agent.md`

---

## Non-goals (for now)

- Rewriting the agent framework from scratch.
- Per-repo Actions as the primary distribution model.
- Claiming parity with closed-source commercial reviewers (Qodo, etc.) — we own the fork and the prompts; we iterate on what our teams need.
