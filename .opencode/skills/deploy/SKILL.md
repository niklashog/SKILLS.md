---
name: deploy
description: Checklist-driven deployment workflow with pre-deploy verification and rollback plan
license: MIT
compatibility: opencode
metadata:
  tags: devops, deployment
---

## What I do

Guide a deployment from pre-flight checks through go-live and post-deploy verification, with a clear rollback plan documented before the first command runs.

## Steps

### Pre-deploy checklist

1. Confirm the correct ref: `git log -1` — is this the commit/tag intended for deploy?
2. Verify CI passed: `gh run list --branch <branch> --limit 5` — all checks green?
3. Check for pending migrations:
   - Do migrations need to run *before* the new code? (adding a column the app reads)
   - Or *after*? (dropping a column the old app still writes)
   - See `migrate` skill for zero-downtime column change order
4. Confirm env vars and secrets are in place in the target environment
5. Identify deploy strategy:
   - **Rolling update** (default for most platforms) — new instances replace old gradually; app must handle both schema versions during rollout
   - **Blue/green** — full parallel environment; instant cutover, instant rollback
   - **Canary** — route X% of traffic to new version; watch metrics before full rollout
6. Document the rollback plan *before* starting:
   - What tag/image to redeploy
   - Whether migrations need to be reversed (and if that's possible)
   - Estimated time to roll back

### Deploy

7. Identify the deploy mechanism:
   - CI/CD pipeline: trigger via `gh workflow run deploy.yml` or merge to main
   - Kubernetes: `kubectl set image deployment/<name> app=<image>:<tag>`
   - Fly.io: `fly deploy --image <registry>/<app>:<tag>`
   - Railway / Render: push to the deploy branch or trigger via dashboard
   - AWS ECS: `aws ecs update-service --cluster <c> --service <s> --force-new-deployment`
   - Serverless (Lambda): `aws lambda update-function-code --function-name <f> --zip-file fileb://dist.zip`
8. Note the exact command run and the timestamp
9. Monitor logs during rollout:
   - `kubectl rollout status deployment/<name>` (k8s)
   - `fly logs` (Fly.io)
   - CloudWatch / Datadog / Grafana — watch error rate and latency

### Post-deploy verification

10. Smoke test the critical path within 5 minutes of deploy:
    - Can a user log in?
    - Does the main feature work end-to-end?
    - Do key API endpoints return 200?
11. Check error rate in monitoring — is it the same as or lower than pre-deploy baseline?
12. Confirm any pre/post migrations ran successfully
13. If anything looks wrong: **initiate rollback immediately** — do not try to hotfix under live traffic

### Rollback

```bash
# Kubernetes — roll back to previous image
kubectl rollout undo deployment/<name>
kubectl rollout status deployment/<name>

# Fly.io — redeploy previous release
fly releases list
fly deploy --image <registry>/<app>:<previous-tag>

# AWS ECS — force previous task definition
aws ecs update-service --cluster <c> --service <s> --task-definition <name>:<previous-revision>

# Lambda — point alias to previous version
aws lambda update-alias --function-name <f> --name live --function-version <previous>
```

## Deployment strategy comparison

| Strategy | Rollback speed | Downtime | Schema complexity |
|---|---|---|---|
| Rolling update | ~2-5 min | Zero | App must handle both schema versions |
| Blue/green | Instant (DNS/LB swap) | Zero | New env can have new schema |
| Canary | Minutes (reduce traffic %) | Zero | Same as rolling |
| Recreate | ~2-5 min | Brief | Simplest — old fully down before new starts |

## Rules

- Never deploy directly to production without confirming with the user
- Always document the rollback plan before starting — not after
- If post-deploy smoke tests fail, roll back first and investigate second — never debug under live traffic
- Record what was deployed, when, and the git SHA in a deploy log or git tag
- For schema changes: follow the zero-downtime order in the `migrate` skill
