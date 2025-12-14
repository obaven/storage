# Storage Status

## Components
| Component | Type | Status | Notes |
|Str|Str|Str|Str|
| `local-path-provisioner` | Submodule | Active | Primary storage class `storage-local` |
| `minio` | Submodule | Active | S3 Compatible Storage |
| `velero` | Submodule | Active | Backup & Restore |
| `nfs-server` | Submodule | Active | Generic NFS |
| `nfs-subdir-external-provisioner` | Submodule | Active | NFS Storage Class |

## Features
- **Modularization**: All components split into git submodules (`obaven/storage-*`).
- **Rotation**:
    - MinIO: Config decentralized to `apps/storage/minio/rotation.yaml`.
    - Hook: Post-rotation hook defined to sync to Velero.
    - Status: **Active** (Automated via `rotato` with CLI integration).
- **GitOps**: Managed via ArgoCD "App of Apps" pattern (`apps/storage/overlays/prod`).
