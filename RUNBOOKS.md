# Storage Runbooks

## MinIO Secret Rotation (Manual Workaround)
Due to `rotato create-items` failure, initial credential creation requires manual steps.

1.  **Create Credentials**:
    - Log in to Vaultwarden.
    - Create a new item named `minio-creds` in the `Storage` folder.
    - Add custom fields:
        - `rootUser`: (static value, e.g., `admin`)
        - `rootPassword`: (generate a strong password)
    - Note the **Item ID** (Cipher ID).

2.  **Configure Rotation**:
    - Edit `apps/storage/minio/rotation.yaml`.
    - Update `vaultwarden.cipherId` with the ID from step 1.
    - Ensure `hooks.post` is correctly defined to sync to Velero.

3.  **Trigger Rotation**:
    - Run `rotator_helper rotate` (or wait for scheduled job).
    - Verify `minio-creds` secret is updated in `storage-system` namespace.
    - Verify `velero-s3-creds` is created/updated via the hook.

## Restoring Backup
(To be added)

## Adding New Storage Nodes
(To be added: Mount points setup, adjusting overlays)
