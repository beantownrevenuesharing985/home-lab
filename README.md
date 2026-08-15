# home-lab

Kubernetes manifests for the single-node **k3s** cluster I run at home. Everything
here is applied to a live cluster — self-hosted notes, monitoring, LLM inference,
an AI gateway, and a small production app for a community site.

Plain YAML, applied with `kubectl apply`. No Helm, no Kustomize: the cluster is
one node and one operator, and I'd rather read the actual objects than a chart's
rendered output.

## Cluster

| | |
|---|---|
| Distribution | k3s (single node, on Proxmox) |
| Ingress | Traefik (bundled with k3s) |
| Load balancer | MetalLB in L2 mode, pool `192.168.68.200-210` |
| Storage | `local-path` for per-app PVCs; NFS PV (900Gi) for shared/bulk data |
| Monitoring | Prometheus + node-exporter + Grafana |
| DNS | Cloudflare, wildcard record onto the cluster |

## Topology

```
                    internet
                        │
                  Cloudflare DNS
                        │
              ┌─────────▼─────────┐
              │  Traefik ingress  │   (k3s built-in)
              └─────────┬─────────┘
      ┌───────────┬─────┴─────┬───────────┬───────────┐
      │           │           │           │           │
  ┌───▼───┐  ┌────▼────┐ ┌────▼────┐ ┌────▼────┐ ┌────▼────┐
  │affine │  │ hermes  │ │ 9router │ │ grafana │ │polashbari│
  │(notes)│  │ (agent) │ │(gateway)│ │(monitor)│ │  (app)  │
  └───┬───┘  └────┬────┘ └─────────┘ └────┬────┘ └────┬────┘
      │           │                       │           │
  ┌───▼────┐  ┌───▼────┐            ┌─────▼─────┐ ┌───▼────┐
  │postgres│  │ ollama │            │prometheus │ │ sqlite │
  │ +redis │  │(llama3)│            │+node-exp. │ │  PVC   │
  └───┬────┘  └────────┘            └───────────┘ └────────┘
      │
  ┌───▼──────────────────────────────────────────┐
  │  NFS PV — 900Gi, ReadWriteMany, 192.168.68.103│
  └───────────────────────────────────────────────┘
```

## Workloads

| Directory | What it runs | Notes |
|---|---|---|
| `metallb/` | MetalLB controller + IP pool | Split install/config so the pool can be re-applied alone |
| `monitoring/` | Prometheus, node-exporter (DaemonSet), Grafana | Prometheus has a scoped read-only ClusterRole |
| `storage/` | NFS PersistentVolume + claim | `ReadWriteMany`, backs the bulk data |
| `affine/` | AFFiNE notes app + Postgres + Redis | Multi-container app, SMTP, first-boot admin seeding |
| `ollama/` | Ollama (llama3.2:3b) | In-cluster LLM inference, reused by other apps |
| `anythingllm/` | AnythingLLM | RAG UI pointed at the in-cluster Ollama service |
| `9router/` | 9Router | OpenAI-compatible AI gateway; see its own README |
| `omniroute/` | OmniRoute | Second multi-provider AI gateway |
| `hermes/` | Hermes agent | Telegram-driven agent with scoped in-cluster `kubectl` |
| `polashbari/` | amaderpolashbari | Small production app (Astro + SQLite), own container image |
| `uptime/` | Uptime Kuma | External uptime checks |

## Secrets

No credentials are committed. Every app that needs them ships a
`secret.example.yaml`; the real `secret.yaml` is gitignored and lives only on
the node.

```bash
cd <app>
cp secret.example.yaml secret.yaml
$EDITOR secret.yaml          # fill in real values
kubectl apply -f secret.yaml # apply BEFORE the app manifest
```

## Applying

```bash
# infrastructure first
kubectl apply -f metallb/metallb.yaml
kubectl apply -f metallb/config.yaml
kubectl apply -f storage/nfs-pv.yaml
kubectl apply -f monitoring/monitoring.yaml

# then any app
kubectl apply -f <app>/secret.yaml
kubectl apply -f <app>/<app>.yaml
kubectl -n <app> rollout status deploy/<app>
```

## Things that took real debugging

A homelab is mostly a debugging exercise. The ones worth writing down:

- **CoreDNS + external TLS.** Several pods failed outbound HTTPS with
  `ERR_SSL_VERSION_OR_CIPHER_MISMATCH`. The node's own search domain is my
  wildcard-routed domain, so with `ndots: 5` every lookup — internal service
  names included — got the wildcard suffix appended and resolved to the wrong
  IP. `dnsPolicy: None` alone breaks internal resolution instead. The fix is
  `dnsPolicy: None` with an explicit config: CoreDNS first for cluster names,
  public resolvers for external, and only the real cluster search suffixes.
  See the comment block in `anythingllm/anythingllm.yaml`.
- **`kubectl` inside a pod without shipping a binary.** The k3s binary
  dispatches on `argv[0]`, so hostPath-mounting `/usr/local/bin/k3s` as
  `kubectl` gives a pod a working kubectl with no extra download
  (`hermes/hermes.yaml`).
- **Kubernetes `$(VAR)` expansion.** `$(PATH)` in an env value only expands
  vars defined in the same `env:` list, not ones baked in via Dockerfile `ENV`.
  It silently produced a literal broken string and wiped the path to the app's
  own binary.
- **SQLite on RWO volumes.** Every SQLite-backed app uses
  `strategy: Recreate` — the default RollingUpdate briefly runs two pods
  against one single-writer database.
- **Least privilege by default.** `automountServiceAccountToken: false`
  everywhere it isn't needed; the agent that does need cluster access is bound
  to the built-in `view` ClusterRole, which excludes Secrets.

## Known gaps

Honest list of what I'd fix next:

- Ingresses terminate on the `web` entrypoint (TLS is handled upstream) —
  cert-manager and end-to-end TLS are not set up yet.
- Grafana and Uptime Kuma rely on their own login pages; only the Hermes
  dashboard sits behind a Traefik basic-auth middleware.
- Single node, so `local-path` PVCs are node-local and best-effort. Only the
  NFS-backed data is durable.
- No CI: manifests aren't linted or validated on push.
- ArgoCD was set up and then removed. Moving this repo to real GitOps
  (Argo watching this repo) is the next thing I want to do.
