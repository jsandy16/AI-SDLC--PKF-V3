# Walkthrough of the examples

## cdar_routes

1. A scheduled run lists incoming/ (R-TRG-1, R-SRC-3). Each of the 3 files becomes its own package, with study d0000c00004 (R-SRC-9, R-SRC-10).
2. No transformation (R-TRN-1).
3. a.sas7bdat: the folder path contains /cdar/ and /adam/. Only CDAR matches, since there is no /data_reuse/ and DEFAULT is excluded by /cdar/. The level after 'ar' is csr → d0000c00004/primary/complete/adam/a.sas7bdat (R-DST-19, R-DST-22).
4. b.sas7bdat: format sdtm, level ia100 → ia_100 → d0000c00004/primary/interim/sdtm/ia_100/b.sas7bdat (R-DST-23).
5. c.sas7bdat: format raw, level bdr200 → d0000c00004/primary/other/raw/bdr200/c.sas7bdat (R-DST-24).
6. Each file is copied, checked and its source deleted. The tag ADDITIONAL_TAG=<original path> is set (R-RUN-1..3, R-RUN-6). The run succeeds.

## cdap_routes

1. A scheduled run lists 3 files, one package each (R-SRC-9).
2. a.sas7bdat: the path contains /data_reuse/, /cdap_ and /raw/, so only CDAP matches. It contains /cdap_csr → d0000c00004/extended_primary_use/primary/raw/a.sas7bdat (R-DST-7, R-DST-8).
3. b.sas7bdat: the last folder cdap_20210617_ia300 contains _ia. Its third part ia300 becomes ia_300 → d0000c00004/extended_primary_use/interim/raw/ia_300/b.sas7bdat (R-DST-9).
4. c.sas7bdat: format adam, last folder cdap_20210617, second part 20210617 → d0000c00004/extended_primary_use/other/adam/20210617/c.sas7bdat (R-DST-11).
5. Each file is moved and tagged (R-RUN-1..6).

## idap_med_others

1. A scheduled run lists 3 files, one package each.
2. a.sas7bdat: /data_reuse/ + /idap + raw, so IDAP matches. OTHERS is excluded by /idap → d0000c00004/internal_reuse/raw/idap_20200922/a.sas7bdat (R-DST-13).
3. adae.sas7bdat (study 04-3077): /data_reuse/ + /med + adam, so MED matches. The last folder is data → 04-3077/external_reuse/adam/data/adae.sas7bdat (R-DST-14).
4. adsl.sas7bdat: /data_reuse/ + adam with no /med, /cdap, /idap or anon, so OTHERS matches → 04-3077/other/adam/data/adsl.sas7bdat (R-DST-25).
5. Each file is moved and tagged (R-RUN-1..6).

## anon_routes

1. A scheduled run lists 3 files, one package each.
2. a.sas7bdat: the path contains _anon_ and _raw_, so ANON matches. DEFAULT is excluded by _anon_. It contains _anon_csr → d0000c00004/anonymised/primary/raw/a.sas7bdat (R-DST-15, R-DST-16).
3. b.sas7bdat: the last '_' part of the last folder is ia500, which starts with lowercase ia → d0000c00004/anonymised/interim/raw/ia_500/b.sas7bdat (R-DST-17).
4. c.sas7bdat: the last part is 20220120 → d0000c00004/anonymised/other/raw/20220120/c.sas7bdat (R-DST-18).
5. Each file is moved and tagged (R-RUN-1..6).

## default_errors_and_skips

1. A scheduled run lists 6 objects. incoming/readme.txt (no study folder) and no_extension (no '.') are skipped silently and not reported (R-SRC-6..8).
2. The Export_Inventory pdf: the folder path has no /data_reuse/, _anon_ or /cdar/, so DEFAULT applies → ABCD-0000-0000/Export_Inventory_usr00001_2024-05-29_08-39-32.pdf (R-DST-26). It is moved and tagged.
3. .../shared/prod/data/adae.sas7bdat: it has /data_reuse/ but no data format, so no rule matches → error/04-3077/root/data_reuse/d525/04-3077/shared/prod/data/adae.sas7bdat (R-DST-27). It is moved with no tag change (R-RUN-8).
4. .../data_reuse/cdar/.../ar/csr/adam/...: both CDAR and OTHERS match → error/04-3077/root/data_reuse/cdar/d168/04-3077/ar/csr/adam/prod/data/adae.sas7bdat (R-DST-6, R-DST-27).
5. .../root/cdar/d168/04-3077/adam/...: CDAR matches but there is no 'ar' folder → error/04-3077/root/cdar/d168/04-3077/adam/prod/data/adae.sas7bdat (R-DST-21, R-DST-27).
6. All copies, checks and deletes succeed, so the run succeeds even though files went to error/ (R-RUN-10, R-RUN-11).
