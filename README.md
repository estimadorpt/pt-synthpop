# População Sintética de Portugal — v1.0.0

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
| Licence | [CC BY 4.0](LICENSE_DATA.md) |
| Published | 2026-10-05 |

Explore it by parish, read the methodology and the quality scorecard at
**<https://estimador.pt/pt/populacao/>**.

## Download

Files are attached to the [v1.0.0 release](https://github.com/estimadorpt/pt-synthpop/releases/tag/v1.0.0):

| File | What it is |
|---|---|
| `pt-synthpop-v1.0.0.zip` | The complete release package (≈ 182 MB): national and per-district parquet, the quality table, `metadata.json`, the public query bundle and schemas, and the trust files |
| `pt-synthpop-v1.0.0-persons.parquet` | All persons, one national file (≈ 78 MB) |
| `pt-synthpop-v1.0.0-households.parquet` | All households, one national file (≈ 7 MB) |
| `pt-synthpop-v1.0.0-quality.csv` | One row per parish: population, households, quality tier, fallback município. **Read this first.** |
| `pt-synthpop-v1.0.0-metadata.json` | Column dictionary, label maps and provenance |
| `checksums.sha256` | SHA-256 of every file inside the package |
| `SHA256SUMS` | SHA-256 of the release assets above |

```bash
sha256sum -c SHA256SUMS                     # the downloads
unzip pt-synthpop-v1.0.0.zip && cd pt-synthpop-v1.0.0 && sha256sum -c checksums.sha256
```

The package's `checksums.sha256` itself has SHA-256
`e85bc2da93cc0b745ca2b4aa4e81317241bf39c1d13402d7b2552130dac2c2f1`.

## Using it

Join persons to households on **`(freguesia, synthetic_hh_id)`**: `synthetic_hh_id`
is unique only within a parish.

```python
import duckdb
duckdb.sql("""
  select p.freguesia, count(*) as people
  from 'pt-synthpop-v1.0.0-persons.parquet' p
  join 'pt-synthpop-v1.0.0-households.parquet' h using (freguesia, synthetic_hh_id)
  where h.hh_size = 1 and p.age >= 65
  group by 1
""")
```

**Quality.** Every parish is published; small parishes carry labels instead of
being suppressed. Tier C parishes (1,611) should be read at the level of their
município (`fallback_geography` in the quality table). The labels, the fit by parish
size and the known limitations are in the [model card](MODEL_CARD.md).

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

> Fonte: Instituto Nacional de Estatística, IP – Portugal (Recenseamento Geral da População e Habitação — Censos 2021; período de referência: 2021). Informação modificada: os dados aqui publicados são uma população sintética gerada por estimador.pt a partir das distribuições marginais publicadas e do Ficheiro de Uso Público (FUP) dos Censos 2021, ao abrigo da licença CC BY 4.0; não constituem microdados oficiais do INE e o INE não é responsável pelo seu conteúdo.

Cite as: estimador.pt, População Sintética de Portugal v1.0.0 (2026), CC BY 4.0.
([CITATION.cff](CITATION.cff); `CITATION.package.cff` is the copy shipped inside the package.)

## Corrections

Stable releases are never replaced in place. Corrections are recorded in
[ERRATA.md](ERRATA.md) under the [source revision policy](SOURCE_REVISION_POLICY.md).
Report a problem by opening an issue in this repository or writing to
info@estimador.pt.
