# sas_folder_restructure_wf — pseudocode (display only)

1. Every 45 minutes (cron "*/45 * * * *", starting 2026-10-01), start one run. A failed run is retried up to 2 times, 2 minutes apart.
2. List every object under s3://{sas_bucket}/incoming/ at any depth. Take each path relative to incoming/.
3. Silently drop: keys ending in "/", paths with no folder (directly under incoming/), and file names without a ".".
4. Make each remaining file its own package. study = first folder of the path.
5. For each file: folder = path without the file name; last = last folder of folder.
6. Work out the data format: /adam/ → adam, else /sdtm/ → sdtm, else contains raw/ → raw, else none. Work out the underscore format: _adam_ → adam, else _sdtm_ → sdtm, else _raw_ or _restricted_raw_ → raw, else none.
7. Test the match conditions of all 7 rules: CDAP, IDAP, MED, ANON, CDAR, OTHERS, DEFAULT.
8. If exactly one rule matches, compute its target with that rule's sub-rules. If the computation fails (missing "_" parts, no "ar" folder, reporting level is adam/sdtm/raw), mark the file as an error with a reason.
9. If no rule or more than one rule matches, or step 8 failed: target = error/{path}, with a reason.
10. For each file: copy it server-side (multipart for files over 5 GiB) from incoming/{path} to the target, overwriting any existing object.
11. Confirm the target exists (with the same size). If the copy or the check fails: keep the source, mark the file as failed, and continue with the other files.
12. Delete incoming/{path}. If the delete fails: mark the file as failed.
13. For study targets only (not error/): replace the target's tags with {ADDITIONAL_TAG: path}. If tagging fails, log a warning only.
14. Report one line per file: source, target, rule or error reason, status.
15. Fail the run if any copy, check or delete failed. Otherwise succeed, even if files went to error/.
