# População Sintética de Portugal — v1.0.3

**Estas pessoas não são reais.** A synthetic population of every parish
(freguesia) in Portugal, calibrated to the published tables of **INE Censos 2021**
and published by [estimador.pt](https://estimador.pt/pt/populacao/).

> Tanto quanto nos foi possível apurar, a primeira população sintética de acesso
> aberto a cobrir todas as freguesias de Portugal, gerada a partir dos Censos 2021.

| | |
|---|---|
| Persons | 10,340,441 |
| Households | 4,154,571 (5,475 of them collective living quarters) |
| Parishes | 3,092, all published, with quality labels (A 776 · B 705 · C 1,611) |
| Census vintage | INE Censos 2021; geography DICOFRE / CAOP 2021 |
| Licence | [CC BY-NC 4.0](LICENSE_DATA.md) (Attribution-NonCommercial; INE's source data stay CC BY 4.0) |
| Published | 2026-10-05 |

Explore it by parish, read the methodology and the quality scorecard at
**<https://estimador.pt/pt/populacao/>**.

## Download

Files are attached to the [v1.0.3 release](https://github.com/estimadorpt/pt-synthpop/releases/tag/v1.0.3). v1.0.3 replaced the earlier releases of the same day (5 October 2026): v1.0.1 removed v1.0.0's launch thresholds so that every parish answers with its own numbers; v1.0.2 asked the household questions of private households and fixed the presentation; v1.0.3 corrected the `nuts2` column to NUTS-2013. The generated people and households have not changed since v1.0.0; only three município names and the `nuts2` labels did. See [ERRATA.md](ERRATA.md).

| File | What it is |
|---|---|
| `pt-synthpop-v1.0.3.zip` | The complete release package (≈ 182 MB): national and per-district parquet, the quality table, `metadata.json`, the public query bundle and schemas, and the trust files |
| `pt-synthpop-v1.0.3-persons.parquet` | All persons, one national file (≈ 78 MB) |
| `pt-synthpop-v1.0.3-households.parquet` | All households, one national file (≈ 7 MB) |
| `pt-synthpop-v1.0.3-quality.csv` | One row per parish: population, households, quality tier, NUTS-2013 region. **Read this first.** |
| `pt-synthpop-v1.0.3-metadata.json` | Column dictionary, label maps and provenance |
| `checksums.sha256` | SHA-256 of every file inside the package |
| `SHA256SUMS` | SHA-256 of the release assets above |

```bash
sha256sum -c SHA256SUMS                     # the downloads
unzip pt-synthpop-v1.0.3.zip && cd pt-synthpop-v1.0.3 && sha256sum -c checksums.sha256
```

The package's `checksums.sha256` itself has SHA-256
`0992bcfec9595df4fc2102349a6692e570ebc4322377e85438d795dc5a4ab8d8`.

## Using it

Join persons to households on **`(freguesia, synthetic_hh_id)`**: `synthetic_hh_id`
is unique only within a parish.

```python
import duckdb
duckdb.sql("""
  select p.freguesia, count(*) as people
  from 'pt-synthpop-v1.0.3-persons.parquet' p
  join 'pt-synthpop-v1.0.3-households.parquet' h using (freguesia, synthetic_hh_id)
  where h.hh_size = 1 and p.age >= 65
  group by 1
""")
```

**Quality.** Every parish is published with its own numbers; small parishes carry
quality labels instead of being withheld. Tier C (1,611 parishes: under 500 residents,
or a weaker fit to INE's tables) calls for more care; the quality table also names each
tier C parish's município (`fallback_geography`) for anyone who prefers to aggregate.
The labels, the fit by parish size and the known limitations are in the
[model card](MODEL_CARD.md).

**Use it for** descriptive and small-area analysis within the quality rules,
journalism, teaching, reproducible research, and testing tools that need realistic
but non-identifying records. **Not for** identifying, locating or deciding about real
people, treating a row as a real person, family or address, causal policy
conclusions, or behavioural prediction without a separately validated model. See
[DISCLOSURE.md](DISCLOSURE.md) and the model card.

**Privacy.** The national output-privacy audit passed. 10.5% of synthetic persons
and 0.8% of households match a record of the source sample exactly on 13 core
attributes; synthetic does not mean that accidental attribute matches are impossible.

## Credit

Required attribution ([ATTRIBUTION.txt](ATTRIBUTION.txt)):

> Fonte: Instituto Nacional de Estatística, IP – Portugal (Recenseamento Geral da População e Habitação — Censos 2021; período de referência: 2021). Informação modificada: os dados aqui publicados são uma população sintética gerada por estimador.pt a partir das distribuições marginais publicadas e do Ficheiro de Uso Público (FUP) dos Censos 2021, informação do INE reutilizada ao abrigo da licença CC BY 4.0; não constituem microdados oficiais do INE e o INE não é responsável pelo seu conteúdo. A população sintética é publicada por estimador.pt ao abrigo da licença CC BY-NC 4.0 (Atribuição-NãoComercial 4.0 Internacional).

Cite as: estimador.pt, População Sintética de Portugal v1.0.3 (2026), CC BY-NC 4.0.
([CITATION.cff](CITATION.cff), identical to the copy inside the package.)

## Corrections

Stable releases are never replaced in place. Corrections are recorded in
[ERRATA.md](ERRATA.md) under the [source revision policy](SOURCE_REVISION_POLICY.md).
The one exception, by the publisher's decision, is the licence: on 6 October 2026 every
published release (v1.0.0, v1.0.1, v1.0.3) was relabelled in place to CC BY-NC 4.0. Only
the licence statements, `ERRATA.md`, `metadata.json`, `checksums.sha256` and the zip
changed; the data are byte-identical. See the 2026-10-06 entry in [ERRATA.md](ERRATA.md).
Report a problem by opening an issue in this repository or writing to
info@estimador.pt.
