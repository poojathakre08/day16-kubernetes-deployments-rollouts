# Day 16/90 — Kubernetes Deployments & Rollouts

## Overview

This lab explores safe application releases using Kubernetes Deployments. I performed a rolling update, inspected Deployment revision history, simulated a failed image update, diagnosed the Pod Events, and rolled back to a working revision.

## Objectives

- Perform a controlled rolling update.
- Record a release change-cause annotation.
- Inspect Deployment and ReplicaSet revision history.
- Diagnose an image-pull failure.
- Roll back and verify application recovery.
- Understand `maxSurge` and `maxUnavailable`.

## Environment

- Kubernetes: Local learning cluster
- Workload: Existing Nginx Deployment
- Deployment name: `nginx-deployment`
- Known-good image: **[Fill in actual image]**
- Updated image: **[Fill in actual image]**
- Failed test image: `nginx:1.99-broken`

## 1. Controlled Release

I recorded the release reason using the `kubernetes.io/change-cause` annotation and updated the container image.

Commands:

```bash
kubectl annotate deployment/nginx-deployment kubernetes.io/change-cause="Day 16: controlled Nginx rolling update practice" --overwrite

kubectl set image deployment/nginx-deployment nginx=nginx:1.28

kubectl rollout status deployment/nginx-deployment
```

**Result:** [Record your actual rollout result.]

## 2. Revision History

```bash
kubectl rollout history deployment/nginx-deployment
kubectl get rs -o wide
```

Revision inspected: **[Actual revision number]**

Release image: **[Actual image]**

Release change-cause: **[Actual observed annotation or note]**

## 3. Failed Image Update

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.99-broken
kubectl get pods
kubectl get events --sort-by=.metadata.creationTimestamp
```

**Observed failure:** [Record the actual Pod status and Event message.]

**Root cause:** The deliberately invalid image tag could not be pulled, so the new container could not become Ready and the rollout could not complete.

## 4. Rollback and Verification

```bash
kubectl rollout undo deployment/nginx-deployment
kubectl rollout status deployment/nginx-deployment
kubectl get deployment nginx-deployment -o wide
kubectl get pods -o wide
```

**Restored image:** [Actual image]

**Verification:** [Record the actual rollout status and Pod readiness.]

## 5. Rolling Update Concepts

### maxSurge

The maximum number of extra Pods allowed above the desired replica count during a rolling update.

For 3 desired replicas and `maxSurge: 1`, Kubernetes may temporarily run up to 4 Pods.

### maxUnavailable

The maximum number of desired replicas that may be unavailable during the update.

For 3 replicas and `maxUnavailable: 0`, Kubernetes aims to keep all 3 desired replicas available during the rollout.

### Rollback

A rollback restores a previous saved Deployment revision. I verified recovery by checking the restored image, rollout status, and Pod readiness.

## 6. Evidence/Screenshots

- `screenshots/01-initial-cluster-state.png`
- `screenshots/02-rollout-success.png`
- `screenshots/03-rollout-history.png`
- `screenshots/04-failed-rollout-events.png`
- `screenshots/05-rollback-verified.png`

## Key Learnings

- Rolling updates replace Pods gradually.
- Deployment history helps track and restore releases.
- Pod Events are useful for diagnosing image-pull failures.
- Rollback restores an earlier Pod template but does not undo external database or other application-side changes.
- Release annotations improve traceability.

## Safety and Cleanup

This exercise was performed on a local learning cluster. I rolled back the invalid image and verified the restored Deployment. No Kubernetes credentials or secrets should be committed.
