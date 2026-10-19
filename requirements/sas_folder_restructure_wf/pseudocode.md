# sas_folder_restructure_wf — pseudocode (display only)

1. Trigger on cron "*/45 * * * *"; no events.
2. List all objects under incoming/ in the SAS bucket (eu-north-1, set per environment), recursively.
3. Drop silently: keys ending in "/", files directly under incoming/ (no study folder), files whose name has no ".".
4. Make one package per remaining file; study = first folder after incoming/.
5. Compute the folder path (without file name) and the data format (adam, sdtm, raw; underscore forms for anonymised folders).
6. Evaluate rules 1-7 (CDAP, IDAP, MED, ANON, CDAR, OTHERS, DEFAULT) on the folder path.
7. If exactly one rule matches and yields a target, the target is that rule's key template. Otherwise (none, several, or an error such as too few "_" parts or a bad CDAR reporting level) the target is error/{path without incoming/}.
8. No transformation; file contents and names are untouched.
9. Deliver by custom move: server-side copy to the target (overwrite), check the target exists, delete the source from incoming/.
10. If the copy or check fails, keep the source in incoming/ and fail that file; the next run retries.
11. After a successful move to a non-error target, replace the target's tags with ADDITIONAL_TAG = path without incoming/. A tag failure is a warning only. Error/ targets get no tag change.
12. Report one line per file: source, target, rule or error reason.
13. The run fails only if a copy, check or delete failed.
