# revision-health-repair

ParcelStatus builds but exits because its startup command is wrong. Its health probe also fails. Repair both faults and document a proposed recovery using the supplied hypothetical revision record.

## Prerequisites

Git, Docker Engine and POSIX shell (Linux/macOS or WSL with Docker integration). Compose v2 required for Compose commands; curl required for host HTTP checks. Docker must be running; internet access needed to download python:3.12-slim. Reserve documented host ports. Docker Desktop is subject to its license; Linux Docker Engine is sufficient. No cloud account, billing, registry login, real credentials, Kubernetes cluster or paid tools needed. Fork or clone this starter and run commands from repository root.

## Allowed edits

Dockerfile (CMD and health URL path only), recovery-review.md. Preserve all other source, routes, ports, fixtures and checks. No failure suppression or fabricated output.

## Required behaviour and commands

Build original image as parcelstatus:broken and run m4-q4 with APP_MODE=production, host port 18084 mapped to container port 8080. Capture actual exit status/code and startup log. Task 1 repairs only CMD to start the supplied application. Rebuild as parcelstatus:startup and recreate. Show stable running status and clean startup, leaving the health probe unchanged for Task 2.

Task 2 compares HTTP 200 at /health with Docker unhealthy status and the probe log. Repair only the health URL path to match the application endpoint; retain interval, timeout, start period, retries and probe enforcement. Build parcelstatus:fixed, recreate, wait for healthy and inspect probe history and FailingStreak, plus /health HTTP 200.

Task 3 compares running image ID against the built fixed image ID and completes five recovery-review.md lines. Use fixtures/revision-record.md to identify the hypothetical failed and known-good revisions. Propose revision listing, revision-scoped logs investigation, traffic rollback to the supplied known-good revision, and readiness/log/service-health verification. These are written proposals only; execute no cloud commands and claim no observed cloud recovery.
```sh
docker build -t parcelstatus:broken .
docker run -d --name m4-q4 -e APP_MODE=production -p 18084:8080 parcelstatus:broken
docker ps -a --filter name=m4-q4
docker logs m4-q4
docker inspect --format '{{.State.ExitCode}} {{json .State.Health}}' m4-q4
curl -i http://localhost:18084/health
```
Cleanup: docker rm -f m4-q4. Local Docker health is not a Cloud Run readiness result.
