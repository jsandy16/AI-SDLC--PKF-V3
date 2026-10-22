# sas_restructure — pseudocode (display only)

1. Every cron slot "*/45 * * * *" start one run (retries: 2, 2 minutes apart).
2. List every object under s3://{sas_bucket}/incoming/ at any depth; path = key without "incoming/".
3. Skip silently: keys ending in "/", paths with no folder (directly under incoming/), file names without ".".
4. Make one package per remaining file.
5. For each file: study = first folder; folder_path = path without file name; last_folder = last segment of folder_path.
6. slash_format = adam if "/adam/" in folder_path, else sdtm if "/sdtm/", else raw if "raw/", else none (folder_path checked with "/" around it).
7. underscore_format = adam if "_adam_", else sdtm if "_sdtm_", else raw if "_raw_" or "_restricted_raw_", else none.
8. Evaluate rules 1-7 (CDAP, IDAP, MED, ANON, CDAR, OTHERS, DEFAULT) on folder_path; collect every rule whose condition holds.
9. If exactly one rule holds and it computes a target -> target = that key with the file name kept; else target = error/{path} with reason (no rule / several rules / target cannot be worked out).
10. Server-side copy incoming/{path} -> target (multipart for > 5 GiB), overwriting any existing object.
11. Confirm the target exists; if the copy or check fails -> leave the source, mark file failed.
12. Delete incoming/{path}; if the delete fails -> mark file failed.
13. If the target is not under error/: replace all tags on the target with ADDITIONAL_TAG = {path}; a tagging failure is logged as a warning only.
14. Write one report line per file: source, target, rule or error reason.
15. Fail the run if any copy, check or delete failed; otherwise succeed (error/ moves do not fail it).
