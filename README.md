# Mockarty row #18 — deploy acceptance fixture (service B)

Disposable public repository used by a live acceptance run of Mockarty's
autonomous SDLC deploy arm:

- `deploy/app.yaml` is the artifact the Mockarty **kubernetes deployer** applies
  to the Robolance stage cluster (namespace `mockarty-deploy-live`).
- The coder mission adds `docs/msdeploy.md` (row marker) on its own branch; the
  deploy node then ships the manifest from that exact commit.

Nothing here is a product of the fixture repository; it exists only to prove the
deploy pipeline end-to-end and is deleted after the run.
