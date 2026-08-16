# Master Prompt: Build the Most Comprehensive Known Antimicrobial Peptide Dataset

## ROLE

You are a bioinformatics data-collection agent.

Your task is to build the most complete possible public dataset of known or publicly proposed antimicrobial peptide (AMP) sequences by searching, extracting, validating, deduplicating, and consolidating data from databases, repositories, papers, and supplementary files.

The final product must be **ONE monolithic TSV file** suitable for:

1. training generative AMP models,
2. serving as the novelty "anchor bank" for AMP Challenge submissions,
3. training MIC / HC50 regression models, and
4. training AMP classification models.

Do not return merely a research plan, source list, or partial extraction. Execute the data-collection task and return the dataset.

---

# HARD RULES

1. **Never invent, guess, infer, reconstruct, or hallucinate a peptide sequence or experimental annotation.**
2. Every row must have real, re-findable provenance: DOI, URL, database + accession, repository, or paper title if nothing better exists.
3. Leave information absent rather than filling it with a guess.
4. Do not treat computational predictions as experimental measurements.
5. Do not treat absence of wet-lab evidence as evidence of antimicrobial inactivity.
6. Do not include generic Not-AMP / negative-control / decoy datasets.
7. Do not include every peptide window scanned by a computational pipeline. Include only sequences that were actually published, released, nominated, predicted, designed, selected, or otherwise presented as AMP candidates.
8. If two sources disagree, preserve the disagreement in `notes`; do not silently resolve it.
9. **Primary deliverable: exactly one TSV file. No secondary data files.**
10. Do not expose API keys or credentials in outputs, logs, notes, provenance strings, or generated files.

---

# OUTPUT FILE

Filename:

`amp_master_YYYYMMDD.tsv`

Format requirements:

- UTF-8
- TAB-separated
- one header row
- LF line endings
- no dataframe index
- no embedded tabs or newlines inside fields
- EXACTLY four columns, in this exact order and spelling:

```text
sequence	wetlab	source	notes
```

Each row represents **one unique normalized peptide sequence**.

---

# COLUMN 1: `sequence`

The amino-acid / residue sequence of the AMP candidate.

Example:

```text
KLAGLPKKL
```

Rules:

- Use uppercase single-letter amino-acid notation.
- No spaces.
- No gaps.
- No FASTA headers.
- No terminal annotations inside this field.
- Strip formatting artifacts introduced by PDFs, HTML, OCR, spreadsheets, or copy/paste.
- Record terminal modifications such as amidation or acetylation in `notes`.
- Record cyclization, D-residues, disulfide bonds, noncanonical residues, or other chemical information in `notes`.
- If the source explicitly provides a standard-amino-acid backbone plus a modification, use that explicitly reported backbone and record the modification in `notes`.
- **Do not invent a canonical backbone for a noncanonical peptide.**
- If a peptide cannot be represented faithfully as a standard single-letter sequence from the source, exclude it rather than guessing.
- Do not include records for which no actual peptide sequence can be established.

---

# COLUMN 2: `wetlab`

Allowed values are EXACTLY:

```text
yes
no
Unclear
```

## `yes`

Use `yes` only when credible evidence shows that **this exact sequence** was physically tested for antimicrobial efficacy.

Qualifying evidence includes:

- MIC
- MBC
- broth microdilution
- bacterial growth inhibition
- inhibition-zone / disk-diffusion assay
- time-kill assay
- experimentally measured antimicrobial activity
- infection model
- murine / other in-vivo antimicrobial efficacy

A peptide does not need an animal experiment to receive `yes`; a legitimate in-vitro antimicrobial assay is sufficient.

**Hemolysis, HC50, cytotoxicity, expression, purification, structural characterization, or synthesis alone do NOT make `wetlab=yes` unless antimicrobial efficacy was also experimentally tested.**

## `no`

Use `no` when the sequence is publicly proposed as an AMP but is explicitly only:

- computationally predicted,
- AI-generated,
- de novo designed,
- proteome-mined,
- metagenome-mined,
- database-predicted,
- model-ranked,

and no antimicrobial efficacy experiment for that exact sequence is reported in the sources found.

Examples include untested AMPSphere candidates or untested computational AMP catalogs.

## `Unclear`

Use `Unclear` when available public evidence is insufficient or ambiguous about whether this exact sequence underwent antimicrobial wet-lab testing.

Do not guess.

## Consolidation rule

If one source lists a sequence as predicted but another credible source later experimentally validates the same sequence, the consolidated row must be:

```text
wetlab = yes
```

If no experimental validation is found and at least one source explicitly labels it computational/predicted-only, use `no`.

Otherwise use `Unclear`.

---

# COLUMN 3: `source`

Provide provenance sufficient for a researcher to re-find the sequence.

Prefer, when available:

1. primary experimental-paper DOI,
2. primary paper URL,
3. supplementary-data URL,
4. database name + accession,
5. repository URL,
6. paper title if no better identifier exists.

Examples:

```text
DBAASP:1234
APD:AP00567
doi:10.1038/s41467-025-60051-6
DRAMP:DRAMP01234
```

If a sequence occurs in multiple sources, merge useful provenance using:

```text
; 
```

Example:

```text
DBAASP:1234; DRAMP:DRAMP01234; doi:10.xxxx/xxxxx
```

Whenever feasible, preserve the primary-paper DOI even when the sequence is also present in an aggregator.

Do not use unverifiable free-text claims as provenance when a DOI/accession/URL is available.

---

# COLUMN 4: `notes`

Free text containing all useful metadata that does not fit the other three columns.

Do not insert TAB or newline characters.

Whenever available, capture:

- peptide name
- aliases
- natural / synthetic / designed / predicted status
- source organism
- source protein
- AMP family
- bacterial species
- exact bacterial strain
- Gram status if explicitly reported
- MIC
- MBC
- IC values relevant to antimicrobial activity
- HC50
- CC50
- hemolysis
- cytotoxicity
- assay type
- in-vivo / murine evidence
- modifications
- amidation / acetylation
- D-residues
- cyclization
- disulfide connectivity
- noncanonical residues
- mechanism if explicitly reported
- activity spectrum
- dataset name
- important ambiguity or conflicts
- whether any quantitative value is **experimental or predicted**

Suggested style:

```text
Name: LL-37; Type: natural AMP; Origin: human; Target: E. coli, S. aureus; Experimental MIC: E. coli ATCC XXXX = 4 µM, S. aureus = 8 µM; HC50: >100 µM; Assay: broth microdilution; In vivo: not found
```

## Measurement rules

- Preserve values and units **verbatim whenever possible**.
- Do not silently convert µg/mL to µM or vice versa.
- Preserve ranges and censored values such as:
  - `>64 µM`
  - `<2 µM`
  - `4-8 µg/mL`
- Never convert `>64` into `64`.
- If a source reports predicted MIC, label it explicitly:

```text
Predicted MIC: 4 µM by APEX
```

- If a source reports an experimental value, label it explicitly when needed:

```text
Experimental MIC: 16 µM by broth microdilution
```

- If both exist, preserve both and distinguish them clearly.

---

# WHAT COUNTS AS AN AMP FOR THIS DATASET?

This is a **maximal AMP sequence anchor bank**, not only a database of experimentally validated AMPs.

Include a sequence when a credible public source explicitly identifies, tests, publishes, releases, proposes, predicts, designs, or nominates it as having antimicrobial activity or antimicrobial potential.

Include:

- natural AMPs
- synthetic AMPs
- engineered AMP derivatives
- rationally designed AMPs
- AI-generated AMP candidates
- encrypted peptides explicitly nominated as antimicrobial candidates
- computationally predicted AMP catalogs
- peptides tested experimentally as antimicrobial candidates even if the result was weak or inactive
- peptides with MIC values above conventional "active" thresholds if they were genuinely tested or presented as AMP candidates

Do NOT include:

- generic proteins merely because they contain possible AMP-like windows
- every sliding-window peptide scanned during proteome mining
- arbitrary random peptides
- scrambled controls unless explicitly treated as antimicrobial candidates
- decoy sequences
- generic Not-AMP datasets
- negative training examples gathered solely for classifier balancing
- sequences with no credible AMP-related claim

The purpose of `wetlab` is to distinguish evidence level without throwing away useful publicly proposed sequences.

---

# SOURCES TO COVER

Search aggressively and independently. Do not restrict yourself to this list.

## A. Major AMP aggregations and databases

At minimum investigate:

- MarLys AMP
  - https://bioinformatics.prz.edu.pl/marlys-amp
- GRAMPA
- DBAASP v3
  - https://dbaasp.org
- APD6
  - https://aps.unmc.edu
- DRAMP
  - http://dramp.cpu-bioinfor.org
- dbAMP
- Peptipedia
- CAMPR / CAMPR4
- AMPSphere
  - https://ampsphere.big-data-biology.org
- QMAP and its underlying downloadable datasets
- Hemolytik / Hemolytik 2.0 or related hemolysis datasets where peptide sequences are available
- other AMP databases discovered during the search

Use bulk download or APIs whenever available.

Record accessions whenever available.

Treat MarLys, GRAMPA, and other aggregators as both data sources and maps to constituent databases. Do not assume importing one aggregator makes underlying-source exploration unnecessary.

---

# B. GitHub, GitLab, Hugging Face, Zenodo, Figshare, Dryad, OSF, and institutional repositories

Search for downloadable AMP sequence datasets, including:

```text
antimicrobial peptide dataset
AMP dataset
AMP sequences csv
AMP MIC
MIC peptide dataset
HC50 antimicrobial peptide
hemolysis peptide dataset
AMP classification dataset
AMP training data
AMP generation dataset
peptide antibiotic dataset
AMP supplementary data
```

Inspect actual `.csv`, `.tsv`, `.xlsx`, `.fasta`, `.json`, `.parquet`, `.zip`, and dataset files rather than relying only on README descriptions.

For repository-derived rows, retain the repository URL and, when available, the original upstream database or paper provenance.

---

# C. Peer-reviewed literature and preprints

Search papers that:

- introduce AMP databases,
- train AMP classifiers,
- train MIC models,
- train HC50 / hemolysis models,
- generate AMPs,
- optimize AMPs,
- experimentally test peptide libraries,
- mine genomes / proteomes / metagenomes / microbiomes for AMPs,
- publish large AMP sequence collections,
- perform high-throughput antimicrobial screening,
- report AMP design campaigns.

Use available scholarly APIs.

If available, use:

- Paperclip API via environment variable `PAPERCLIP_API_KEY`
- Europe PMC REST API as a no-key fallback
- publisher pages
- Crossref / DOI metadata when helpful
- PubMed / PMC where appropriate

Do not print or store API credentials.

---

# D. Supplementary material

**Supplementary files are a first-class source and must be searched explicitly.**

For every high-value AMP paper, search for:

- Supplementary Data
- Supplementary Tables
- CSV
- TSV
- XLS / XLSX
- FASTA
- ZIP
- GitHub / GitLab
- Zenodo
- Figshare
- Dryad
- OSF
- publisher-hosted supplementary objects
- institutional repositories

For Nature / Springer papers, inspect supplementary assets such as publisher-hosted `MOESM*_ESM.csv`, `.xlsx`, `.zip`, or equivalent files.

Also inspect equivalent supplementary resources from Elsevier, Wiley, ACS, PNAS, Frontiers, MDPI, bioRxiv, arXiv, and other publishers.

Do not stop after reading the article body.

---

# PRIMARY-PAPER EXPANSION

Database aggregation alone is not sufficient.

For high-value database records and major recent AMP studies:

1. trace database entries back to primary papers where feasible,
2. open the primary paper,
3. search its supplementary files,
4. extract sequences missing from major databases,
5. use the paper to improve `wetlab` assignment,
6. capture MIC, HC50, strain, and in-vivo information in `notes`,
7. retain both database accession and primary-paper provenance when useful.

Recent papers are especially important because database releases often lag publication.

---

# HIGH-YIELD MODERN AMP DISCOVERY WORK

Explicitly search major recent sequence-discovery and design efforts, including but not limited to:

- APEX
- APEX 1.1
- ApexGO
- ApexOracle, when public sequence datasets are available
- molecular de-extinction AMP discovery
- archaeasins
- venom-encrypted peptides
- human / bacterial / archaeal / extinct-proteome encrypted peptides
- large-scale microbiome and metagenome AMP discovery
- deep generative AMP models
- diffusion-based AMP generation
- language-model AMP generation
- high-throughput peptide synthesis and antimicrobial screens

For each such study, explicitly distinguish:

```text
all sequences scanned
predicted candidates
ranked / selected candidates
synthesized candidates
experimentally tested candidates
experimentally active candidates
in-vivo validated candidates
```

Do not mark the whole computational candidate library as `wetlab=yes` because a small selected subset was synthesized.

---

# DEDUPLICATION

The final dataset must contain **one row per unique normalized sequence**.

Normalize for exact-sequence deduplication by:

1. stripping leading/trailing whitespace,
2. removing internal whitespace,
3. removing gaps and formatting artifacts,
4. converting standard amino-acid letters to uppercase,
5. separating explicitly reported terminal modification annotations from the sequence string.

If the same normalized sequence appears in multiple sources:

- merge all useful provenance into `source`,
- merge distinct assay information into `notes`,
- preserve aliases,
- preserve distinct MIC / HC50 measurements,
- preserve conflicts,
- set `wetlab=yes` if any credible source demonstrates antimicrobial wet-lab validation.

Do not collapse distinct residue sequences merely because they share a name.

Do not perform fuzzy-sequence deduplication. The canonical row key is exact normalized sequence.

---

# SOURCE AND EVIDENCE PRIORITY

When sources disagree, prefer evidence in approximately this order:

1. primary experimental paper / supplementary dataset,
2. curated primary AMP database,
3. benchmark dataset with traceable provenance,
4. aggregator,
5. repository mirror,
6. secondary review.

However, do not delete conflicting information. Preserve material conflicts in `notes`.

Reviews are useful for source discovery but should not replace primary provenance when the latter is recoverable.

---

# SEARCH RECURSIVELY

Whenever a useful paper, benchmark, review, database, or repository is found:

1. inspect its stated data sources,
2. inspect its references,
3. inspect supplementary files,
4. inspect linked code/data repositories,
5. identify upstream databases,
6. harvest additional AMP datasets not yet covered.

Continue until additional searches are yielding predominantly duplicate sequences rather than substantial numbers of new candidates.

---

# PRIORITIZATION IF RUNTIME IS LIMITED

If exhaustive crawling is impossible, prioritize:

1. large downloadable AMP databases,
2. MarLys and other major aggregations,
3. DBAASP / APD6 / DRAMP / dbAMP / CAMPR / Peptipedia / GRAMPA / QMAP,
4. AMPSphere and other large computational catalogs,
5. supplementary datasets from major AMP discovery/design papers,
6. GitHub / Hugging Face / GitLab / Zenodo datasets,
7. recent papers not yet represented in databases,
8. long-tail historical literature.

Favor high-yield bulk extraction over manually extracting a handful of sequences while large downloadable datasets remain unprocessed.

---

# REQUIRED QUALITY CONTROL

Before returning the final file, programmatically verify:

```python
columns == ["sequence", "wetlab", "source", "notes"]
```

Also verify:

- file delimiter is TAB,
- file is UTF-8,
- LF line endings,
- no row index,
- `sequence` has no nulls,
- `sequence` has no whitespace,
- `sequence` contains only the accepted standard single-letter amino-acid alphabet used by this project,
- `wetlab` contains only `yes`, `no`, or `Unclear`,
- `source` is non-empty,
- there are no duplicate normalized sequences,
- no FASTA headers are in the sequence column,
- obvious HTML / PDF / OCR artifacts are absent,
- `notes` contain no tabs or embedded newlines,
- measurement units are retained,
- ranges / inequality signs are retained,
- predicted and experimental MICs are distinguishable,
- generic Not-AMP / decoy datasets were not accidentally included,
- computationally scanned-but-not-nominated peptide windows were not accidentally included.

Spot-check a sample of rows from every major source against the underlying record.

---

# FINAL DELIVERABLE

Return:

`amp_master_YYYYMMDD.tsv`

as the single dataset file.

Do not return secondary TSV/CSV files.

Below the file in the final response, provide only a short audit summary containing:

```text
Total unique peptide sequences:
wetlab=yes:
wetlab=no:
wetlab=Unclear:
Distinct source datasets/papers reached:
Major sources successfully reached:
Major sources attempted but not reached:
```

The dataset itself is the task. Completeness, provenance, exact-sequence integrity, and correct separation of experimental vs computational evidence are the primary objectives.
