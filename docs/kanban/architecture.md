# Storage Architecture (Kanban Snapshot)

Context: ArgoCD and Kro are installed. Goal is a lightweight stack using predefined apps that we can compose with Kustomize/Helm and optional Kro graphs.

## Building blocks
- **local-path-provisioner (LPP):** Already scaffolded (`apps/storage/local-path-provisioner`). Managed by Kustomize; can be wrapped in Helm if we choose. Kro not required unless we generate per-node configs dynamically.
- **nfs-subdir-external-provisioner (NSEP):** Add as a separate app under `apps/storage/nfs-subdir-external-provisioner` with a vendored chart (Helm) and Kustomize overlay to point at a bulk export path (e.g., `/mnt/storage-bulk/nfs`). Can be templated via Kro if we want inventory-driven exports per node.
- **Velero + restic:** Add under `apps/storage/velero` with Helm values (S3/minio creds, restic enabled). Kustomize wraps values; Kro can emit backup locations/schedules based on an inventory of namespaces/PVC classes.
- **Kro:** Optional layer to synthesize PV/SC manifests from a drive inventory ConfigMap (fast/bulk). Kro output is consumed by Kustomize; ArgoCD renders with `--enable-helm`/plugin enabled.
- **ArgoCD:** App-of-apps already wired; add new storage apps (NSEP, Velero) under `apps/cicd/argocd/overlays/main/applications/prod/storage/` and include in the prod kustomization.

## Integration patterns
- **Kustomize-first:** Keep base/overlay structure; use HelmChartInflationGenerator for charts (LPP, NSEP, Velero). Keep chart homes in `base/charts` when vendored.
- **Kro for dynamic bits:** Use Kro to read drive inventories and emit SC/PV YAML (including tiered classes `storage-fast`, `storage-bulk`, `storage-local`), or to produce per-namespace Velero schedules. ArgoCD invokes Kro as part of kustomize build.
- **Auth/SSO:** If any UI is exposed (e.g., Velero dashboard via optional plugins or NFS exporter UI), front with authentik via ingress annotations in the overlay.

## Immediate adds (if we proceed)
1) Add `apps/storage/nfs-subdir-external-provisioner` with base + prod overlay, vendored Helm chart, SC `storage-nfs` for RWX.
2) Add `apps/storage/velero` with Helm values (restic enabled, backup storage location), plus ArgoCD app entry.
3) Decide if Kro will drive SC/PV generation; if yes, add the graph and inventory ConfigMap and wire into `apps/storage/overlays/prod` (and enforce class selection per app).
