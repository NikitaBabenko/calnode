# Calnode on VibeNest

[![Deploy to VibeNest](https://vibenest.net/deploy-to-vibenest.svg)](https://vibenest.net/deploy?repo=https://github.com/NikitaBabenko/calnode)

This fork preserves the Apache-2.0 license and builds the upstream application
from source with the existing root Dockerfile. Baseline upstream commit:
`9502e61ecf1b6c89014adc069d71743c50cdf68f` (checked 2026-10-11).
The badge opens VibeNest's repository preview and setup; it does not provision
secrets, reserve hardware or prove a successful deployment.

## Configure before the first launch

1. Create a dedicated project and select this repository, branch `main`, root
   directory `/`, Dockerfile build pack and internal port **3000**. Attach an
   available Free server if testing Free. Keep other projects' servers attached.
2. In **Project Settings → Storage → Persistent Volume**, mount **`/data`**.
   Verify the managed mount is attached before the application starts. The
   Dockerfile alone does not provision a persistent volume.
3. In **Environment**, configure:

   | Name | Value |
   |---|---|
   | `PORT` | `3000` |
   | `DATABASE_URL` | `sqlite:///data/calnode.db` |
   | `BASE_URL` | The project's actual public `https://…vibenest.net` URL |
   | `CALNODE_ENCRYPTION_KEY` | A securely generated, permanent random secret |

   Generate at least 32 random bytes (for example `openssl rand -hex 32`) and
   save the key in a private password manager. Keep the same value across
   restarts and redeploys. Never put it in Git, a README, a badge URL or logs.
   HTTPS enforces this key and enables secure cookies. `PUBLIC_BASE_URL` can
   remain unset when it matches `BASE_URL`. Do not disable secure cookies.
4. Leave LiveKit, recordings, payment providers, Google/Microsoft OAuth,
   SMTP and AI integrations unconfigured for the basic scheduling test.
   Without mail configuration, booking confirmation email delivery is not tested.
5. Deploy through VibeNest. Open the public HTTPS URL; `/` redirects to `/admin/`.
   Complete first-run owner setup immediately, then configure availability and
   create an event type with a simple non-video meeting location.

VibeNest's proxy must preserve the public Host for Calnode's same-origin CSRF
checks. Test admin writes and sign-out/sign-in over HTTPS, not just `/healthz`.

## Free-tier acceptance

The requested test budget is **256 MB RAM, 0.5 vCPU and 4 GB SSD**. Upstream's
`fly.toml` asks for 1 GB; it does not prove a 256 MB deployment works. The
current VibeNest UI labels existing Free inventory as Permanent; verify the
actual entitlement duration for the new test project instead of assuming a
three-month trial.

Record the exact fork SHA, provider attempt, serving image digest/revision,
actual cgroup memory/CPU limits, startup peak and idle RAM, CPU/disk usage,
restart count and OOM events. Create a public booking page and one booking;
verify it in admin and reject a duplicate occupied slot. Exercise a small
bounded request burst and confirm the service remains healthy. Then use
VibeNest's normal restart and redeploy controls, verify data and encrypted
settings survive, and confirm session/cookie behavior.

Before each launch/restart/redeploy, check the exact project's `IsBlocked` and
`SafetyHoldId` through the production Security Operator API. Read any hold
dossier; an active restriction or unknown state stops the test. The fresh
read does not replace VibeNest's server-side dispatch gate.

Free acceptance remains **unverified** until these checks pass on an actual
Free server. A successful higher-tier deployment is a separate result.

## Images and build resources

On 2026-10-11, GHCR `edge` resolved to multi-architecture digest
`sha256:ee64ec94641c89e1c9f265bbef5824290557c8c8936db0e69f916ec1f2013284`,
whose amd64 image label reports upstream revision
`91e9932c957de0bb9cbd9993dda2ff99a0be4f42`. It is older than this fork's source.
No `sha-9502e61` image was available. Do not substitute the moving `edge` or
`latest` tag for a test of this fork's exact source revision.

The Dockerfile builds the Svelte admin and Go backend; runtime limits do not
bound build-resource consumption. Check the selected builder's RAM, CPU and
disk before compiling. Do not fall back automatically to a weak laptop or
perform an unbounded build alongside production load. Fork Actions are disabled
pending a confirmed build budget; the inherited publisher names the upstream
GHCR repository and must not be enabled unchanged.

For backup, optional Litestream configuration is covered by [DEPLOY.md](DEPLOY.md).
Volume persistence is not an off-host backup.
