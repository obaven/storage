# Storage Planning

## Objectives
From `kanban/storage-refactor/00_OBJECTIVES.md`:
1.  **Decentralize Security**: Move secret rotation configuration to component-specific configs. [IN PROGRESS]
2.  **Automate Sync**: Ensure credentials rotated by `rotato` are automatically synchronized. [IN PROGRESS]

## Backlog
- [ ] **Verify MinIO Rotation**: Manual intervention needed to create base credentials in Vaultwarden due to `rotato` limits.
- [ ] **Fix Rotato Error**: Investigate "OS error 2" during `create-items` (likely `kubeseal` or `git` interaction).
- [ ] **Expand Documentation**: Finalize `README.md` and `RUNBOOKS.md`.
- [ ] **Kro Integration**: Add Kro ResourceGraphDefinition for dynamic PV provisioning (future).
- [ ] **Monitoring**: Add alerts for storage capacity and provisioner health.

## Issues
- `rotato create-items` fails with "OS error 2".
- CLI automation for Vaultwarden (`bw`) hangs in non-interactive sessions.
