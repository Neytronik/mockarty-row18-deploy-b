# Row #18 MSDeploy Record

Documentation-only record for Row #18. This page describes the delivery
handoff; it does not instruct anyone to deploy by hand.

## Row #18 — svc-b

| Field | Value |
| --- | --- |
| Service | `svc-b` |
| Delivery target | `k8s-prod-b` |
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
