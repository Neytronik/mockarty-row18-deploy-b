# svc-b deployment marker

row18-k8s-deploy-live-ok svc-b k8s-live-b

## Delivery record

- Service: svc-b
- Repository: https://github.com/Neytronik/mockarty-row18-deploy-b.git
- Delivery target: k8s-live-b
- Deploy owner: Mockarty deploy node (owns the post-deploy health window)
- Marker contract: the marker line above is the single machine-readable deploy token; keep it on one line, unwrapped and unduplicated.

This file is the documentation-only deploy marker for row #18. The change author
does not deploy; the Mockarty deploy node deploys the accepted commit of this
branch to the delivery target above and records the post-deploy health window.
