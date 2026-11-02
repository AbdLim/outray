# Public edge deployment

This repository's release pipeline builds the tunnel edge, CLI and SDKs. Deployment scripts manage only the edge and its gateway routing, not unrelated services or database migrations.

`Dockerfile` builds the edge image. The GitHub image workflow publishes `ghcr.io/outray-platform/outray-edge:<full-source-commit>` for amd64/arm64. Deployment pipelines pin the immutable registry digest and verify the OCI revision label.

Every push to `main` automatically promotes the hosted VPS edge after the Woodpecker install, build, and test steps pass. Manual reruns remain available on `main`; pull requests and other branches do not receive deployment credentials or deploy. The existing `edge_ssh_key` repository secret must allow **Push** and **Manual** events, with pull-request exposure disabled.

`deploy/woodpecker/deploy-edge.sh` uploads a unique `/root/outray-edge/releases/<commit>-<id>` artifact and holds `/var/lock/outray-deployment.lock` during promotion. All deployers sharing the host must respect this lock and use separate artifact directories. Existing edge runtime settings are retained from the active PM2 edge. Deployment starts the inactive slot and checks its health before switching Caddy; the old process is retired after the switch, so long-lived connections may reconnect.

The blue/green deployment preserves the authenticated control-plane API, status forwarding, Redis queues and accounting adapters.

For local edge development, configure `.env.example` against a compatible control plane.
