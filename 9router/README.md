# 9Router on k3s

Self-hosted AI model router / OpenAI-compatible proxy. Routes requests from CLI tools
(Cursor, Cline, Claude Desktop, Codex, RooCode, Continue, ...) through subscription /
cheap / free tier fallback with real-time quota tracking.

- Image: `9router/9router:latest`
- Dashboard: <http://router.mdmahfujarrahman.com/>
- OpenAI-compatible endpoint: `http://router.mdmahfujarrahman.com/v1`
- In-cluster endpoint: `http://nine-router-service.9router.svc:20128/v1`

## One-time apply

```bash
# 1. namespace (idempotent)
sudo k3s kubectl create namespace 9router --dry-run=client -o yaml | sudo k3s kubectl apply -f -

# 2. generate + create Secret
JWT=$(openssl rand -hex 32)
PW=$(openssl rand -base64 18 | tr -d '/+=' | cut -c1-24)
sudo k3s kubectl -n 9router create secret generic 9router-secrets \
  --from-literal=jwt-secret="$JWT" \
  --from-literal=initial-password="$PW"
echo "Dashboard initial password: $PW"   # copy this, log in, change it immediately

# 3. apply the workload
cd k8s-manifests/9router
sudo k3s kubectl apply -f 9router.yaml

# 4. wait + verify
sudo k3s kubectl -n 9router rollout status deploy/9router-deployment
sudo k3s kubectl -n 9router get svc,ingress,pvc
curl -s http://router.mdmahfujarrahman.com/health
```

## Pointing a CLI tool at it

Set the OpenAI base URL to `http://router.mdmahfujarrahman.com/v1` (or the in-cluster
service URL above for tools running inside the cluster) and use any API key — the
dashboard will show usage per key. Configure providers (Claude Code, OpenAI, Gemini,
GLM, MiniMax, Kimi, iFlow, Qwen, Kiro, ...) in the dashboard after first login.

## First-login checklist

- Log in with the password from step 2 (printed once).
- **Change the dashboard password immediately** (Settings → Change Password).
- Add at least one provider via the dashboard.
- Grab an API key from the dashboard for your CLI tools.

## Persistence

Data lives in `db.json`, `api-keys.json`, and `logs/` under the `/data` mount, backed
by a `local-path` PVC (`9router-data`, 1Gi). On a single-node k3s cluster this is
node-local and best-effort — if the node dies the data dies with it. Snapshot
`/var/lib/rancher/k3s/storage` (or wherever local-path stores its pvc root) for
backups, or migrate to the existing NFS PV if you want shared/durable storage.

## Rotating the JWT secret

```bash
sudo k3s kubectl -n 9router create secret generic 9router-secrets \
  --from-literal=jwt-secret="$(openssl rand -hex 32)" \
  --from-literal=initial-password="$(sudo k3s kubectl -n 9router get secret 9router-secrets -o jsonpath='{.data.initial-password}' | base64 -d)" \
  --dry-run=client -o yaml | sudo k3s kubectl apply -f -
sudo k3s kubectl -n 9router rollout restart deploy/9router-deployment
```