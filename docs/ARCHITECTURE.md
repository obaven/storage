# Storage Orchestrator Architecture (draft)

## Objectives
- Present two node-local drives as Kubernetes storage with clear class separation (`fast`, `bulk`).
- Stay lightweight (no distributed cluster storage) while remaining robust (affinity/health checks and clear ownership).
- Integrate with ArgoCD (gitops), Kro (templating), Helm/Kustomize (render), authentik (future SSO for any UI/API).

## Proposed design
1. **Predefined, lightweight apps (initial stack)**
   - **local-path-provisioner** (Rancher) for local PV dynamic provisioning; configured with two host mount points (fast/bulk) on the storage node.
   - (Roadmap) **Kro graph** to generate additional StorageClasses/PVs from a drive inventory ConfigMap when we need finer control.
   - (Optional later) **nfs-subdir-external-provisioner** for shared RWX if/when needed.

2. **local-path-provisioner wiring (present in repo)**
   - Runs in `storage-system` namespace with RBAC and a single Deployment.
   - ConfigMap defines paths per node; default config points at `/mnt/storage-fast` and `/mnt/storage-bulk` on `obaven-alpha`.
   - StorageClasses:
     - `storage-local` (default, WaitForFirstConsumer, Retain, expansion) → general use.
     - `storage-fast` (pinned to `/mnt/storage-fast`) → latency-sensitive apps (DB primaries, Vaultwarden, Forgejo).
     - `storage-bulk` (pinned to `/mnt/storage-bulk`) → capacity-focused apps (Harbor registry, backups, mail data).
   - Consumers pick a class via `storageClassName`.

3. **Kustomize/Helm glue**
   - The app lives at `apps/storage/local-path-provisioner` with base manifests (no external chart dependency).
   - Environment overlay `apps/storage/local-path-provisioner/overlays/prod` patches the ConfigMap for the two drives.

4. **ArgoCD & workflows**
   - ArgoCD application `storage-prod` targets `apps/storage/overlays/prod` (aggregates storage components).
   - Optional Argo Workflow (future) to render Kro graph/helm and alert on errors.

5. **Security & auth**
   - Use authentik/OIDC on any future admin UI/API (none required for the current headless provisioner).
   - Restrict access to `storage-system` via dedicated service account/RBAC; no ingress exposed.

6. **Consumption pattern**
   - Apps request PVCs using `storageClassName: storage-local|storage-fast|storage-bulk`. Capacity and node placement are governed by the local-path config (mount points per node).
   - When Kro graphs are added, we can emit per-app PVC templates or additional classes and add validation (e.g., ensure Harbor uses `storage-bulk`).

## Workload-to-class guidance (assembly-style layering)
- Root domains (`x`) are top-level app domains (cicd, networking, security, data, storage, web, auth), keep `1 <= x <= 7`. Sub-creations (`n`) are per-app storage units. Introduce `H` as the optimal logical growth bound (minimum 3, upper bound determined by app needs).
- Shape definitions (lightweight “assembly” nesting):
  - `s1(x,n) = {x,H,n}` ⇒ for each root, offer three tiers at minimum (`storage-local`, `storage-fast`, `storage-bulk`) with `H >= 3` for logical growth if more tiers are ever added.
  - `s2(x,n) = {s1(x,n), x,H,n}` ⇒ repeat the tier choice per app/component (DB/data/logs).
  - `s(R,M,x,n) = [repeat s2(x,n) R times for M iterations]` ⇒ apply the same tier decision for each replica/namespace as you scale.
- Practical mapping (pick the smallest tier that fits):
  - **storage-fast**: Postgres primaries (prod/dev), Vaultwarden (prod/dev), Forgejo app + Redis, Authentik (when restored), latency-sensitive caches, anything with high IOPS.
  - **storage-bulk**: Harbor registry/Trivy, mailserver data, long-lived backups/archives, large object stores.
  - **storage-local**: Small configs/queues/logs, DHCP leases, minor services where either tier is overkill.
  - **postgres-local-storage** (static): Keep for the existing 1Ti PV or migrate it into `storage-fast`/`storage-bulk` based on device choice.
- Kro validation (future): enforce per-app class (e.g., Harbor → bulk, Postgres/Vaultwarden/Forgejo → fast) and emit PVC templates accordingly.
## Open questions / next steps
- Confirm host mount points and perms on storage node (`/mnt/storage-fast`, `/mnt/storage-bulk`) and ensure taints/labels align.
- Add Kro graph for richer inventory-driven PVs (per-drive PVs with nodeAffinity).
- Decide if we need RWX (NFS provisioner) or snapshots/backup integration (Velero/restic).
