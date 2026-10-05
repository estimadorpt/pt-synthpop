# Portugal SynthPop Model Card

> Evidence paths under `data/products/` refer to the modelling repository
> (estimador-microsynthesis), where the audits and fit records are kept.

**Status:** V1.0.0, built and verified on 2026-10-05 and published on 2026-10-05.

## Model and data product

**Name:** População Sintética de Portugal / Portugal SynthPop  
**Publisher:** estimador.pt  
**Data vintage:** INE Censos 2021  
**Geography:** freguesia, with município fallback where parish publication is
not supported  
**Record levels:** households and persons  
**Release:** v1.0.0 (verified 2026-10-05; published 2026-10-05)

Portugal SynthPop generates household and person records whose aggregate
distributions follow published Census constraints. The records support
combinations that are unavailable in published small-area tables.

The release is synthetic data produced by estimador.pt. It is not official INE
microdata, and INE is not responsible for its contents.

## Intended uses

- descriptive demographic exploration;
- small-area household and population analysis within published quality rules;
- journalism and civic-data applications;
- education and reproducible research;
- aggregate scenario and poststratification work that propagates uncertainty;
- testing tools that require realistic but non-identifying population records.

## Uses that are not supported

- identifying, locating, or making decisions about real people;
- treating a synthetic row as an individual, family, or address;
- unrestricted cross-tabulation in tiny or weak-quality parishes;
- claims based on fields marked unvalidated;
- causal conclusions about policy effects;
- behavioral prediction or agent-based simulation without a separate validated
  model;
- replacing official Census statistics;
- legal, credit, insurance, employment, policing, or eligibility decisions.

## Training and calibration data

The model uses:

- the public-use Census 2021 household/person sample as its relationship seed;
- published INE Census 2021 aggregate tables as geographic constraints;
- geographic reference data for DICOFRE names and concordance.

The public-use seed is not redistributed with the release. Full reproduction
therefore requires obtaining it under its source terms.

## Method

The V1.0 pipeline is a constrained generative process (the A′ / v10 engine,
docs 107 and 139; the earlier VAE pipeline is retained only as a fallback):

1. an autoregressive generator over person-attribute tokens learns household/
   person relationships from the public-use sample, trained once and frozen
   (single-year ages and workplace attributes are generated, not derived);
2. for each freguesia a candidate pool is generated and tilted by maximum
   entropy onto the published Census tables, after a universe/mass check that
   refuses any population whose tables imply inconsistent totals;
3. an integer allocation draws the exact household and person counts;
4. the institutional population is appended from INE's published counts as
   partial records, and fine occupation/industry codes are derived (the
   occupation code is published; the industry code is held pending a decision,
   because no published table carries parish information below the section); and
5. release packaging ships exactly one declared public schema — every published
   column has a declared meaning and label map, everything else is dropped with
   a recorded reason (doc 204b). The release carries no couple link.

The final population must therefore be described as a **constrained generative
pipeline**, not as the untouched output of one neural network.

## Quality position — the v1.0.0 national (2026-10-05)

**The release gates pass.** These are the fail-closed decisions the release
depends on:

- coverage complete: 21 scored tables in all 3,092 parishes;
- structural violations 0;
- the child share, no worse than −10% in every region and size stratum, with the
  worst at +0.01%;
- the exact under-15 total in all 3,092 parishes, in both the private and the
  resident universe; and
- an integrity audit of every parish, with 0 errors.

**Fit to the published tables.** This is the median, per parish, of the
all-cell SRMSE over the 12 person tables, on the published resident population.
It is reported, not gated (doc 190).

| Parish size | Parishes | Median |
|---|---|---|
| fewer than 500 residents | 882 | 0.104 |
| 500–2,000 | 1,215 | 0.056 |
| 2,000–10,000 | 755 | 0.017 |
| more than 10,000 | 240 | 0.008 |

- **Municipalities:** the median over the 308 municipalities is 0.017, and 90% are
  below 0.038.
- **Regions:** the median by NUTS2 region ranges from 0.020 (Algarve, Lisbon and
  Tagus Valley) to 0.060 (Alentejo).
- **Quality tiers:** 776 parishes are tier A, 705 tier B and 1,611 tier C.
- **Against the previous engine's baseline national** (job 143), on the same
  parishes and the same private basis: the error falls by about three quarters
  in parishes under 500 residents (0.442 → 0.118), and in 3,058 of 3,092 parishes.

Release decisions taken earlier still apply (doc 156 §7, doc 158):

- a resident universe;
- exact under-15 totals as a fail-closed gate;
- **one replicate (R = 1) as a declared product downgrade:** no across-run variation
  layer and no rank claims;
- generated single-year ages; and
- a generated workplace type.

**Known limitations of v1.0.0, declared rather than hidden** (the plan to remove
them is the next version's, `docs/launch_readiness_20260714/205_next_version_plan.md`):

- **Workplace and commuting are the weakest attributes.** Work location, transport
  mode, industry and occupation fit the published tables far less well than the
  demographic and household tables do. They are not fitted to those tables by
  design: the model generates them, conditioned on the region.
- **Rare combinations.**
  - About 22,000 people sit in a cell that INE publishes as zero for their
    parish. Most are in the household-activity table, which is held out as an
    independent check. The rest, about 7,900 table entries (at most 0.08% of
    persons; one person can count in more than one table), sit in fitted person
    tables (labour × education, education, labour status, marital status),
    because those combinations never occur in the training sample, so the model
    has no cell for them.
  - About 47,000 small published cells, 87% of them holding a single person, are
    not reproduced.
- **Family structure.** Well under 1% of family units have a shape that real
  households in the sample do not have: for example a family unit without a
  member aged 15 or over, or a mother–child gap above 50 years.
- **Clock-bound optimisation.** In 116 large parishes (114 of the 238 above 10,000
  residents) the allocator's final search stopped at its time budget rather than
  at convergence. In one parish (050225) the exact selection stopped at its guard.
  These results depend on machine speed; each parish's record names the exit.
  Parish 110665 is the furthest from its band (median 0.023).
- **The appended institutional residents** are partial records by design.

## Small-area publication policy

Every parish is published as microdata. There is no size floor: the project
decided on 2026-09-25 that all 3,092 parishes ship, because this is a synthetic
population (doc 201 §9.4). Small parishes carry quality labels instead of being
withheld. Each has a `quality_tier` and a `publication_population` in the
quality table, and a parish under 500 residents can't reach tier A or B.

Public-query quality is query-specific:

- direct publication when the geography and requested variables pass;
- municipality fallback when the parish result is too weak;
- refusal when neither result is supported;
- cell suppression below the public minimum.

The machine-readable rules ship with the release as `public/response.schema.json`,
`public/bundle.schema.json` and `public/reason_glossary.json`.

## Variation and uncertainty

V1.0 publishes ONE synthesis run, as a declared product downgrade: the
per-number variation layer is out of v1.0, so no across-run range is published
and every directional or ranking claim between cells is refused by code rather
than discouraged in prose. A second run follows after publication as a
non-gating measurement; from two runs onward the spread across runs measures
model-run variation and the range returns.

It is not called a confidence interval unless empirical coverage is
demonstrated in the national evaluation. The public contract carries a
variation type and calibration status so interfaces cannot silently strengthen
the claim.

## Privacy position

The records are generated and are not intended to represent identifiable
people. They contain no names or addresses, and the release drops internal
donor pointers.

The current Aveiro study found:

- substantially less exact matching on the measured core attributes than the
  SA/CO replay benchmark;
- synthetic records no closer to the seed than real records are to other real
  records;
- no excess membership-inference signal in the proxy test; and
- no attribute-inference advantage beyond the population conditional measured
  by the real-data comparison.

Some generated records can still share coarse attribute combinations with seed
records because those combinations are common and statistically redundant.
Synthetic does not mean that accidental attribute matches are impossible.

The Aveiro evidence cannot be generalized silently to the national product,
so the national audit was run on the v1.0.0 population. It is stratified by
region and size and controlled by the SA/CO donor-replay benchmark, which the
same audit flags `not_ready`.

**Decision: pass.**

- **Exact matching** on the 13 core person attributes: 10.5% of synthetic persons
  and 0.8% of synthetic households have an exact counterpart in the seed. For the
  SA/CO replay, the figures are 99.1% and 99.9%.
- **Distance to the closest record:** synthetic persons are further from the seed
  than seed persons are from each other. 10.8% are exact matches, against 74.3%
  real-to-real.
- **Membership inference:** no excess signal (−0.002 AUC for persons, −0.007 for
  households).
- **Attribute inference:** no advantage over the real-data oracle (−0.002).

Fine occupation codes are compared at the seed's coarser granularity. An exact
match would otherwise be impossible by construction, and the audit would read
that impossibility as novelty.

On the 11 attributes the model generates, 80% of synthetic persons share an
attribute combination with some seed person. Those are common profiles; the
11-attribute distance is still larger than the seed's own.

Source: `data/products/validation/privacy_audit_p11_national_ht.json` and
`novelty_p11_national_ht.json`.

This model card is a technical disclosure, not legal advice.

## Provenance classes

Each public field is classified as:

- Census-calibrated;
- derived;
- modelled;
- carried forward; or
- unvalidated.

The release data dictionary records field meaning, code labels, universe,
source table, calibration status, and publication rule. Unvalidated seed-
correlated fields are withheld.

## Release contents

The planned V1.0 package contains:

- national and district-partitioned household/person parquet files;
- one quality row per freguesia, each carrying its quality label;
- code and label metadata;
- checksums and model hash;
- this model card, methodology, privacy note, and data dictionary;
- citation, attribution, errata, and source-revision policy.

**v1.0.0, as built on 2026-10-05.**

- **Contents:** 74 files plus `checksums.sha256`, 285 MB in all; 3,092 parishes
  published, 0 suppressed; 10,340,441 persons and 4,154,571 households.
- **Hashes:** the SHA-256 of `checksums.sha256` is `e85bc2da93cc0b745ca2b4aa4e81317241bf39c1d13402d7b2552130dac2c2f1`;
  the model is `062e2ad784886b7287536233f853db151c57615d2b1a952fb2e12368581e76d3`.
- **Code revision:** `5677c84`, packaged from a clean checkout. Run start, declared from the run's provenance
  marker: 2026-09-27T20:05:36+01:00.
- **Verification:** `verify-release --release-ready` passed on that checkout, again from a
  fresh clone, and once more immediately before upload (2026-10-05).
- **Provenance:** the code and scheduling changed in flight without changing the
  population configuration. These changes are recorded per phase in
  `data/products/validation/p11_national_ht_provenance_addendum.json`, with the
  identity controls that prove every earlier parish unchanged.

Release-ready packaging and independent verification are implemented in the
modelling repository. The final national command
will fail closed unless all 3,092 freguesias, evaluation results, model hash,
place names, public bundle, a zero size floor (every parish published), and
clean source revision are present.

## Public positioning

Recommended first-use wording:

> Tanto quanto nos foi possível apurar, a primeira população sintética de
> acesso aberto a cobrir todas as freguesias de Portugal, gerada a partir dos
> Censos 2021.

Do not shorten this to “the first synthetic population of Portugal.” The open,
national, and freguesia-level qualifiers are load-bearing.

The project may be described as a **foundation layer** for demographic
analysis. It is not described as a “foundation model” unless later adaptation
and weights-release evidence supports that term.

## Maintenance

- Stable releases remain citable.
- Corrections are recorded in a public errata log.
- INE source revisions trigger a documented patch or re-evaluation decision.
- V1.1 must identify which fields were updated to 2025, carried forward from
  2021, or newly modelled.
- Publishing weights requires a separate memorization, privacy, and licence
  review; output privacy does not clear the weights.

See the maintained [errata](ERRATA.md) and
[source revision policy](SOURCE_REVISION_POLICY.md). Every packaged release
contains pinned copies of both documents plus citation, attribution, licence,
disclosure, schemas, and checksums.
