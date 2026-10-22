# Walkthrough of the examples

## cdar_routes

1. Scheduled run starts (R-TRG-1); three files listed under incoming/, one package each (R-SRC-3, R-SRC-7).
2. No transformation (R-TRN-1).
3. a.sas7bdat: /cdar/ + /adam/, no /data_reuse/ or _anon_, so only CDAR matches (R-DST-6, R-DST-15); level csr -> d0000c00004/primary/complete/adam/a.sas7bdat (R-DST-17).
4. b.sas7bdat: level ia100 -> ia_100, format sdtm -> d0000c00004/primary/interim/sdtm/ia_100/b.sas7bdat (R-DST-18).
5. c.sas7bdat: level bdr200, format raw -> d0000c00004/primary/other/raw/bdr200/c.sas7bdat (R-DST-19).
6. Each is copied, checked, source deleted and tagged ADDITIONAL_TAG (R-AFT-1, R-AFT-4); run succeeds (R-RUN-3).

## cdap_routes

1. Scheduled run lists three files (R-TRG-1, R-SRC-3).
2. a: /data_reuse/ + /cdap_ + raw/ (R-DST-2, R-DST-7); OTHERS excluded by /cdap; /cdap_csr -> d0000c00004/extended_primary_use/primary/raw/a.sas7bdat (R-DST-8).
3. b: last folder cdap_20210617_ia300 contains _ia; third part ia300 -> ia_300 -> d0000c00004/extended_primary_use/interim/raw/ia_300/b.sas7bdat (R-DST-9).
4. c: format adam, last folder cdap_20210617 -> second part 20210617 -> d0000c00004/extended_primary_use/other/adam/20210617/c.sas7bdat (R-DST-10).
5. Moved and tagged (R-AFT-1, R-AFT-4).

## idap_med_others

1. Scheduled run lists three files (R-SRC-3).
2. a: /data_reuse/ + /idap + raw -> d0000c00004/internal_reuse/raw/idap_20200922/a.sas7bdat (R-DST-11).
3. adae: /data_reuse/ + /med + adam, last folder data -> 99-0001/external_reuse/adam/data/adae.sas7bdat (R-DST-12).
4. adsl: /data_reuse/ + adam and none of /med, /cdap, /idap, /anon, _anon_ -> 99-0001/other/adam/data/adsl.sas7bdat (R-DST-20).
5. Moved and tagged (R-AFT-1, R-AFT-4).

## anon_routes

1. Scheduled run lists three files (R-SRC-3).
2. All contain _anon_ and _raw_ -> ANON, format raw (R-DST-3, R-DST-13); DEFAULT excluded by _anon_ (R-DST-21).
3. a: _anon_csr -> d0000c00004/anonymised/primary/raw/a.sas7bdat (R-DST-13).
4. b: last part ia500 -> d0000c00004/anonymised/interim/raw/ia_500/b.sas7bdat (R-DST-14).
5. c: last part 20220120 -> d0000c00004/anonymised/other/raw/20220120/c.sas7bdat (R-DST-14).
6. Moved and tagged (R-AFT-1, R-AFT-4).

## default_errors_and_skips

1. Scheduled run lists six objects (R-SRC-3); incoming/readme.txt skipped (R-SRC-5) and no_extension skipped (R-SRC-6), with no report line.
2. Export_Inventory pdf: none of /data_reuse/, _anon_, /cdar/ -> DEFAULT -> ABCD-0000-0000/Export_Inventory_2024-03-01.pdf (R-DST-21).
3. shared/prod/data/adae: /data_reuse/ but no data format -> no rule -> error/... (R-DST-22).
4. data_reuse/cdar/.../ar/csr/adam: CDAR and OTHERS both match -> error/... (R-DST-6, R-DST-22).
5. root/cdar/.../adam with no 'ar' folder -> CDAR target cannot be worked out -> error/... (R-DST-16, R-DST-23).
6. Error files are moved without tag change (R-AFT-6); run succeeds (R-RUN-3).

## edge_cases

1. Scheduled run lists objects; the folder marker incoming/S-001/ is skipped (R-SRC-4).
2. x: /cdar/ + /adam/, folder 'AR' matches case-insensitively, level ia_psc -> ia_psc -> S-001/primary/interim/adam/ia_psc/x.sas7bdat (R-DST-15, R-DST-18).
3. y: _anon_ + _sdtm_, last part IA2 is not lowercase ia -> S-001/anonymised/other/sdtm/IA2/y.sas7bdat (R-DST-14).
4. z: CDAP, last folder cdap_ia2 contains _ia but has only 2 parts -> error/S-001/data_reuse/d000/S-001/shared/sdtm/dev/data/cdap_ia2/z.sas7bdat (R-DST-9, R-DST-22).
5. w: restricted_raw/ counts as raw; IDAP -> S-001/internal_reuse/raw/idap_x/w.sas7bdat (R-DST-2, R-DST-11).
6. Moves done; tags only on non-error targets (R-AFT-4, R-AFT-6).
