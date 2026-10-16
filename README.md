# HER ontology review

Review of the **HEREDITARY** (Horizon Europe GA101137074) ontologies, focused on the Gut-Brain Ontology (GBO), for use in knowledge-graph construction from the GutBrainIE gold data (T4.6 / D4.10).

## Context

- Project: [HEREDITARY](https://hereditary.dei.unipd.it/); the ontology work is led by the University of Padua (UNIPD).
- Ontologies: [official release](https://hereditary.dei.unipd.it/ontology/), [Gut-Brain Ontology](https://hereditary.dei.unipd.it/ontology/gutbrain/), [HERO-Clinical](https://hereditary.dei.unipd.it/ontology/phenoclinical/), [HERO-Genomics](https://hereditary.dei.unipd.it/ontology/genomics/).
- UNIPD deliverable D3.10: internal revision due end of November 2026, with room for adjustments before EC submission.

HERO (HEREDITARY Ontology) refines the Brainteaser Ontology (BTO) for ALS/MS phenoclinical and genomics data and is being extended toward gut–brain interplay. In the federated analytics stack it is the OBDA backbone (GraphDB + Ontop over a virtual lakehouse).

## Contents

- `ontology/hero_gutbrain.ttl`: GBO **v1.0** in Turtle (IRI `https://w3id.org/hereditary/ontology/gutbrain/1.0/`, from the [published schema](https://hereditary.dei.unipd.it/ontology/gutbrain/schema/)), with the fixes listed below applied. The original is in the [initial commit](https://github.com/nikolatulechki/her-ontology-review/commit/b4dc6cb339bda25768c11f2afa6ff08d6b01494a).
- `proposals/`: discussion drafts for UNIPD:
  1. [Reuse SemOpenAlex for the bibliographic part](proposals/1-semopenalex-publication-migration.md)
  2. [Reuse existing vocabularies for taxa and ranks](proposals/2-taxon-rank-reuse.md)
  3. [Modelling issues in entities, relations and annotations](proposals/3-gbo-modelling-issues.md)

## Taxonomic ranks

The first release models ranks with `rdfs:subClassOf` (e.g. `Species rdfs:subClassOf Genus`), which wrongly makes every `Species` instance also a `Genus` and a `Family`. The agreed direction (2026-10-01) is to model taxa as instances of `:Taxon` with `:hasTaxonomicRank` and `:broaderTaxon`, and to relate ranks via `:broaderRank`; SHACL can enforce rank constraints if needed. See proposal 2.

## Fixes applied

- **Namespaces:** removed the unused `…/gutbrain/schema/conceptSchema/` namespace, the bare `:` prefix and `@base` (`gutbrain:` is used throughout); the `gutbrain:` prefix and ontology IRI now end with `/`; the file is reserialized with prefixed names. ([`da8f672`](https://github.com/nikolatulechki/her-ontology-review/commit/da8f6721a7734f3636586411ca968b0cfb147dd5), [`538e3a8`](https://github.com/nikolatulechki/her-ontology-review/commit/538e3a83831b8a789f64d3db7c042932211425c7), [`220ac82`](https://github.com/nikolatulechki/her-ontology-review/commit/220ac82478ed69371ddb9b5f8e9d48c08bd30582))
- **Mappings:** all `skos:exactMatch` values are IRIs (`obo:`, `snomed:`, `mesh:`) instead of string literals; `CHEBI:25212` normalized to `obo:CHEBI_25212`. `genomics:Assay` and `gutbrain:Sentence` now use `skos:exactMatch` (`obo:OBI_0000070`, `obo:NCIT_C62638`) instead of a literal `rdfs:isDefinedBy`. ([`27df9a6`](https://github.com/nikolatulechki/her-ontology-review/commit/27df9a6ac2693683f7d5242b0d79d998a8421c61), [`ff3b664`](https://github.com/nikolatulechki/her-ontology-review/commit/ff3b6645aafd5a3b60a756d314bfd8e885507961), [`8598438`](https://github.com/nikolatulechki/her-ontology-review/commit/8598438f913d2b63fd6e2ee83ed34a79440901c7))
- **Species:** merged the two Species classes, keeping `genomics:Species` (the one in the rank chain and in domains/ranges); its mapping changed from `edam:data_1045` (a data item) to `obo:NCIT_C45293`, and the unused `edam:` prefix was dropped. ([`49bc0b8`](https://github.com/nikolatulechki/her-ontology-review/commit/49bc0b8ae0b9e8339667754d9005352b990bd0f6))
- **Food:** replaced the copied Gene definition and mapping with the NCIt Food definition and `skos:exactMatch obo:NCIT_C62695`. ([`3b0aa9d`](https://github.com/nikolatulechki/her-ontology-review/commit/3b0aa9deeb3ac23c8f46e9a843368cd86fcaa2e5))
- **Substances:** `gutbrain:Food`, `Drug` and `Chemical` are no longer subclasses of `bto:InterventionOrProcedure` (NCIT_C25218 is an activity). They are listed explicitly in the unions that used to cover them, so domain/range entailments of `administered`, `changeAbundance`, `changeExpression`, `containedIn`, `influence`, `interact` and `treatedBy` are unchanged. ([`381b02c`](https://github.com/nikolatulechki/her-ontology-review/commit/381b02cbf19321d05392d17dd88b6c24ddfe2b7a))
- **Dublin Core:** dropped `dc:` (Elements 1.1) declarations and `rdfs:subPropertyOf dc:*` bridges; metadata uses `dcterms:` only. ([`a222854`](https://github.com/nikolatulechki/her-ontology-review/commit/a222854e4c3d2f663d1deb794ee9721af12b185a))
- **SKOS:** removed local redeclarations of `skos:exactMatch` (as an annotation property, which breaks OWL 2 DL), `skos:broaderTransitive` and `skos:inScheme`; SKOS comes only from `owl:imports`. ([`4357e8f`](https://github.com/nikolatulechki/her-ontology-review/commit/4357e8fcccf098d8c224eb2850cc4498309a43b6))
- **Orphans:** removed `obo:OMIT_0003283`, used in no axiom and carrying only an LLM-generated definition. ([`5ae61cb`](https://github.com/nikolatulechki/her-ontology-review/commit/5ae61cb31d13724141b4add13b5da4ca9da2197c))
- **Header:** `rdfs:label` is the ontology title (it previously held an author line); `owl:versionInfo` is the string `"1.0"`; `dcterms:license` is the CC BY 4.0 IRI, with copyright and funding text in `dcterms:rights`. ([`855b21b`](https://github.com/nikolatulechki/her-ontology-review/commit/855b21b8cc877e7e0b2429c235dfe0a2bf606f7a))
