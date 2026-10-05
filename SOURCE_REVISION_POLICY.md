# Source-revision and geography policy

## Recorded vintages

- Census calibration vintage: `INE Censos 2021`
- Geography code vintage: `DICOFRE / CAOP 2021`
- Name-lookup vintage: recorded separately in `metadata.json`

## When a source change arrives

1. Record the changed INE table, geography file, publication date, and whether
   values or only labels/metadata changed.
2. Diff affected constraints and geographic mappings against the pinned release.
3. Re-run the smallest verification that covers the change; re-run national
   evaluation whenever calibrated values, universes, or boundaries change.
4. Publish the decision and link it from the errata log.

## Version decision

- Documentation, label, or attribution correction with unchanged numbers:
  patch release.
- Corrected source values or compatible new fields that change outputs:
  minor release plus re-evaluation.
- Census-vintage, geography-boundary, record-meaning, or incompatible schema
  change: major release.
- A privacy or material data-integrity problem may require immediate withdrawal
  before a replacement is available.

Stable releases remain available and citable unless withdrawal is required.
They are never replaced in place.
