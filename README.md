# Remote Development for GPU-Powered Containers and VMs

Secure, private access to GPU-powered containers and VMs running in the cloud from your laptop — without exposing them to the public internet.

Two approaches are documented in this repository:

| Approach | Best for | Details |
|---|---|---|
| **Tailscale** | Full IP connectivity, near line-rate P2P performance, identity-based SSH (no key management), free for personal use | [README-DEV.MD](README-DEV.MD) |
| **VS Code Remote - Tunnel** | Occasional access and lightweight development, direct access from VS Code and a web browser, less intrusive | [README-RT.MD](README-RT.MD) |

## Approach 1: Tailscale

Tailscale creates a private WireGuard-based tailnet between your laptop and the container. The container registers itself as a Tailscale machine and is then reachable directly by its tailnet IP.

### Quick Start

1. Create a personal Tailscale account.
2. Install and run Tailscale on your laptop (`sudo tailscale up`).
3. Generate a reusable, ephemeral auth key in the [Tailscale admin console](https://console.tailscale.com/admin/settings/keys).
4. Store the key in a Kubernetes Secret (`rs-ts-auth-key` → `TS_AUTH_KEY`).
5. Deploy the dev-server manifest ([dev-server.yaml](dev-server.yaml)) — it installs Tailscale inside the pod, registers it in your tailnet, and enables Tailscale SSH.
6. Access the pod from your laptop: `tailscale ssh root@<pod-tailscale-ip>`.

Full step-by-step instructions with example session output are in [README-DEV.MD](README-DEV.MD).

### Key Implementation Details

- **`/dev/net/tun`** is mounted into the pod (hostPath) — required by Tailscale's WireGuard networking.
- The container runs with **`NET_ADMIN`** (Tailscale) and **`IPC_LOCK`** (RDMA/GPU workloads) capabilities.
- The Tailscale auth key is injected from a Kubernetes Secret, never hardcoded in manifests.
- `tailscale up` uses the pod name as the machine hostname; DNS resolution is left to Kubernetes CoreDNS.
- For production use, bake the Tailscale client into a custom image instead of installing it at container start.

## Approach 2: VS Code Remote - Tunnel

Built on top of Microsoft Dev Tunnels. Less intrusive than Tailscale — no networking changes inside the container, but access is limited to VS Code (desktop and browser). Free to use, with monthly limits on active tunnels and traffic; well suited for occasional access and lightweight development.

### Quick Start

1. Install the VS Code CLI in the container (access it via the web console, `kubectl exec`, or any other supported method):

   ```bash
   curl -Lk 'https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-x64' -o vscode_cli.tar.gz && \
     tar -xf vscode_cli.tar.gz && \
     mv code /usr/local/bin/code && \
     rm vscode_cli.tar.gz
   ```

2. Authenticate the tunnel with your GitHub account, then start it:

   ```bash
   code tunnel user login --provider github
   nohup code tunnel --accept-server-license-terms --name 001 &> output.log &
   ```

3. Connect from VS Code Desktop or a browser: `https://vscode.dev/tunnel/001`

Full details are in [README-RT.MD](README-RT.MD).

## Repository Layout

```
.
├── README-DEV.MD            # Tailscale integration guide (step-by-step)
├── README-RT.MD             # VS Code Remote - Tunnel guide
├── dev-server.yaml          # StatefulSet: GPU dev server with Tailscale
├── rs_tailscale/            # Supporting manifests
│   ├── dev-pod-1-gpu.yaml       # Plain GPU pod (no NFS)
│   ├── dev-pod-1-gpu-nfs.yaml   # GPU pod with NFS-backed storage
│   ├── nfs-pv.yaml              # Static PersistentVolume for a DO NFS export
│   └── nfs-pvc.yaml             # PersistentVolumeClaim bound to the PV
├── scenario.png             # Architecture diagram
└── key.png                  # Auth key generation in the admin console
```

## Prerequisites

- A Kubernetes cluster with GPU nodes (tested on DOKS with AMD GPU nodes, e.g. `doks.digitalocean.com/gpu-brand: amd`)
- `kubectl` configured for the target cluster
- Tailscale installed on your laptop (Approach 1)
- A GitHub (or Microsoft) account (Approach 2)

## Resources

- [Tailscale Solution Overview](https://docs.salad.com/container-engine/explanation/platform-integrations/tailscale-integration)
- [Add Tailscale Support into Workloads](https://docs.salad.com/container-engine/how-to-guides/platform-integrations/tailscale-basic)
- [Tailscale Plans](https://tailscale.com/pricing)
- [VS Code Remote - Tunnels](https://code.visualstudio.com/docs/remote/tunnels)

## License

[MIT](LICENSE) — Copyright (c) 2026 rxsalad
