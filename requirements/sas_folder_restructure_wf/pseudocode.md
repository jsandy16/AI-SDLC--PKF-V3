# sas_folder_restructure_wf — pseudocode (display only)

1. Trigger: cron "*/45 * * * *" starts a run (retries 2, 2 minutes apart).
2. List all objects under incoming/ in the SAS bucket (eu-north-1), recursively.
3. Drop silently: keys ending in "/", files directly under incoming/ (no study folder), file names with no ".".
4. Make one package per remaining file; study = first folder after incoming/.
5. No transformation; files are not opened or changed.
6. For each file, take the folder path (without the file name) and work out the data format (/adam/, /sdtm/, raw/; for anonymised folders _adam_, _sdtm_, _raw_, _restricted_raw_).
7. Evaluate rules 1-7 (CDAP, IDAP, MED, ANON, CDAR, OTHERS, DEFAULT).
8. If exactly one rule matches and the target can be built, the target key = that rule's layout + file name. Otherwise the target = error/{path without incoming/}.
9. Server-side copy source to target (multipart-capable, overwriting any existing object).
10. Check that the target exists. If the copy or check fails, keep the source and mark the file failed.
11. Delete the source from incoming/. A delete failure marks the file failed.
12. For files not sent to error/, replace the target's tags with ADDITIONAL_TAG = original path without incoming/. A failure only logs a warning.
13. Report one line per file: source, target, rule or error reason.
14. The run fails only if any copy, check or delete failed.
