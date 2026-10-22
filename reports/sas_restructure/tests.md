# sas_restructure: test cases to review

Each unit's test cases were written from the plan before any code. Keep the ones that matter; defer the others with `pkf defer <kind> <name> <step> <case> --reason …`, then `pkf review-tests sas_restructure --done` and `pkf rebuild sas_restructure --from C3`.

## destination/sas_restructure_destination · restructure_keys: Work out each file's target folder from its folder path using the CDAP/IDAP/MED/ANON/CDAR/OTHERS/DEFAULT rules, or error/ when no single rule gives a target

```
1. For each path in layout['files'] whose layout['keys'].get(path) is not set:
2.   key, rule, reason = decide_target(path)
3.   layout['keys'][path] = key
4.   log one line: source=incoming/<path>, target=key, rule=rule or error reason=reason
5. Return layout.
(Never sets None: no file is dropped. Only raises ValueError when even the error/ key is longer than 1,024 bytes.)
```

- **cdar_csr** (step case): CDAR with level csr goes to primary/complete
  covers R-DST-1, R-DST-2, R-DST-4, R-DST-15, R-DST-17, R-DST-6, R-RUN-2; input `{"files": ["S-001/root/cdar/d000/S-001/ar/csr/adam/dev/data/a.sas7bdat"], "vars": {"package": "S-001/root/cdar/d000/S-001/ar/csr/adam/dev/data/a.sas7bdat"}, "keys": {}}`; expected `["S-001/primary/complete/adam/a.sas7bdat"]`
- **cdar_without_ar_goes_to_error** (step case): CDAR matched but no ar folder gives error/ key
  covers R-DST-16, R-DST-22, R-DST-23; input `{"files": ["S-001/root/cdar/d000/S-001/adam/prod/data/b.sas7bdat"], "vars": {"package": "x"}, "keys": {}}`; expected `["error/S-001/root/cdar/d000/S-001/adam/prod/data/b.sas7bdat"]`
- **cdap_interim** (step case): CDAP interim last folder gives ia_300
  covers R-DST-7, R-DST-9, R-DST-5, R-DST-2; input `{"files": ["d0000c00004/data_reuse/d000/d0000c00004/ar/shared/raw/dev/data/cdap_20210617_ia300/b.sas7bdat"], "vars": {"package": "x"}, "keys": {}}`; expected `["d0000c00004/extended_primary_use/interim/raw/ia_300/b.sas7bdat"]`
- **anon_other** (step case): ANON other uses last underscore part and raw format
  covers R-DST-13, R-DST-14, R-DST-3; input `{"files": ["d0000c00004/d0000c00004_raw_anon_20220120/c.sas7bdat"], "vars": {"package": "x"}, "keys": {}}`; expected `["d0000c00004/anonymised/other/raw/20220120/c.sas7bdat"]`
- **default_passthrough** (step case): DEFAULT keeps the path
  covers R-DST-21, NAM-BAD-CHARS; input `{"files": ["ABCD-0000-0000/Export_Inventory_2024-03-01.pdf"], "vars": {"package": "x"}, "keys": {}}`; expected `["ABCD-0000-0000/Export_Inventory_2024-03-01.pdf"]`
- **several_rules_matched** (step case): CDAP and CDAR both match so error/
  covers REQ-3, R-DST-6, R-DST-22; input `{"files": ["S1/data_reuse/cdap_x/cdar/ar/csr/raw/f.txt"], "vars": {"package": "x"}, "keys": {}}`; expected `["error/S1/data_reuse/cdap_x/cdar/ar/csr/raw/f.txt"]`
- **anon_uppercase_ia_is_other** (step case): Uppercase IA500 is other, not interim
  covers REQ-8, REQ-7; input `{"files": ["S1/S1_raw_anon_IA500/f.txt"], "vars": {"package": "x"}, "keys": {}}`; expected `["S1/anonymised/other/raw/IA500/f.txt"]`
- **cdar_mixed_case_ar** (step case): Ar folder recognised; other level; no match for data_reuse without format
  covers REQ-9, REQ-2, REQ-1, R-DST-19; input `{"files": ["S1/cdar/Ar/bdr200/adam/f.txt", "S1/data_reuse/x/f.txt", "S1/f.txt"], "vars": {"package": "x"}, "keys": {}}`; expected `["S1/primary/other/adam/bdr200/f.txt", "error/S1/data_reuse/x/f.txt", "S1/f.txt"]`
- **fn:folder_path_nested** (function case of folder_path): Folder is wrapped in slashes
  covers R-DST-4; input `["a/b/c.txt"]`; expected `"/a/b/"`
- **fn:folder_path_root** (function case of folder_path): No folder gives a single slash
  covers R-DST-4; input `["c.txt"]`; expected `"/"`
- **fn:slash_format_adam_first** (function case of slash_format): adam wins over sdtm
  covers R-DST-2; input `["/x/sdtm/adam/"]`; expected `"adam"`
- **fn:slash_format_restricted_raw** (function case of slash_format): restricted_raw counts as raw
  covers R-DST-2; input `["/x/restricted_raw/"]`; expected `"raw"`
- **fn:slash_format_none** (function case of slash_format): No format gives None
  covers R-DST-2; input `["/x/y/"]`; expected `null`
- **fn:underscore_format_restricted_raw** (function case of underscore_format): restricted_raw is raw
  covers R-DST-3, REQ-7; input `["/a_restricted_raw_x/"]`; expected `"raw"`
- **fn:underscore_format_sdtm** (function case of underscore_format): sdtm detected
  covers R-DST-3; input `["/a_sdtm_x/"]`; expected `"sdtm"`
- **fn:matching_rules_two** (function case of matching_rules): Several matches listed in order
  covers R-DST-6, R-DST-7, R-DST-15; input `["/S1/data_reuse/cdap_x/cdar/ar/csr/raw/"]`; expected `["CDAP", "CDAR"]`
- **fn:matching_rules_default** (function case of matching_rules): Plain folder is DEFAULT
  covers R-DST-21; input `["/S1/"]`; expected `["DEFAULT"]`
- **fn:matching_rules_others** (function case of matching_rules): OTHERS matches
  covers R-DST-20; input `["/S1/data_reuse/foo/adam/"]`; expected `["OTHERS"]`
- **fn:matching_rules_idap_med** (function case of matching_rules): IDAP matches alone
  covers R-DST-11; input `["/S1/data_reuse/idap_x/raw/"]`; expected `["IDAP"]`
- **fn:ia_label_basic** (function case of ia_label): ia300 becomes ia_300
  covers R-DST-9; input `["ia300"]`; expected `"ia_300"`
- **fn:ia_label_underscore** (function case of ia_label): Leading underscore removed
  covers R-DST-18; input `["ia_psc"]`; expected `"ia_psc"`
- **fn:ia_label_upper** (function case of ia_label): Case-insensitive ia
  covers R-DST-18; input `["IA1"]`; expected `"ia_1"`
- **fn:target_cdap_csr** (function case of target_cdap): CDAP csr primary
  covers R-DST-8; input `["S", "/S/data_reuse/cdap_csr_x/", "cdap_csr_x", "adam", "f"]`; expected `"S/extended_primary_use/primary/adam/f"`
- **fn:target_cdap_other** (function case of target_cdap): CDAP other uses second part
  covers R-DST-10; input `["S", "/S/data_reuse/cdap_2021/", "cdap_2021", "raw", "f"]`; expected `"S/extended_primary_use/other/raw/2021/f"`
- **fn:target_cdap_ia_too_few** (function case of target_cdap): Interim with 2 parts raises
  covers R-DST-9, REQ-6; input `["S", "/S/data_reuse/cdap_ia/", "cdap_ia", "raw", "f"]`; expected `raises an error`
- **fn:target_cdap_other_too_few** (function case of target_cdap): Other with 1 part raises
  covers R-DST-10, REQ-6; input `["S", "/S/data_reuse/cdap/", "cdap", "raw", "f"]`; expected `raises an error`
- **fn:target_anon_csr** (function case of target_anon): ANON csr primary
  covers R-DST-13; input `["S", "/S/S_anon_csr/", "S_anon_csr", "adam", "f"]`; expected `"S/anonymised/primary/adam/f"`
- **fn:target_anon_interim** (function case of target_anon): lowercase ia is interim
  covers R-DST-14; input `["S", "/S/S_raw_anon_ia500/", "S_raw_anon_ia500", "raw", "f"]`; expected `"S/anonymised/interim/raw/ia_500/f"`
- **fn:target_anon_upper** (function case of target_anon): uppercase IA is other
  covers REQ-8; input `["S", "/S/S_raw_anon_IA500/", "S_raw_anon_IA500", "raw", "f"]`; expected `"S/anonymised/other/raw/IA500/f"`
- **fn:target_anon_empty_part** (function case of target_anon): Empty last part raises
  covers R-DST-14; input `["S", "/S/S_raw_anon_/", "S_raw_anon_", "raw", "f"]`; expected `raises an error`
- **fn:target_cdar_csr_upper_ar** (function case of target_cdar): AR and CSR case-insensitive
  covers R-DST-17, REQ-9; input `["S", "/S/cdar/AR/CSR/adam/", "adam", "f"]`; expected `"S/primary/complete/adam/f"`
- **fn:target_cdar_interim** (function case of target_cdar): ia_psc level
  covers R-DST-18; input `["S", "/S/cdar/ar/ia_psc/adam/", "adam", "f"]`; expected `"S/primary/interim/adam/ia_psc/f"`
- **fn:target_cdar_other** (function case of target_cdar): Other level kept as written
  covers R-DST-19; input `["S", "/S/cdar/ar/bdr200/sdtm/", "sdtm", "f"]`; expected `"S/primary/other/sdtm/bdr200/f"`
- **fn:target_cdar_no_level** (function case of target_cdar): ar last folder raises
  covers REQ-4, R-DST-16; input `["S", "/S/cdar/adam/ar/", "adam", "f"]`; expected `raises an error`
- **fn:target_cdar_format_level** (function case of target_cdar): Level adam any case raises
  covers REQ-5, R-DST-16; input `["S", "/S/cdar/ar/ADAM/", "adam", "f"]`; expected `raises an error`
- **fn:decide_no_rule** (function case of decide_target): No match goes to error
  covers REQ-2, R-DST-22; input `["S1/data_reuse/x/f.txt"]`; expected `["error/S1/data_reuse/x/f.txt", "", "no rule matched"]`
- **fn:decide_several** (function case of decide_target): Reason lists matched rules
  covers REQ-3; input `["S1/data_reuse/cdap_x/cdar/ar/csr/raw/f.txt"]`; expected `["error/S1/data_reuse/cdap_x/cdar/ar/csr/raw/f.txt", "", "several rules matched: CDAP, CDAR"]`
- **fn:decide_default** (function case of decide_target): Pass-through result
  covers REQ-1, R-DST-21; input `["S1/f.txt"]`; expected `["S1/f.txt", "DEFAULT", ""]`
- **fn:decide_cdar_error_reason** (function case of decide_target): Rule error reason prefixed with rule
  covers R-DST-23; input `["S-001/root/cdar/d000/S-001/adam/prod/data/b.sas7bdat"]`; expected `["error/S-001/root/cdar/d000/S-001/adam/prod/data/b.sas7bdat", "", "CDAR: CDAR: no ar folder"]`
- **fn:decide_idap** (function case of decide_target): IDAP uses last folder
  covers R-DST-11; input `["S1/data_reuse/idap_x/raw/f.txt"]`; expected `["S1/internal_reuse/raw/raw/f.txt", "IDAP", ""]`
- **fn:decide_med** (function case of decide_target): MED uses last folder
  covers R-DST-12; input `["S1/data_reuse/med_x/adam/f.txt"]`; expected `["S1/external_reuse/adam/adam/f.txt", "MED", ""]`
- **fn:decide_others** (function case of decide_target): OTHERS result
  covers R-DST-20; input `["S1/data_reuse/foo/adam/f.txt"]`; expected `["S1/other/adam/adam/f.txt", "OTHERS", ""]`
- **fn:decide_error_key_too_long** (function case of decide_target): Error key over 1024 bytes raises
  covers NAM-KEY-LENGTH; input `[{"$concat": ["S1/data_reuse/", {"$repeat": "a", "times": 1100}, "/f.txt"]}]`; expected `raises an error`
- **fn:decide_long_ok_key** (function case of decide_target): Key under limit passes
  covers NAM-KEY-LENGTH; input `[{"$concat": ["S1/", {"$repeat": "a", "times": 1000}, "/f.txt"]}]`; expected `[{"$concat": ["S1/", {"$repeat": "a", "times": 1000}, "/f.txt"]}, "DEFAULT", ""]`

## destination/sas_restructure_destination · move_and_tag: Move each file server-side to its target (copy, check, delete the source), then replace its tags with ADDITIONAL_TAG unless it went to error/

```
For each transfer t (one failing never stops the others):
1. key, rule, reason = decide_target(t['path']); to_error = (rule == '')
2. extra = {}; if t['etag']: extra['CopySourceIfMatch'] = t['etag']; d = ctx['delivery']; if d.get('server_side_encryption'): extra['ServerSideEncryption'] = it; if d.get('kms_key_id'): extra['SSEKMSKeyId'] = it
3. try aws.s3_destination.copy({'Bucket': t['source_bucket'], 'Key': t['source_key']}, t['target_bucket'], t['target_key'], ExtraArgs=extra, SourceClient=aws.s3_source) (managed multipart copy, works above 5 GiB; an existing target is overwritten). except Exception as e: result failed, detail 'copy failed: <e>'; source left; log; continue
4. try head = aws.s3_destination.head_object(Bucket=target_bucket, Key=target_key); if head['ContentLength'] != t['size']: raise. except: result failed, detail 'target check failed: <reason>'; source left; continue
5. try resp = aws.s3_source.delete_object(Bucket=source_bucket, Key=source_key). except Exception as e: result failed, detail 'copied but source delete failed: <e>'; continue. If resp has DeleteMarker/VersionId: add 'delete marker <VersionId>' to the detail
6. if not to_error: value = t['path']; if tag_value_valid(value): try aws.s3_destination.put_object_tagging(Bucket=target_bucket, Key=target_key, Tagging={'TagSet': [{'Key': 'ADDITIONAL_TAG', 'Value': value}]}) except Exception as e: aws.log.warning(...); detail += ' tagging failed (warning): <e>'. else: aws.log.warning('tag value not allowed'); detail += ' tag skipped (warning)'
7. status = 'moved_to_error' if to_error else 'moved'; detail = f'rule {rule}' or f'error: {reason}' plus any notes
8. aws.log.info one line: source=s3://source_bucket/source_key target=target_key status rule/reason
9. Append {'path': t['path'], 'key': t['target_key'], 'status': status, 'detail': detail}
Return the results (exactly one per transfer).
```

- **normal_move_and_tag** (step case): Rule-placed file is copied, source deleted, old tags replaced by ADDITIONAL_TAG=path
  covers R-AFT-1, R-AFT-4, R-RUN-2, R-RUN-3, DEL-TAGS, DEL-VERIFY; input `{"objects": {"bkt-test/incoming/S-001/root/cdar/d000/S-001/ar/csr/adam/dev/data/a.sas7bdat": {"body": "hello", "tags": {"old": "x"}}}, "transfers": [{"path": "S-001/root/cdar/d000/S-001/ar/csr/adam/dev/data/a.sas7bdat", "source_bucket": "bkt-test", "source_key": "incoming/S-001/root/cdar/d000/S-001/`; expected `{"objects": ["bkt-test/S-001/primary/complete/adam/a.sas7bdat"], "tags": {"bkt-test/S-001/primary/complete/adam/a.sas7bdat": {"ADDITIONAL_TAG": "S-001/root/cdar/d000/S-001/ar/csr/adam/dev/data/a.sas7bdat"}}, "statuses": {"S-001/root/cdar/d000/S-001/ar/csr/adam/dev/data/a.sas7bdat": "moved"}}`
- **error_target_no_tag_change** (step case): File sent to error/ is moved, status moved_to_error, tags left as copied
  covers R-AFT-6, R-RUN-3, R-RUN-2, REQ-12: file moved to error/ -> status moved_to_error, no tagging call, does not fail the run; input `{"objects": {"bkt-test/incoming/S-001/root/cdar/d000/S-001/adam/prod/data/b.sas7bdat": {"body": "abc", "tags": {"k": "v"}}}, "transfers": [{"path": "S-001/root/cdar/d000/S-001/adam/prod/data/b.sas7bdat", "source_bucket": "bkt-test", "source_key": "incoming/S-001/root/cdar/d000/S-001/adam/prod/data/b`; expected `{"objects": ["bkt-test/error/S-001/root/cdar/d000/S-001/adam/prod/data/b.sas7bdat"], "tags": {"bkt-test/error/S-001/root/cdar/d000/S-001/adam/prod/data/b.sas7bdat": {"k": "v"}}, "statuses": {"S-001/root/cdar/d000/S-001/adam/prod/data/b.sas7bdat": "moved_to_error"}}`
- **copy_fails_source_left** (step case): Copy failure (e.g. precondition) gives failed and leaves source, no target
  covers R-AFT-2, R-RUN-3, REQ-10: copy fails with PreconditionFailed (source changed since listing) -> failed, source left, DEL-LARGE-COPY; input `{"objects": {"bkt-test/incoming/ABCD-0000-0000/f.pdf": "data"}, "transfers": [{"path": "ABCD-0000-0000/f.pdf", "source_bucket": "bkt-test", "source_key": "incoming/ABCD-0000-0000/f.pdf", "size": 4, "etag": null, "target_bucket": "bkt-test", "target_key": "ABCD-0000-0000/f.pdf"}], "fail_calls": [{"op`; expected `{"objects": ["bkt-test/incoming/ABCD-0000-0000/f.pdf"], "tags": {}, "statuses": {"ABCD-0000-0000/f.pdf": "failed"}}`
- **head_check_fails_source_left** (step case): Target check failure gives failed and the source is not deleted
  covers R-AFT-2, DEL-VERIFY, R-RUN-3; input `{"objects": {"bkt-test/incoming/ABCD-0000-0000/f.pdf": "data"}, "transfers": [{"path": "ABCD-0000-0000/f.pdf", "source_bucket": "bkt-test", "source_key": "incoming/ABCD-0000-0000/f.pdf", "size": 4, "etag": null, "target_bucket": "bkt-test", "target_key": "ABCD-0000-0000/f.pdf"}], "fail_calls": [{"op`; expected `{"objects": ["bkt-test/incoming/ABCD-0000-0000/f.pdf", "bkt-test/ABCD-0000-0000/f.pdf"], "tags": {}, "statuses": {"ABCD-0000-0000/f.pdf": "failed"}}`
- **delete_fails_run_fails** (step case): Delete failure after good copy gives failed; target stays
  covers R-AFT-1, R-RUN-3, REQ-11: delete fails after a good copy -> failed (run fails), target stays and the retry overwrites it; input `{"objects": {"bkt-test/incoming/ABCD-0000-0000/f.pdf": "data"}, "transfers": [{"path": "ABCD-0000-0000/f.pdf", "source_bucket": "bkt-test", "source_key": "incoming/ABCD-0000-0000/f.pdf", "size": 4, "etag": null, "target_bucket": "bkt-test", "target_key": "ABCD-0000-0000/f.pdf"}], "fail_calls": [{"op`; expected `{"objects": ["bkt-test/incoming/ABCD-0000-0000/f.pdf", "bkt-test/ABCD-0000-0000/f.pdf"], "tags": {}, "statuses": {"ABCD-0000-0000/f.pdf": "failed"}}`
- **tagging_failure_is_warning** (step case): Tagging failure still counts as moved
  covers R-AFT-5, DEL-TAGS, R-RUN-3; input `{"objects": {"bkt-test/incoming/ABCD-0000-0000/f.pdf": "data"}, "transfers": [{"path": "ABCD-0000-0000/f.pdf", "source_bucket": "bkt-test", "source_key": "incoming/ABCD-0000-0000/f.pdf", "size": 4, "etag": null, "target_bucket": "bkt-test", "target_key": "ABCD-0000-0000/f.pdf"}], "fail_calls": [{"op`; expected `{"objects": ["bkt-test/ABCD-0000-0000/f.pdf"], "statuses": {"ABCD-0000-0000/f.pdf": "moved"}, "tags": {}}`
- **overwrite_and_partial_failure** (step case): Existing target overwritten; one file failing does not stop the other
  covers R-AFT-3, DEL-OVERWRITE, DEL-PARTIAL, R-RUN-3, R-RUN-1, DEL-IDEMPOTENT, DEL-SSE, DEL-VERSIONED; input `{"objects": {"bkt-test/incoming/ST1/a.txt": "new", "bkt-test/ST1/a.txt": "older-body", "bkt-test/incoming/ST2/b.txt": "bb"}, "transfers": [{"path": "ST2/b.txt", "source_bucket": "bkt-test", "source_key": "incoming/ST2/b.txt", "size": 2, "etag": null, "target_bucket": "bkt-test", "target_key": "ST2/b`; expected `{"objects": ["bkt-test/incoming/ST2/b.txt", "bkt-test/ST1/a.txt"], "tags": {"bkt-test/ST1/a.txt": {"ADDITIONAL_TAG": "ST1/a.txt"}}, "statuses": {"ST2/b.txt": "failed", "ST1/a.txt": "moved"}}`
- **fn:valid_simple** (function case of tag_value_valid): Normal path with allowed characters is valid.
  covers DEL-TAGS; input `["S-001/root/a.sas7bdat"]`; expected `true`
- **fn:empty_valid** (function case of tag_value_valid): Empty value is allowed.
  covers DEL-TAGS; input `[""]`; expected `true`
- **fn:len_256_valid** (function case of tag_value_valid): Exactly 256 characters is allowed.
  covers DEL-TAGS; input `[{"$repeat": "a", "times": 256}]`; expected `true`
- **fn:len_257_invalid** (function case of tag_value_valid): 257 characters is too long.
  covers DEL-TAGS; input `[{"$repeat": "a", "times": 257}]`; expected `false`
- **fn:bad_char_invalid** (function case of tag_value_valid): A disallowed character makes it invalid.
  covers DEL-TAGS; input `["a#b"]`; expected `false`
- **fn:allowed_specials_valid** (function case of tag_value_valid): All allowed special characters are accepted.
  covers DEL-TAGS; input `["a b+c-d=e.f:g/h@i_j"]`; expected `true`
