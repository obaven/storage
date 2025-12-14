# Storage Application

Goal: lightweight, gitops-managed storage orchestration using predefined apps (starting with local-path-provisioner) that maps our two node-attached drives into Kubernetes via Kustomize, and integrates with ArgoCD and existing platform services (authentik, etc.).

## Approach (draft)
- **Predefined app first**: use `local-path-provisioner` (lightweight) to expose local PVs from two drive mount points; `StorageClass` is `storage-local` (default, GitOps-labeled).
- **Inventory-ready**: keep room to add a Kro graph to generate per-drive PVs/StorageClasses when needed.
- **Integration**: ArgoCD manages the storage app (`storage-prod`). Argo Workflows (future) can re-render tests. authentik can front any future UI/API.
- **Lightweight**: no distributed storage; uses local PVs with `WaitForFirstConsumer`, simple hostPath/FS mount, and templated manifests.

## Layout
- `local-path-provisioner/`: Local storage provisioner.
- `minio/`: Object storage (S3 compatible).
- `velero/`: Backup and disaster recovery.
- `nfs-server/` & `nfs-subdir-external-provisioner/`: Network file storage.
- `overlays/prod/`: Aggregates all storage components for the `storage-prod` ArgoCD app.
- `docs/`: architecture notes and inventory examples.
- `PLANNING.md`, `RUNBOOKS.md`, `STATUS.md`: day-2 and backlog notes.

## Next
- Confirm host mount points (`/mnt/storage-fast`, `/mnt/storage-bulk`) on storage node(s); adjust overlay config.
- Add Kro ResourceGraphDefinition for per-drive PVs/StorageClasses if we outgrow local-path-provisioner.
- Add Argo Workflows smoke test and integrate authentik if we add a UI/API.
