# Deploy — Jambo Devs PR review

This file documents **real upstream deployment paths** from [The-PR-Agent/pr-agent](https://github.com/The-PR-Agent/pr-agent).  
We do **not** ship a fake root `docker-compose.yml`: the only compose in this tree is `docker/mosaico/docker-compose.pr-agent.yml`, which `extends` external MOSAICO `base-definitions.yml` and is **not** a standalone PR-review stack.

Canonical upstream docs: https://docs.pr-agent.ai/installation/

Preferred for Jambo Devs: **one self-hosted GitHub App (Docker `github_app` target)** covering many repos.

---

## 1. Preferred — GitHub App (self-hosted webhook)

Source of truth: `docs/docs/installation/github.md` → “Run as a GitHub App”.

### Build

From this repo root:

```bash
docker build . -t jambo-pr-review:github_app --target github_app -f docker/Dockerfile
```

Optional: push to your registry:

```bash
docker tag jambo-pr-review:github_app <your-registry>/jambo-pr-review:github_app
docker push <your-registry>/jambo-pr-review:github_app
```

Upstream also publishes images under [`pragent/pr-agent`](https://hub.docker.com/r/pragent/pr-agent) (e.g. versioned `*-github_app` tags). Pin a **version or digest** in production, not a floating rolling tag.

### Secrets (required)

```bash
cp pr_agent/settings/.secrets_template.toml pr_agent/settings/.secrets.toml
# Edit: openai.key, github app_id, private_key, webhook_secret
```

- **Webhook secret is mandatory.** If `GITHUB.WEBHOOK_SECRET` / `GITHUB__WEBHOOK_SECRET` is missing, the server rejects webhooks with **HTTP 403**.
- Generate one: `python -c "import secrets; print(secrets.token_hex(10))"`
- **Never bake** `.secrets.toml` into the image (excluded from Docker context). Inject at runtime via env vars or a mounted secret volume.
- Service images run as UID/GID **10001** — mounts must be readable/writable by that user.
- Example env-style names: `OPENAI.KEY` / `OPENAI__KEY`, GitHub App fields as documented in the secrets template.

Also set deployment type for App mode in configuration (`deployment_type` / GitHub section — see upstream github.md and `configuration.toml`).

### Run (pattern)

Upstream hosts the app on a server/container; default gunicorn **PORT=3000**. Example pattern (adjust mounts/env to your host):

```bash
docker run -d --name jambo-pr-review \
  -p 3000:3000 \
  -e PORT=3000 \
  -e GUNICORN_WORKERS=2 \
  -e OPENAI__KEY="<key>" \
  # plus GitHub App env / or mount secrets to /app/pr_agent/settings_prod \
  jambo-pr-review:github_app
```

Exact secret mount paths used by upstream K8s examples: `/app/pr_agent/settings_prod` (see github.md). Prefer copying their volumeMount pattern rather than inventing paths.

### GitHub App checklist

1. Create GitHub App with required permissions & events (PR, issue comment, etc. — full list in upstream github.md).
2. Webhook URL → your public HTTPS endpoint (or smee.io while developing).
3. Webhook secret → same value as in secrets.
4. Install the app on target repositories (and on a settings repo if using org-wide `.pr_agent.toml`).

### Sizing (small VPS)

From `docs/docs/installation/index.md`:

| Variable | Default | Notes |
|----------|---------|--------|
| `GUNICORN_WORKERS` | unset (auto) | Pin e.g. `2` if OOM |
| `GUNICORN_MAX_WORKERS` | `4` | Cap on auto workers |
| `PORT` | `3000` | Bind port |

Rough memory: ~1Gi for default 4 workers; lower workers if the container is OOMKilled at startup.

---

## 2. Alternative — GitLab / Gitea / Bitbucket webhooks

Same Dockerfile, different `--target`:

| Target | Role |
|--------|------|
| `gitlab_webhook` | GitLab |
| `gitea_app` | Gitea |
| `bitbucket_server_webhook` | Bitbucket Server |
| `bitbucket_app` | Bitbucket Cloud app entry |
| `github_polling` | Polling instead of webhook |
| `github_action` | Action/entrypoint image |

See `docker/Dockerfile` and provider pages under `docs/docs/installation/`.

---

## 3. CLI / one-off review (no long-running server)

Useful to validate API keys before standing up the App:

```bash
docker run --rm -it \
  -e OPENAI__KEY=<your_openai_key> \
  -e GITHUB__USER_TOKEN=<your_github_token> \
  pragent/pr-agent:latest \
  --pr_url <pr_url> review
```

Env-file form and other providers: `docs/docs/installation/locally.md`.

---

## 4. Per-repo GitHub Action (not preferred for “all my repos”)

Upstream still supports Actions (`uses: the-pr-agent/pr-agent@main` + `OPENAI_KEY` / `GITHUB_TOKEN`).  
For Jambo Devs product goals, treat this as a **fallback**, not the primary distribution — see README.

---

## 5. MOSAICO compose (do not use as default)

`docker/mosaico/docker-compose.pr-agent.yml` is an overlay for the MOSAICO demonstrator (`extends: base-definitions.yml`). It will not run alone. Ignore unless you are integrating MOSAICO intentionally (`docs/docs/installation/mosaico_server.md`).

---

## 6. Local build checklist

```bash
cd /workspace/jambo-pr-review   # or your clone path
docker build . -t jambo-pr-review:github_app --target github_app -f docker/Dockerfile
```

Requires Docker, network for `uv sync` during build, and enough disk/RAM for the image build.

---

## References

- Upstream GitHub App: https://docs.pr-agent.ai/installation/github/
- Upstream installation index (workers, Docker Hub tags): https://docs.pr-agent.ai/installation/
- Config reference: `pr_agent/settings/configuration.toml`
- Secrets template: `pr_agent/settings/.secrets_template.toml`
- Product roadmap: [IMPROVEMENTS.md](IMPROVEMENTS.md)
