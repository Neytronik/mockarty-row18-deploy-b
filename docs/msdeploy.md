# Deployment Evidence Marker: svc-b

k8s-deploy-variant-B-ok

This marker is consumed by Mockarty's kubernetes deployer for svc-b deployment to
k8s-live-b in environment stage. Deployment and post-deploy health evidence are
produced by the deploy node, not by this repository.

## Delivered artifact

- Repository: https://github.com/Neytronik/mockarty-row18-deploy-b.git
- Delivery target: k8s-live-b
- Environment: stage
- Artifact revision: the exact accepted commit SHA recorded for this repository
- Deployment actor: mockarty-k8s-deployer (deploy node only, no manual deployment)
- Deployment descriptors: deployed unchanged from the accepted commit

## Deploy-node receipt and health-window contract

The deploy node deploys the exact accepted commit SHA to k8s-live-b/stage and
records one append-only receipt. Reviewers accept the deployment only when the
receipt carries every field below.

| Field | Required value |
|-------|----------------|
| `repository` | `https://github.com/Neytronik/mockarty-row18-deploy-b.git` |
| `commitSha` | the exact accepted commit SHA delivered by this run |
| `target` | `k8s-live-b` |
| `environment` | `stage` |
| `healthProbe` | `GET http://127.0.0.1:18798/` |
| `healthResult` | `200` |
| `healthWindow` | start and end timestamps of a contiguous green window after deployment |
| `result` | `green` |
| `actor` | `mockarty-k8s-deployer` |
| `manualDeploy` | `false` |

### Acceptance flow

1. Take the accepted svc-b commit SHA as the only artifact revision.
2. The deploy node deploys that SHA to k8s-live-b/stage.
3. The deploy node probes `GET http://127.0.0.1:18798/` and requires HTTP 200
   for the whole post-deploy health window.
4. The deploy node stores the receipt binding repository, exact SHA, target,
   environment and the green health window.
5. Review-readiness combines this receipt with the separate service-a run
   receipt. This run alone verifies svc-b only.

### Fail-closed review conditions

- `commitSha` differs from the accepted delivered commit.
- `target` is not `k8s-live-b` or `environment` is not `stage`.
- The health probe returns non-200, times out, or the window is not contiguous.
- The receipt is missing the actor or any health field.
- The actor is anything but `mockarty-k8s-deployer`, or any manual deployment
  step is attributed.
