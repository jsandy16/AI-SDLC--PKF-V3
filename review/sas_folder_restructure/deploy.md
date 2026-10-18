# Deploying `sas_folder_restructure`

1. Apply the approved dev plan (pkf Studio S6 does this).
2. Copy `dags/pkf_sas_folder_restructure.py` and `dags/pkf/sas_folder_restructure/` into the Airflow DAGs folder (the bundle's 04_code/dags).
3. Install the runtime on the workers: `pip install 'package-factory[runtime]'` (same version as the build).
4. Give the Airflow worker role the policy in `infra/<p>/iam-policy.json` (05_infra/iam-policy.json).
5. Set PKF_STAGING_URL=s3://<bucket>/<prefix> on the workers (with a lifecycle rule) unless Airflow runs on one machine: every task of a run must reach the same staging.
6. Unpause the DAG; the first run should match the dry run in 07_runs/dry-run.json.

Resources: nothing to apply — policy only.
