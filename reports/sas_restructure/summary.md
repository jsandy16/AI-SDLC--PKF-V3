# sas_restructure: pkf create

Run status: **ok** (2026-10-22T08:52:15+00:00)

| stage | name | status | cost (USD) | note |
|---|---|---|---|---|
| C0 | preflight | ok | 0.0000 |  |
| C1 | analyse | ok | 0.7223 |  |
| C2 | components | ok | 0.1491 |  |
| C3 | units | ok | 0.0000 |  |
| C4 | build units | ok | 0.0000 |  |
| C5 | components tested | ok | 0.0000 |  |
| C6 | pipeline class | ok | 0.0000 |  |
| C7 | DAG | ok | 0.0000 |  |
| C8 | infra | ok | 0.0000 | nothing to apply |
| C9 | dev run | skipped | 0.0000 | dev run failed; going on was approved |

## What was built

| component | unit | kind | functions | cases | quality |
|---|---|---|---|---|---|
| trigger `sas_restructure_schedule` | `scheduled_run` | schedule | — | — |  |
| source `sas_incoming` | `list_incoming` | list | — | — |  |
| source `sas_incoming` | `skip_unusable` | select | — | — |  |
| source `sas_incoming` | `one_package_per_file` | split | — | — |  |
| destination `sas_restructure_destination` | `restructure_keys` | custom | folder_path, slash_format, underscore_format, matching_rules, ia_label, target_cdap, target_anon, target_cdar, decide_target | 44 | ok |
| destination `sas_restructure_destination` | `move_and_tag` | custom_delivery | tag_value_valid | 13 | ok |

## Assumptions

- analyst: The pipeline is named sas_restructure (from the request), not sas_folder_restructure (the document's name).
- analyst: Path patterns such as /adam/, /data_reuse/ and raw/ are checked against the folder path relative to incoming/ with a '/' added at its start and end, so a pattern also matches the first or last folder.
- analyst: All substring checks are case-sensitive except the 'ar' folder name and reporting level in CDAR (compared in lowercase); the ANON 'ia' check is case-sensitive as stated.
- analyst: The CDAR reporting level is written in the target as found in the path (original case) after the ia_ normalisation; only the comparison is done in lowercase.
- analyst: Underscore formats use the same precedence as slash formats: _adam_, then _sdtm_, then _raw_/_restricted_raw_.
- analyst: In CDAP and ANON interim paths, 'ia_ + the rest' means the part with its leading 'ia' removed, prefixed with 'ia_' (ia300 -> ia_300).
- analyst: A rule whose condition holds but whose target cannot be worked out (e.g. CDAR with no 'ar' folder) still counts as a match; the file goes to error/.
- analyst: Error targets keep the full path relative to incoming/, including the file name.
- analyst: The cron expression is kept as written; '*/45' in the minute field fires at minutes 0 and 45 of each hour, not strictly every 45 minutes.
- analyst: Report lines are written to the run log; no separate report file is stated.
- analyst: Moved files and error/ files are outside incoming/, so they are never picked up again.
- analyst: Synthetic study IDs (99-0001, S-001) and file names replace the document's sample identifiers in the examples.
- split: The source and destination bucket name is environment-specific and left as <REQUIRED>.
- split: Each file is its own package, so the package_name template is only a placeholder. The files are moved by the custom delivery, not packaged.
- split: Retry (2 times, 2 minutes apart) is described in the trigger requirement because the settings have no key for it.
- split: Anything the catalogue cannot express (routing rules, move, tagging, report) is built by the destination planner as a custom_delivery step.

## Deferred tests

- destination/sas_restructure_destination · move_and_tag · `overwrite_and_partial_failure`: deferred after it failed (Studio user, 2026-10-22T07:13:33+00:00)

## Costs

This run: 0.8714 USD. Recorded for the pipeline: 2.9569 USD of a 25.00 USD budget.
