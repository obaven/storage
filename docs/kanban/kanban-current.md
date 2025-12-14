# Storage Orchestrator – Kanban

## Now
- Clear node disk pressure so ArgoCD/storage pods can schedule (`node.kubernetes.io/disk-pressure:NoSchedule` on obaven-alpha).
- Host prep: ensure `/mnt/storage-fast` and `/mnt/storage-bulk` exist with correct perms.
- Apply/sync `storage-prod` (local-path-provisioner SC `storage-local`).
- Assign SC for pending PVCs (postgres dev/stage/prod second replica) to unblock bindings.

## Next
- Patch workloads to use `storage-local` (forgejo, harbor, mailserver, redis dev/prod, vaultwarden dev/prod, kea-dhcp, postgres dev).
- Create rsync jobs/templates to migrate data PVC→PVC with minimal downtime; cut over per app.
- Decide fate of `postgres-local-storage` 1Ti PV (migrate or keep).
- Add alerts on `/mnt/storage-fast` and `/mnt/storage-bulk` (disk/inode thresholds).

## Later
- Add Kro graph for inventory-driven PVs/StorageClasses (`fast`/`bulk`) and per-app PVC templates.
- Add RWX option (nfs-subdir-external-provisioner) if needed.
- Add Velero + restic for backups/snapshots of critical PVCs.
- Argo Workflow smoke test for storage renders (Kustomize/Helm/Kro).
- Optionally deprecate old `local-path` SC after migration and cleanup Released PVs.
