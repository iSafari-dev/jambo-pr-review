# Upstream

**Jambo Devs PR review** is a productized fork / snapshot of
[The-PR-Agent/pr-agent](https://github.com/The-PR-Agent/pr-agent) (MIT).

| Field | Value |
|-------|--------|
| Upstream repo | https://github.com/The-PR-Agent/pr-agent |
| Snapshot | `main` zipball @ `821c6ad173194d443a3f768b5a1087924025e90f` |
| Upstream package name | `pr-agent` (Python imports stay `pr_agent` — do not rename casually) |
| Upstream docs | https://docs.pr-agent.ai |
| Docker Hub (upstream images) | https://hub.docker.com/r/pragent/pr-agent |

## Syncing from upstream later

1. Fetch upstream main / a release tag.
2. Merge or cherry-pick carefully — preserve our README, IMPROVEMENTS.md, DEPLOY.md, NOTICE.
3. Keep Python package paths (`pr_agent/`) unless you plan a large rename + migration.
4. Re-test Docker `--target github_app` (or GitLab webhook) before promoting to the shared server.

Prefer **configuration + docs branding** over renaming the Python package until the product is stable.
