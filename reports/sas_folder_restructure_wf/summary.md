# sas_folder_restructure_wf: pkf create

Run status: **ok** (2026-10-21T17:12:19+00:00)

| stage | name | status | cost (USD) | note |
|---|---|---|---|---|
| C0 | preflight | ok | 0.0000 |  |
| C1 | analyse | ok | 0.8121 |  |
| C2 | components | ok | 0.1649 |  |
| C3 | units | ok | 0.8501 |  |
| C4 | build units | ok | 0.0000 |  |
| C5 | components tested | ok | 0.0000 |  |
| C6 | pipeline class | ok | 0.0000 |  |
| C7 | DAG | ok | 0.0000 |  |
| C8 | infra | ok | 0.0000 | nothing to apply |
| C9 | dev run | skipped | 0.0000 | no usable AWS credentials: no valid AWS credentials (ExpiredToken): run aws sso login --profile my-custom-profile (or source ~/pkf/env.sh), then try again |

## What was built

| component | unit | kind | functions | cases | quality |
|---|---|---|---|---|---|
| trigger `sas_folder_restructure_schedule` | `scheduled_run` | schedule | — | — |  |
| source `sas_incoming` | `list_incoming` | list | — | — |  |
| source `sas_incoming` | `skip_unwanted` | select | — | — |  |
| source `sas_incoming` | `one_package_per_file` | split | — | — |  |
| destination `sas_study_bucket` | `route_by_folder_rules` | custom | split_path, data_format, underscore_format, matching_rules, ia_label, target_cdap, target_anon, target_cdar, decide_route | 52 | ok |
| destination `sas_study_bucket` | `move_and_tag` | custom_delivery | is_error_target, tag_value_ok, report_line | 15 | ok |

## Assumptions

- analyst: Path-contains checks (/data_reuse/, /cdap_, /adam/, raw/, _anon_, ...) are made on the folder path relative to incoming/, with a '/' added at the start and at the end, so that a first or last folder can still match.
- analyst: Rules 1, 2, 3, 5 and 6 ('a data format') use the slash-style format (/adam/, /sdtm/, raw/). Only rule 4 (ANON) uses the underscore format.
- analyst: The underscore format follows the same order as the slash format: _adam_, then _sdtm_, then _raw_ or _restricted_raw_.
- analyst: Moving a file to error/ is also a move: server-side copy, check, then delete the source. If that fails, the file fails and so does the run.
- analyst: 'Confirm the target exists' means a HEAD on the target returns an object of the same size as the source.
- analyst: The server-side copy keeps the object's metadata. Files moved to error/ keep the tags copied from the source unchanged ('no tag change').
- analyst: If the delete fails after a successful copy, the target stays in place and the source stays in incoming/. The next run copies it again, overwriting the target.
- analyst: A file name has an extension when it contains a '.'.
- analyst: The 'contains _ia' check on the CDAP last folder is case-sensitive.
- analyst: CDAP interim: if the third part starts with 'ia', that 'ia' is replaced by 'ia_'. Otherwise 'ia_' is put in front of the whole part.
- analyst: The per-file report goes to the run log.
- analyst: Runs do not overlap: at most one run is active at a time.
- analyst: The cron '*/45 * * * *' is used as written (it fires at :00 and :45 of each hour).
- split: The schedule start date (2026-10-01) and the retry policy (2 retries, 2 minutes apart) are described in the trigger requirement, because the trigger settings have no keys for them.
- split: One package per file: the source splits by relative path and sets the variable file_key, which is used as the package name.
- split: The bucket name is set per environment, so the config holds <REQUIRED>. The development value is given in the source requirement.
- split: Delivery overwrites existing target objects (if_exists: overwrite) and is done by a custom_delivery step (move, check, tag).
- split: Source and destination are the same bucket, with no transformation step.
- trigger/sas_folder_restructure_schedule plan: The cron expression, the start date 2026-10-01, the retries (2, 2 minutes apart) and at most one active run are pipeline/DAG schedule settings. The `schedule` step takes no parameters for them.
- trigger/sas_folder_restructure_schedule plan: The R-RUN clauses and the listing of incoming/ (R-TRG-2) are carried out by the source and destination components. This trigger only provides the one event per run that starts them, so it cites those clauses for coverage.
- trigger/sas_folder_restructure_schedule plan: "*/45 * * * *" is standard cron, so it fires at minutes 0 and 45 of every hour, not strictly every 45 minutes.
- source/sas_incoming plan: The R-RUN clauses (server-side copy above 5 GiB, existence check, delete, overwrite, tag replacement, warning-only tagging, no tags in error/, report lines, run outcome) are carried out by the destination's custom_delivery step. The source steps cite them because they provide the per-file S3 reference, size and file_key that delivery needs, and the one-file-per-package isolation.
- source/sas_incoming plan: A name 'has an extension' when its last path segment contains a '.' anywhere. Dotfiles like '.keep' and names ending in '.' therefore count as having one.
- source/sas_incoming plan: Zero-byte files inside a study folder that have an extension are kept, like any other file.
- source/sas_incoming plan: The bucket name comes from the per-environment S3 connection settings and is never written in the plan.
- destination/sas_study_bucket plan: All contains checks run on the folder path relative to incoming/ (without the file name), wrapped in a leading and a trailing '/'. The file name is never checked.
- destination/sas_study_bucket plan: Study = the first folder of the package path. Package paths are already relative to incoming/; the source key is incoming/ + path.
- destination/sas_study_bucket plan: CDAP interim: the third '_' part is written as 'ia_' + the part with a leading 'ia' (any case) and any leading '_' removed. If the part does not start with 'ia', the whole part goes after 'ia_'. An empty rest gives an error.
- destination/sas_study_bucket plan: CDAR: the first folder named 'ar' (case-insensitive) is used. The reporting level is written in the key as it appears in the path; the lowercase form is used only for the comparisons.
- destination/sas_study_bucket plan: ANON 'last part' = the text after the last '_' in the last folder. ANON with _anon_csr is checked against the whole wrapped folder path.
- destination/sas_study_bucket plan: OTHERS matches only on /data_reuse/, a data format and the absence of /med, /cdap, /idap, /anon and _anon_. The CDAR marker /cdar/ does not exclude it, so data_reuse+cdar paths match two rules and go to error/, as the worked example shows.
- destination/sas_study_bucket plan: A delete that fails after a good copy gives status 'failed' (the run fails, as the requirement says). The copied target is left in place.
- destination/sas_study_bucket plan: When the tag value (the path) is not a valid S3 tag value (more than 256 characters or characters S3 does not allow), tagging is skipped with a warning and the file still counts as moved. The value is not truncated.
- destination/sas_study_bucket plan: The delivery step calls decide_route again to get the rule or error reason for its report line. The function is pure, so it gives the same result as in the routing step.
- destination/sas_study_bucket plan: Delivery policies stay as configured: overwrite, no SSE, no success marker.

## Deferred tests

- destination/sas_study_bucket · route_by_folder_rules · `fn:route_key_too_long`: deferred after it failed (Studio user, 2026-10-21T17:09:38+00:00)
- destination/sas_study_bucket · move_and_tag · `fn:tag_len_256_ok`: deferred after it failed (Studio user, 2026-10-21T17:09:39+00:00)

## Costs

This run: 1.8271 USD. Recorded for the pipeline: 6.8417 USD of a 25.00 USD budget.
