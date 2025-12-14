# Storage Runbooks (draft)

## Add/replace a drive
1. Update the drive inventory ConfigMap (e.g., `drives.yaml`) with the new device path, capacity, and class tag.
2. Commit/push and let ArgoCD sync the storage app.
3. Verify rendered PVs and StorageClasses: `kubectl get pv,storageclass -n storage-system` (namespace TBD).
4. Rebind dependent PVCs if needed (retain policy keeps data; manual attach/rsync may be required).

## Validate render
- Once Kro graph exists, run: `kustomize build overlays/prod --enable-helm` (with Kro plugin) or via the planned Argo Workflow.

## Troubleshooting
- PV not bound: check node labels/affinity and devicePath correctness.
- Capacity mismatch: ensure `capacityGi` matches the actual filesystem size; adjust and resync.
- Auth issues (future UI): confirm authentik OIDC client is reachable and ingress enforces SSO.
