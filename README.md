# Jambo Devs PR review

**One self-hosted AI code-review service for many repos** — branded for [Jambo Devs](https://github.com/), built on the open-source [PR-Agent](https://github.com/The-PR-Agent/pr-agent) (MIT).

Repo slug: `jambo-pr-review`

> This is **not** a per-repo GitHub Action you copy into every project.  
> Preferred setup: **one Docker webhook / GitHub App server** on your machine that you install (or point webhooks at) across all org repos. That keeps Actions minutes and secret sprawl under control.

Upstream credit and license: see [NOTICE](NOTICE), [LICENSE](LICENSE), and [UPSTREAM.md](UPSTREAM.md).  
Python package imports remain `pr_agent` — we brand in docs and config first; we do **not** rename the package until the product is stable.

---

## Why this shape

| Approach | Fit for Jambo Devs |
|----------|--------------------|
| Per-repo GitHub Action | Unsustainable across many repos (workflow files + secrets everywhere) |
| **Self-hosted webhook / GitHub App (Docker)** | **Preferred** — one service, many repos, you own prompts & models |
| Upstream CLI (`pr-agent … review`) | Fine for local smoke tests |

Official upstream install paths (GitHub App, GitLab webhook, CLI, etc.):  
https://docs.pr-agent.ai/installation/

---

## Quick start (preferred: self-hosted GitHub App)

Full step-by-step with real upstream commands: **[DEPLOY.md](DEPLOY.md)**.

Short version:

1. Create a GitHub App (webhook secret required).
2. Copy secrets template and fill keys:
   ```bash
   cp pr_agent/settings/.secrets_template.toml pr_agent/settings/.secrets.toml
   ```
3. Build the upstream `github_app` image target:
   ```bash
   docker build . -t jambo-pr-review:github_app --target github_app -f docker/Dockerfile
   ```
4. Run the container with secrets at runtime (env or mounted volume — never bake `.secrets.toml` into the image). Default listen port is `3000` (`PORT`).
5. Point the GitHub App webhook at your server URL and install the app on the repos you want reviewed.

**Do not invent a random docker-compose** — upstream’s only compose under `docker/mosaico/` is for the MOSAICO agent overlay and needs external files. Use `docker build --target …` / published `pragent/pr-agent` tags as documented in DEPLOY.md.

---

## Environment & secrets (common)

Env var names often use `__` instead of `.` (especially Lambda / some hosts). Examples from upstream:

| Purpose | Typical vars |
|---------|----------------|
| OpenAI | `OPENAI__KEY` or `OPENAI_KEY` (Actions) |
| GitHub user / PAT (CLI & polling) | `GITHUB__USER_TOKEN` |
| GitHub App | App ID, private key, `GITHUB__WEBHOOK_SECRET` (required — missing secret → HTTP 403) |
| Model override | `CONFIG__MODEL`, `config.model` |
| Git provider | `CONFIG__GIT_PROVIDER` (`github`, `gitlab`, …) |
| Webhook workers / memory | `GUNICORN_WORKERS`, `GUNICORN_MAX_WORKERS`, `PORT` |

Secrets template: `pr_agent/settings/.secrets_template.toml`  
Default config knobs: `pr_agent/settings/configuration.toml`  
Per-repo / org overrides: `.pr_agent.toml` (see upstream usage guide).

CLI smoke test (upstream image, no self-host required):

```bash
docker run --rm -it \
  -e OPENAI__KEY=<your_openai_key> \
  -e GITHUB__USER_TOKEN=<your_github_token> \
  pragent/pr-agent:latest \
  --pr_url https://github.com/owner/repo/pull/123 review
```

Or with pip: `pip install "pr-agent[github]"` then `pr-agent --pr_url … review`.

---

## How to improve this fork (product path)

See **[IMPROVEMENTS.md](IMPROVEMENTS.md)** for the roadmap. In short:

1. **Custom prompts** — edit TOML under `pr_agent/settings/` (e.g. `pr_reviewer_prompts.toml`) and/or ship a shared org `.pr_agent.toml`.
2. **Team rules** — checklist YAML, ignore lists, review categories; optional `AGENTS.md` / repo context files.
3. **Models** — switch via `config.model` / LiteLLM (OpenAI, Anthropic, Gemini, Ollama, …) to control cost and latency.
4. **Brand** — bot name, help text, docs (this README); keep `pr_agent` imports until a deliberate rename.
5. **Measure** — useful comments vs noise, $/PR, then tighten prompts.

Upstream extension guide (models, providers, tools):  
`docs/docs/usage-guide/extending_pr_agent.md` and https://docs.pr-agent.ai/

---

## What’s in this tree

| Path | Role |
|------|------|
| `pr_agent/` | Upstream Python app (do not rename casually) |
| `docker/Dockerfile` | Multi-target image (`github_app`, `gitlab_webhook`, …) |
| `docs/` | Upstream Docusaurus docs |
| `DEPLOY.md` | Jambo Devs deploy notes (real upstream paths) |
| `IMPROVEMENTS.md` | Productization roadmap |
| `NOTICE` / `UPSTREAM.md` | Attribution & sync notes |

Upstream version in this snapshot: see `pyproject.toml` (`pr-agent` package) and [UPSTREAM.md](UPSTREAM.md).

---

## License

MIT — same as upstream PR-Agent. Include [LICENSE](LICENSE) and [NOTICE](NOTICE) in distributions.
