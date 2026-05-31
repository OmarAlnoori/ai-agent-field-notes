# Coolify (self-hosted PaaS)

**Verdict:** ✅ Worked — reliable self-hosted PaaS for n8n and other Docker apps

---

## What It Is

[Coolify](https://coolify.io) is an open-source, self-hosted alternative to
Heroku/Render. You run it on your own VPS, and it handles deploying Docker-based
apps, managing environment variables, SSL certificates, and auto-deploys on git
push. We use it to host n8n and other automation infrastructure.

---

## Why We Use It

- One VPS hosts n8n, the app backend, and other services — no managed hosting cost.
- Git-push deploys: push to `origin/master`, Coolify redeploys automatically.
- Per-service environment variable management in the UI.
- Free and open-source.

---

## Safely Updating Coolify

We upgraded from `4.0.0-beta.463` → `4.1.1` (beta to first stable release).
This is what we learned.

### Before You Update

**1. Check the changelog first**

Read the [GitHub releases page](https://github.com/coollabsio/coolify/releases)
for breaking changes and migration notes. Beta → stable jumps often include
database schema migrations that take longer than patch updates.

**2. Snapshot your server**

Before touching anything, take a full snapshot/backup of your VPS from your
cloud provider (DigitalOcean, Hetzner, etc.). This is your rollback — if
something breaks badly, you restore the snapshot and lose nothing. Do not skip
this step.

**3. Wait for active deployments to finish**

Check the Coolify dashboard — don't update mid-deploy.

### Performing the Update

**Option A — Via the Coolify UI (recommended)**

1. Log in → Settings (bottom-left gear icon)
2. Look for the "Update Available" banner or Updates section
3. Click Update and confirm
4. Coolify pulls the new image and restarts itself (~2–5 minutes for a normal update)

**Option B — Via SSH**

```bash
curl -fsSL https://cdn.coollabs.io/coolify/install.sh | bash
```

The install script detects an existing installation and upgrades in place.

### What to Expect During the Upgrade

| What's happening | Estimated time |
|---|---|
| Pulling new Docker image | 1–3 min (depends on bandwidth) |
| Stopping old container | ~10 sec |
| Running DB migrations | 30 sec – 2 min (longer for big version gaps) |
| Container restart | ~30 sec |
| **Total (normal update)** | **2–5 minutes** |
| **Total (beta → stable)** | **3–10 minutes** |

The Coolify dashboard will be unreachable during the restart — this is normal.
Your deployed apps (n8n, etc.) keep running; only Coolify itself restarts.

### If It's Taking More Than 10 Minutes

This happened to us. Don't panic — SSH in and check before doing anything
drastic:

```bash
# Check container state
docker ps -a | grep coolify

# Check what it's doing
docker logs coolify --tail 100
```

**What each state means:**

| `docker ps -a` status | What it means | Action |
|---|---|---|
| `Up` | Running, dashboard should come back | Wait 2 more min, refresh |
| `Restarting` | Crash-looping | Read logs — likely a migration error |
| `Exited` | Stopped | Read logs for exit reason |

**If logs show DB migration output scrolling** — it's still working. A beta →
stable jump can run dozens of migrations. Give it a few more minutes.

**If logs show `failed`, `exception`, or `exit code 1`** — it's stuck. Common
causes: disk space, DB connection failure, or a migration conflict. Share the
log output with the [Coolify Discord](https://discord.gg/coolify) or
[GitHub Issues](https://github.com/coollabsio/coolify/issues).

**If you need to rollback** — restore the server snapshot from before the update.

### After the Update

1. Reload the dashboard → verify the version number in Settings.
2. Browse your deployed apps — confirm they're still running (they should be;
   Coolify restart doesn't touch running containers).
3. Check the Deployments tab for any unexpected restarts.
4. Optionally trigger a small redeployment to confirm the pipeline still works.

---

## n8n-specific Config in Coolify

When deploying n8n on Coolify, set these environment variables in the Coolify UI
under the service's Environment Variables tab:

```
N8N_HOST=your-n8n-domain.com
WEBHOOK_URL=https://your-n8n-domain.com/
N8N_BINARY_DATA_MODE=filesystem        # or database (see tools/n8n-ce.md)
N8N_BINARY_DATA_STORAGE_PATH=/data/binary
EXECUTIONS_DATA_PRUNE=true
EXECUTIONS_DATA_MAX_AGE=168             # hours — 7 days
```

Set a persistent volume for `/home/node/.n8n` and (if using filesystem binary
mode) `/data/binary` so data survives container restarts.

---

## See Also

- [Tool: n8n Community Edition](n8n-ce.md)
- [Coolify documentation](https://coolify.io/docs)
- [Coolify GitHub](https://github.com/coollabsio/coolify)
