# MinIO to MinIO v2 migration guide

This guide explains how to run the manual MinIO v2 migration job to copy data from your existing MinIO instance to MinIO v2. Use this when migrating to the new MinIO v2 backend. The deployment is managed by KOTS config changes (e.g. after installing or upgrading the MinIO v2 Helm chart via KOTS).

## Prerequisites

- **Kubernetes cluster** with both source MinIO and MinIO v2 deployed in the same namespace.
- **Secrets** `minio` and `minio-v2` must exist in the target namespace with keys:
  - `minio`: `rootUser`, `rootPassword`
  - `minio-v2`: `rootUser`, `rootPassword`
- **Destination buckets** must already exist on MinIO v2. The job checks for these and fails if any are missing:
  - `knime-accounts-service-avatars`
  - `knime-catalog-service`
  - `knime-customization-profiles`
  - `knime-execution-jobs`
  - `knime-postgres-backup`

## Customizing the namespace

The migration job talks to MinIO and MinIO v2 via in-cluster DNS:

- Source: `http://minio.<namespace>.svc.cluster.local:9000`
- Destination: `http://minio-v2.<namespace>.svc.cluster.local:9000`

You **must** set the namespace to the one where MinIO and MinIO v2 are running.

1. Open the job manifest: [manual-minio-v2-migration-job.yaml](./manual-minio-v2-migration-job.yaml).

2. Set the job’s namespace in **two** places:

   **a) Job metadata (where the Job runs):**

   ```yaml
   metadata:
     name: minio-v2-migration
     namespace: <YOUR_NAMESPACE>   # e.g. knime, knime-hub
   ```

   **b) Environment variable (used for service DNS):**

   ```yaml
   env:
     - name: NAMESPACE
       value: <YOUR_NAMESPACE>   # must match the namespace above
   ```

   Replace `<YOUR_NAMESPACE>` with your actual namespace (e.g. `knime`, `knime-hub`). By default the job uses `knime` namespace as this is the default for KNIME Hub installations.

3. Ensure the secrets `minio` and `minio-v2` exist in that same namespace and that the job’s `secretKeyRef` names and keys match your setup (defaults are `minio` / `minio-v2` with `rootUser` and `rootPassword`).

## Running the migration

1. Save the edited job manifest (e.g. `manual-minio-v2-migration-job.yaml`).

2. Apply the job:

   ```bash
   kubectl apply -f manual-minio-v2-migration-job.yaml
   ```

3. Watch the job and logs:

   ```bash
   kubectl -n <YOUR_NAMESPACE> get jobs
   kubectl -n <YOUR_NAMESPACE> logs job/minio-v2-migration -f
   ```

4. The job:
   - Configures `mc` aliases for source and destination.
   - Verifies connectivity to both MinIO and MinIO v2.
   - Checks that all required buckets exist on the destination.
   - Mirrors each bucket from source to destination with `mc mirror --overwrite`.

   On success you’ll see: `MinIO migration completed successfully.`

## Troubleshooting

- **"Cannot access source MinIO"** – Check that MinIO is running in `<YOUR_NAMESPACE>` and that the `minio` secret has the correct `rootUser`/`rootPassword`.
- **"Cannot access destination MinIO"** – Check that MinIO v2 is running in the same namespace and that the `minio-v2` secret is correct.
- **"Destination bucket … does not exist"** – Create the missing bucket(s) on MinIO v2, then re-run the job (you may need to delete the failed job first: `kubectl -n <YOUR_NAMESPACE> delete job minio-v2-migration`).

After a successful run, you can switch your Hub configuration to use MinIO v2 and remove or decommission the old MinIO when ready.
