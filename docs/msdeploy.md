# Row #18 MSDeploy Record

Documentation-only record for Row #18. This page describes the delivery
handoff; it does not instruct anyone to deploy by hand.

## Row #18 — svc-b

| Field | Value |
| --- | --- |
| Service | `svc-b` |
| Delivery target | `k8s-prod-b` |
| Cluster | `mockarty-prod-verify` |
| Namespace | `mockarty-prod-verify` |
| Deployment authority | `deploy node` |
| Manifest mode | `pre-existing` |
| Health owner | `deploy node` |

The deploy node is the only actor that deploys `svc-b` to delivery target
`k8s-prod-b`. It uses the pre-existing manifest, pins the exact accepted
feature-branch SHA, and owns the post-deploy health verification. No manual
`kubectl`, `helm`, or other direct cluster mutation is used, and this run does
not perform an independent hand-run health check.

The deploy stand static surface at `http://127.0.0.1:8798/` is an acceptance
probe only. An HTTP 200 from that surface confirms the stand is reachable; it
does not prove deployment health for `svc-b`. Health status is reported only by
the deploy node.

## Deploy-node target policy, evidence and health ownership

This section records the contract the deploy node enforces for this delivery.
The target registry, the deploy-node control plane and the evidence store are
deploy-node-owned and outside this repository; nothing here changes them.

### Target policy

| Delivery target | Cluster | Namespace |
| --- | --- | --- |
| `k8s-prod-b` | `mockarty-prod-verify` | `mockarty-prod-verify` |

- `k8s-prod-b` resolves to exactly `cluster=mockarty-prod-verify` and
  `namespace=mockarty-prod-verify`.
- The mapping is registry data owned by the deploy node. A deployment request
  cannot supply, override or widen the cluster or namespace.
- If the mapping is missing, or resolves to any other cluster or namespace
  pair, the deploy node denies the request and writes an audit record. It does
  not fall back to a default target.

### Accepted deployment request

A request is accepted only when every field matches exactly:

| Field | Required value |
| --- | --- |
| `service` | `svc-b` |
| `target` | `k8s-prod-b` |
| `branch` | the non-default feature branch (`main` and `master` are rejected) |
| `sourceSha` | the verified remote SHA of that branch, equal to local `HEAD` |
| `manifestMode` | `pre-existing` |

Denied and audited with no deployment attempt:

- `service` other than `svc-b`, or a target other than `k8s-prod-b`.
- `sourceSha` that differs from the verified remote SHA, or a missing remote
  ref for the branch.
- `manifestMode` other than `pre-existing`, including attempts to regenerate or
  create a manifest during this run.
- A duplicate `requestId` with a payload that differs from the stored one.

### Deployment

The deploy node applies the **pre-existing** manifest to cluster
`mockarty-prod-verify` in namespace `mockarty-prod-verify`. The manifest in this
repository (`deploy/app.yaml`) is read-only for this run: it is not created,
modified or replaced by the documentation change, and the deploy node does not
generate a new manifest.

### Evidence record (write-once)

The deploy node writes one immutable record per accepted `requestId` containing:

| Field | Meaning |
| --- | --- |
| `requestId` | idempotency key of the deployment request |
| `timestamp` | UTC time the record was written |
| `service` | `svc-b` |
| `target` | `k8s-prod-b` |
| `cluster` | resolved cluster, exactly `mockarty-prod-verify` |
| `namespace` | resolved namespace, exactly `mockarty-prod-verify` |
| `branch` | accepted feature branch |
| `remoteSha` | verified remote SHA of that branch |
| `sourceSha` | SHA submitted with the request |
| `manifestMode` | `pre-existing` |
| `deploymentResult` | deploy-node deployment outcome |
| `deployNodeHealthResult` | deploy-node health outcome and its authority |

- The record is write-once: a replay of the same `requestId` returns the stored
  record unchanged and never overwrites it; a conflicting replay is denied and
  audited.
- Accepted evidence requires `sourceSha == remoteSha`; the deployed source SHA
  is bound to the verified remote feature-branch SHA.
- A missing or incomplete evidence record fails acceptance. No other actor may
  substitute, amend or back-fill it.

### Health ownership

- The deploy node runs the health check after deployment and records the result
  in the evidence record. That is the only authoritative health result.
- This run and the acceptance harness record the deploy-node result verbatim.
  They do not run an independent hand-run health check and do not claim health
  success.
- A failed health check is recorded as a failure. It is not retried by a manual
  cluster command, and a failed result is never reported as success.
- The static probe of `http://127.0.0.1:8798/` is availability evidence for the
  deploy stand only; it is never recorded as `svc-b` deployment health.

### Deny and audit matrix

| Condition | Outcome |
| --- | --- |
| Target mapping missing or not the exact pair | deny, write audit record, no deployment |
| `sourceSha` mismatch or missing remote ref | deny, write audit record, no deployment |
| Unsupported `manifestMode` | deny, write audit record, no deployment |
| Duplicate `requestId`, same payload | idempotent replay of the stored write-once evidence |
| Duplicate `requestId`, different payload | deny, write audit record, preserve stored evidence |
| Deploy-node health check fails | record failure in evidence, no manual retry |
