# ParcelStatus startup and health recovery

ParcelStatus builds but exits because its startup command is wrong. After startup repair, its Docker health probe still fails. Repair both faults and document Cloud Run recovery as a written proposal using the supplied hypothetical revision record. Do not deploy to a cloud service.

## Get started

Fork or clone this repository and run commands from its root. A standalone working copy is equally valid; fork ownership is not graded.

Requirements: Git, a running Docker Engine, a POSIX shell and curl on Linux, macOS or WSL with Docker integration. Compose is not required. Linux Docker Engine is sufficient; Docker Desktop is optional and subject to its licence. No cloud account, billing, registry login, paid tool or host Python installation is needed. Internet access is needed for the initial public base-image download.

Check tools and download the base image before attempting repairs:

```sh
git --version
docker version
curl --version
docker pull python:3.12-slim
```

Host port `18084` and container name `m4-q4` must be available. Check existing containers with `docker ps -a --filter name=m4-q4`. If the name belongs to unrelated work, do not remove it. Resolve name/port conflicts as setup issues, not application faults. Preparation and download failures are not planted defects.

**Allowed edits:** `Dockerfile` (CMD and health URL path only), `recovery-review.md`. Preserve all other source, ports, responses, fixtures and probe enforcement. Do not suppress failures or fabricate output.

## Task 1: Repair startup command

Build and run the original image:

```sh
docker build -t parcelstatus:broken .
docker run -d --name m4-q4 -e APP_MODE=production -p 18084:8080 parcelstatus:broken
docker wait m4-q4
docker ps -a --filter name=m4-q4
docker logs m4-q4
docker inspect --format '{{.State.ExitCode}}' m4-q4
```

The original process should exit; `docker wait` prints its exit code. Diagnose the startup log, then correct CMD only to launch the supplied application. Leave the health-check fault unchanged.

Remove only the exercise container you just created, rebuild and recreate it:

```sh
docker rm m4-q4
docker build -t parcelstatus:startup .
docker run -d --name m4-q4 -e APP_MODE=production -p 18084:8080 parcelstatus:startup
sleep 10
docker ps -a --filter name=m4-q4
docker inspect --format 'status={{.State.Status}} running={{.State.Running}} started={{.State.StartedAt}}' m4-q4
docker logs m4-q4
```

Verify the process remains running and the log shows successful listening without the original startup error. Expected `/ready` probe 404 entries may remain until Task 2; these do not mean CMD repair failed.

**Evidence:** initial exit code/startup log, CMD correction, rebuilt/recreated running status and successful startup log.

## Task 2: Repair health probe

Compare the responding application endpoint with Docker's failing probe:

```sh
curl --fail -i http://127.0.0.1:18084/health
docker inspect --format '{{json .State.Health}}' m4-q4
```

The supplied probe runs every two seconds. Allow up to 30 seconds for the original unhealthy state to appear, then capture the probe error and application HTTP 200. Investigate the configured health URL against the application's routes.

Correct only the health URL path. Preserve interval, timeout, start period, retries and probe enforcement. Rebuild and recreate:

```sh
docker rm -f m4-q4
docker build -t parcelstatus:fixed .
docker run -d --name m4-q4 -e APP_MODE=production -p 18084:8080 parcelstatus:fixed
```

Wait up to 30 seconds for health convergence, then inspect actual evidence:

```sh
i=0
while [ "$i" -lt 30 ]; do
  state=$(docker inspect --format '{{.State.Health.Status}}' m4-q4) || break
  [ "$state" = healthy ] && break
  sleep 1
  i=$((i + 1))
done
docker ps -a --filter name=m4-q4
docker inspect --format '{{json .State.Health}}' m4-q4
curl --fail -i http://127.0.0.1:18084/health
```

A completed wait is not itself a pass. Verify `healthy`, successful probe log entries, `FailingStreak: 0` and HTTP 200. If these are absent, investigate rather than claiming success.

**Evidence:** application/probe disagreement, original probe error, corrected health path, rebuilt/recreated healthy state, probe history, zero failing streak and HTTP 200.

## Task 3: Document recovery evidence

Compare the running container's image ID with the built fixed image ID:

```sh
docker inspect --format '{{.Image}}' m4-q4
docker image inspect --format '{{.Id}}' parcelstatus:fixed
```

Complete the five existing lines in `recovery-review.md`:

1. Root cause and actual local proof for both faults.
2. Running image identity and comparison with the built fixed image.
3. Proposed Cloud Run revision and revision-scoped log investigation.
4. Proposed traffic rollback to the supplied known-good revision.
5. Proposed post-recovery verification and limits of the local evidence.

Use `fixtures/revision-record.md` for the hypothetical service, region and revision identities. These are written proposals only. Do not execute cloud commands or claim observed Cloud Run recovery. Docker health does not automatically configure or prove Cloud Run readiness.

Review the complete permitted-file diff:

```sh
git diff -- Dockerfile recovery-review.md
```

If using a non-Git working copy, provide equivalent before/after differences against the original starter. Keep a copy of that starter for comparison.

## Submission

Upload one PDF with three labelled task sections, at least one genuine terminal/CLI screenshot per task, the complete corrected permitted-file diff once, requested findings and concise cause/fix/result captions. Include before/after failure evidence where the task requests it. Repository URL, fork URL, branch name and submission commit hash are not required or graded.

After collecting evidence, remove only your exercise container:

```sh
docker rm -f m4-q4
```
