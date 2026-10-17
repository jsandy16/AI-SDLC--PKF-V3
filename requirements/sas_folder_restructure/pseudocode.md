# sas_folder_restructure — pseudocode (display only)

1. Trigger: cron "*/45 * * * *" starts a run (no events).
2. Source: list all objects under incoming/ in the SAS bucket (eu-north-1), recursively; paths relative to incoming/.
3. Skip silently: keys ending in "/", files with no study folder (directly under incoming/), file names without ".".
4. Each remaining file becomes its own package; study = first folder.
5. Transformation: none; files are not opened or changed.
6. Destination: take the folder path (without file name); work out the data format (adam, then sdtm, then raw/).
7. Evaluate rules 1-7 (CDAP, IDAP, MED, ANON, CDAR, OTHERS, DEFAULT); anonymised folders take the format from _adam_/_sdtm_/_raw_/_restricted_raw_.
8. If exactly one rule matches and the target can be built, the key is the rule's target; otherwise the key is error/{path}.
9. Delivery (custom delivery, S3 to S3): server-side copy source to target, overwriting; verify the target exists; delete the source. On failure keep the source and fail that file.
10. For targets outside error/, replace all tags with ADDITIONAL_TAG = original path without incoming/; a failure is a warning only.
11. Report one line per file (source, target, rule or error reason). The run fails only on a copy, check or delete failure.
