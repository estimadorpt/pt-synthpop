# Public errata — População Sintética de Portugal v1.0.3

**Status at packaging:** the entries below, newest last.

### 2026-10-05 — public answers hid parish numbers (display; resolved in v1.0.1)

- **Affected:** the v1.0.0 public bundle (`public_bundle_v1.0.0.json`); the microdata are not affected.
- **What was wrong:** the public answers applied launch thresholds that predate the decision to
  publish every parish: cells under 10 read "Suprimido", the 1,611 tier-C parishes were answered
  with their município's numbers, 2,316 answers were refused, a category with no one in it was
  missing instead of 0, and the daily game held the tier-C parishes out.
- **Resolution:** v1.0.1 answers every question for every parish with its own numbers and its
  quality tier; zero categories appear as 0; the game deck holds all 3,092 parishes.

### 2026-10-05 — `nuts2` is a district grouping, not NUTS-2013 (data; resolved in v1.0.3)

- **Affected:** the household column `nuts2` and the quality table's `nuts2`, in v1.0.0 to v1.0.2;
  417 of 3,092 parishes (1.12 million residents, 10.8%).
- **What is wrong:** the column is derived from the parish's district, not from INE's NUTS-2013
  regions. The Aveiro-district part of the Porto metropolitan area, Tâmega e Sousa and Douro are
  coded Centro (16) instead of Norte (11); Oeste and Médio Tejo are coded Lisboa (17) instead of
  Centro (16); Lezíria do Tejo and Alentejo Litoral are coded Lisboa (17) instead of Alentejo (18).
  Totalled by `nuts2`, Lisboa reads +24.6%, Alentejo −43.0% and Norte −11.6% of residents.
- **What it does not affect:** generation, fitting and every quality figure use the parish's
  NUTS-2013 NUTS3 region, which is correct for all 3,092 parishes.
- **Until corrected:** do not aggregate by `nuts2`. Derive the region from `freguesia` with
  INE's parish → NUTS-2013 table; a NUTS3 code's first two characters are its NUTS2.
- **Resolution (v1.0.3):** `nuts2` is the parish's NUTS-2013 NUTS2, derived at packaging
  from the parish code (`metadata.json` → `nuts2_derivation`). It changes on 417 parishes and
  nothing else in the microdata does.

### 2026-10-05 — public-layer presentation (display/metadata; resolved in v1.0.2)

- **Affected:** the v1.0.0 and v1.0.1 public bundles and the package's município names.
- **What was wrong:** percentages used a dot decimal under `pt-PT`; age bands sorted as text
  ("10 - 14" before "5 - 9"); portrait titles dropped the population a question is about;
  household questions counted the institutional living quarters as households, and "who lives
  alone" counted care-home residents as living with others; the game's similarity read a missing
  feature as 0; município names came from CAOP 2024.1 (two indistinguishable "Calheta").
- **Resolution:** v1.0.2 writes `pt-PT` decimals, sorts bands by their numbers, titles each
  question with its population, asks household questions of private households (INE's household
  universe), reads a missing game feature as the average, and names municípios from INE's
  Censos 2021 geography ("Calheta (R.A.M.)", "Calheta (R.A.A.)", "Lagoa (R.A.A.)"). No number
  changed: in the microdata only the `municipio_name` of those three municípios differs.

### 2026-10-06 — the synthetic population is licensed CC BY-NC 4.0 (metadata; relabelled in place)

- **Affected:** the licence statements of every published release (v1.0.0, v1.0.1 and v1.0.3)
  and of every later one: `LICENSE_DATA.md`, `ATTRIBUTION.txt`, `README.md`, `CITATION.cff` and
  `metadata.json` (`license`, `ine_attribution`).
- **What changed:** from 6 October 2026 the synthetic population is licensed under Creative
  Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0,
  https://creativecommons.org/licenses/by-nc/4.0/). INE's source data (Censos 2021 published
  tables and Public Use File) remain INE's, reused under CC BY 4.0, and the credit to INE is
  unchanged.
- **What did not change:** the data. The persons, households and quality files and the public
  bundle and schemas are byte-identical; their sha256 in `checksums.sha256` are unchanged.
- **Replacement:** none. By the publisher's decision each release is relabelled in place under
  its own version number, rather than as the patch release `SOURCE_REVISION_POLICY.md` calls
  for: its licence statements, this file, `metadata.json`, `checksums.sha256` and the zip
  change, and the release's `SHA256SUMS` lists their new sha256.

## How corrections are recorded

Corrections are never applied silently to a stable release. Each entry records:

- date discovered and date resolved;
- affected release, geography, files, fields, and public queries;
- severity: presentation, metadata, data, privacy, or withdrawal;
- user-visible effect and corrected interpretation;
- replacement version and checksums, when applicable; and
- whether downstream stories, scorecards, or applications require regeneration.

Report a suspected problem through the repository issue tracker with the
release version, stable query URL or file path, and enough detail to reproduce
it. Security or privacy concerns should be reported privately to the publisher
before public disclosure.
