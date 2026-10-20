# entimice_api_pipeline — pseudocode (display only)

1. Trigger: cron "*/45 * * * *"; one event per run; retries 2, delay 2 min.
2. List all objects under incoming/ in the SAS bucket (eu-north-1), recursively; paths relative to incoming/.
3. Skip silently: keys ending "/", files with no study folder (directly under incoming/), file names without ".".
4. Make one package per remaining file; study = first folder.
5. No transformation; files are not opened or changed.
6. For each file: take the folder path (without file name) and work out the data format (adam > sdtm > raw/).
7. Evaluate rules 1-7 (CDAP, IDAP, MED, ANON, CDAR, OTHERS, DEFAULT) on the folder path.
8. If exactly one rule matches and gives a target, the target key is that rule's template; otherwise the target is error/{path without incoming/}.
9. Delivery (custom delivery, last step): server-side copy source to target, overwriting; confirm the target exists; delete the source.
10. If copy or check fails: keep the source, mark the file failed (retried next run).
11. After a successful move to a non-error target: replace all tags with ADDITIONAL_TAG = original path without incoming/; a failure only logs a warning.
12. Files moved to error/ get no tag change.
13. Report one line per file (source, target, rule or error reason). Fail the run only if a copy, check or delete failed.
