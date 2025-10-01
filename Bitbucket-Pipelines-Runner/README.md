# Bitbucket Pipelines Self‑Hosted Runner on Unraid (CA Template)

Run Bitbucket Pipelines jobs on your Unraid box using Atlassian’s **Linux Docker runner**. This guide covers installing via a Community Applications (CA) template, connecting it to Bitbucket Cloud, and targeting it from your pipelines.

> **At a glance**
> - Image: `docker-public.packages.atlassian.com/sox/atlassian/bitbucket-pipelines-runner:latest`
> - Ports: **None required** (outbound-only)
> - Volumes: runner work dir + the Docker socket
> - Security: mounting `/var/run/docker.sock` grants root-equivalent access to the host Docker daemon. Restrict who can run pipelines that target this runner.

---

## 1) Prerequisites

- Unraid 6.12+ with Docker enabled and the **Community Applications** plugin installed.
- Bitbucket Cloud repository or workspace where you have admin permissions.
- Internet egress (HTTPS) from Unraid to Bitbucket.
- Disk space: ensure you have headroom for build images/layers (consider a regular `docker system prune`).

---

## 2) Add a Runner in Bitbucket Cloud (collect credentials)

1. In Bitbucket, navigate to either:
   - **Workspace settings → Runners → Add runner**, or
   - **Repository settings → Runners → Add runner** (repo-scoped).
2. Choose **Linux Docker**.
3. Bitbucket will display a `docker run …` command that contains the values you need (UUIDs, OAuth ID/secret, labels).
4. Copy the following fields from that page; you’ll paste them into Unraid:
   - `ACCOUNT_UUID` (workspace)
   - `REPOSITORY_UUID` (only if repo-scoped; otherwise leave blank)
   - `RUNNER_UUID`
   - `RUNNER_NAME` (you may customise)
   - `RUNNER_LABELS` (e.g. `unraid,linux.docker`)
   - `OAUTH_CLIENT_ID`
   - `OAUTH_CLIENT_SECRET`

> **Tip:** Keep your labels specific (e.g., `unraid,build,k8s`) and always include `self.hosted` in pipelines (Bitbucket requires it).

---

## 3) Install via the Unraid CA Template

You have two ways to use the XML template:

### Option A — Quick local “User template” (no GitHub repo)
1. On Unraid, go to **Docker → Add Container**.
2. Toggle **Advanced View** and fill fields from the XML provided in this repo (or paste the XML into a new file at:  
   `/boot/config/plugins/dockerMan/templates-user/bitbucket-pipelines-runner.xml`).
3. Back on **Docker → Add Container**, select the template from the **User templates** dropdown.
4. Populate the Environment variables with the values collected in Step 2 and click **Apply**.

### Option B — Private CA Repository (shows under Apps → Private)
1. Create a public GitHub repo for your templates (e.g., `unraid-templates`). Place the XML at the repo root or a folder you prefer.
2. In Unraid, go to **Apps → Settings → Template repositories**, and add your repo URL.
3. Return to **Apps**; your template will appear under **Private Apps**. Click **Install** and fill the variables.

---

## 4) Template fields (what they mean)

- **ACCOUNT_UUID** (required): Your workspace UUID including braces `{…}`.
- **REPOSITORY_UUID** (optional): Only for repo-scoped runners; leave blank for workspace-scoped.
- **RUNNER_UUID** (required): Unique runner identifier Bitbucket generated.
- **RUNNER_NAME** (recommended): Friendly name shown in Bitbucket (e.g., `unraid-runner-1`).
- **RUNNER_LABELS** (recommended): Comma-separated list; you’ll match these in `bitbucket-pipelines.yml`.
- **OAUTH_CLIENT_ID/SECRET** (required): Paste from Bitbucket’s Add Runner page; treat as secrets.
- **/runner** volume (required): Stores runner state and logs.
- **/var/run/docker.sock** (required): Lets the runner launch step containers on the host (**security sensitive**).
- **/var/lib/docker/containers** (optional, read-only): Allows the runner to read step container logs.

No WebUI is exposed. Control via the Unraid Docker page and logs (`docker logs -f <container>`).

---

## 5) Verify the runner comes online

After the container starts:
- In Bitbucket **Runners** page, status should change to **ONLINE** within ~30–60 seconds.
- View Unraid logs for the container to see connection/heartbeat messages.

If it stays offline:
- Ensure time is correct on Unraid (NTP), DNS resolves, and outbound HTTPS isn’t blocked.
- Re-check `ACCOUNT_UUID`, `RUNNER_UUID`, OAuth values and label spelling.

---

## 6) Target the runner from a pipeline

Add a labelled step in your `bitbucket-pipelines.yml`:

```yaml
pipelines:
  default:
    - step:
        name: Build on Unraid
        runs-on:
          - self.hosted
          - unraid            # must match one of RUNNER_LABELS
        services:
          - docker            # enables Docker for Linux-Docker runners
        script:
          - docker info
          - echo "Hello from Unraid runner!"
```

> **Docker builds on the runner**: Because the runner mounts the host Docker socket, you can build/push images in your step. Just ensure the `services: docker` line is present for self-hosted Linux Docker runners.

---

## 7) Updating

- Update from Unraid by clicking **Force update** on the container (tag `:latest`).
- The runner reconnects automatically with the same UUID and labels.

---

## 8) Housekeeping

- Image bloat is common on CI hosts. Periodically prune unused layers and containers:
  ```bash
  docker system prune -af
  docker volume prune -f
  ```
- Consider a weekly cron on Unraid if disk space is tight.

---

## 9) Security notes

- **Docker socket = root**: Anyone who can trigger pipelines that target your runner can start containers on your host. Limit runner use to trusted repos and teams; use specific labels and avoid `runs-on: [self.hosted]` without additional labels.
- Store OAuth credentials as masked variables in the Unraid template and limit access to the Unraid UI.
- For decommissioning, **remove the runner in Bitbucket** first, then stop/remove the container.

---

## 10) Uninstall / Rotate tokens

1. In Bitbucket, go to **Runners**, select your runner, and **Remove** it.
2. Stop & remove the container from Unraid.
3. Remove the appdata folder (`/mnt/user/appdata/bitbucket-runner`) if you want a clean re‑install.
4. Re-add a new runner in Bitbucket to obtain fresh OAuth credentials.

---

## 11) Sample Troubleshooting

- **`offline` in Bitbucket**: Incorrect UUIDs or OAuth creds; copy them exactly (including braces).
- **`permission denied` on Docker**: Ensure `/var/run/docker.sock` is mounted RW.
- **Builds are slow**: Add a cron to prune images; ensure cache layers are retained (don’t always use `--no-cache`).
- **Cannot pull image**: Unraid Host DNS; try `nslookup docker-public.packages.atlassian.com` from the Unraid shell.

---

## 12) Example directory layout

```
/mnt/user/appdata/bitbucket-runner/
├─ logs/
└─ data/        # internal runner state
```

---

## 13) License & Credits

- Runner image and functionality © Atlassian.
- This Unraid template is provided as-is.
