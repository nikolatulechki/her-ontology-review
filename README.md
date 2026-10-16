# HER ontology review

In-depth review of the **HEREDITARY** (Horizon Europe GA101137074) ontologies — especially the Gut-Brain Ontology (GBO) — for Graphwise use in knowledge-graph construction from GutBrainIE gold data (T4.6 / D4.10).

## Context

| Item | Detail |
|------|--------|
| Project | [HEREDITARY](https://hereditary.dei.unipd.it/) — University of Padua (UNIPD) lead on ontology |
| Official release | https://hereditary.dei.unipd.it/ontology/ |
| Gut-Brain Ontology | https://hereditary.dei.unipd.it/ontology/gutbrain/ |
| Related modules | [HERO-Clinical](https://hereditary.dei.unipd.it/ontology/phenoclinical/), [HERO-Genomics](https://hereditary.dei.unipd.it/ontology/genomics/) |
| Graphwise role | Ontology review + recommendations; KG for GutBrainIE (T4.6 / D4.10) |
| UNIPD deliverable | D3.10 — internal revision due end of Nov 2026; room for adjustments before EC submission |

HERO (HEREDITARY Ontology) refines the Brainteaser Ontology (BTO) for ALS/MS phenoclinical and genomics data, and is extending toward gut–brain interplay. In the federated analytics stack it is the OBDA backbone (GraphDB + Ontop over a virtual lakehouse).

## Open modeling issue (taxonomy ranks)

UNIPD’s first release models taxonomic ranks with `rdfs:subClassOf` (e.g. `Species rdfs:subClassOf Genus`). That wrongly implies that an instance of `Species` is also an instance of `Genus` / `Family`.

Agreed direction (Giorgio Di Nunzio, 2026-10-01): treat taxa as instances of `:Taxon` with `:hasTaxonomicRank` and `:broaderTaxon`, and relate ranks via `:broaderRank` — not via `rdfs:subClassOf` between rank classes. SHACL can enforce rank–broader constraints if needed.

## People

| Role | Who |
|------|-----|
| Graphwise | Ivelina Nikolova, Pavlin Gyurov, Nikola Tulechki |
| UNIPD | Giorgio Maria Di Nunzio, Laura Menotti, Gianmaria Silvello |

## Local ontology copy

GBO **v1.0** (site last-modified 2025-09-22; IRI `https://w3id.org/hereditary/ontology/gutbrain/1.0/`):

| File | Notes |
|------|--------|
| `ontology/hero_gutbrain.ttl` | Turtle (publisher serialization; content-negotiated from w3id) |
| `ontology/hero_gutbrain.rdf` | RDF/XML source from the same release |

Source: https://hereditary.dei.unipd.it/ontology/gutbrain/schema/

## Workspace notes

- Local path: `~/Projects/her-ontology-review`
- Mail filter (Graphwise): `label:hereditary`
- KB project page: `[[her-ontology-review]]`

# Changes 

Fixing namespaces 

I removed <https://w3id.org/hereditary/ontology/gutbrain/schema/conceptSchema/> and the "naked" : prefix
never use the naked prefix in a production ontology. 
in this case we already have one `gutbrain`

I fix gutbrain: prefix and the uri dor the ontology with a trailing / sot that it is pretty in the turtle

Turned all `skos:exactMatch` values from string literals into prefixed IRIs (`obo:`, `snomed:`, `mesh:`, `edam:`); normalized one `CHEBI:25212` to `obo:CHEBI_25212`.

Merged the two Species classes: removed the unused duplicate `gutbrain:Species` and kept `genomics:Species` (the one in the rank chain and in the relation domains/ranges). Replaced its mapping `edam:data_1045` ("Species name", a data item) with `obo:NCIT_C45293` (NCIt Species rank, consistent with the other rank classes); dropped the now-unused `edam:` prefix.

Dropped `dc:` (Dublin Core Elements 1.1) entirely — removed inlined element declarations and all `rdfs:subPropertyOf dc:*` bridges; metadata stays on `dcterms:` only.

Fixed `gutbrain:Food`, which carried the Gene definition and the Gene mapping (`snomed:67271001`, shared with `bto:Gene`): it now has the NCIt Food definition and `skos:exactMatch obo:NCIT_C62695`.

Removed `rdfs:subClassOf bto:InterventionOrProcedure` from `gutbrain:Food`, `gutbrain:Drug` and `gutbrain:Chemical`: NCIT_C25218 is an activity, so every substance was inferred to be one. To keep the existing domain/range entailments, the three classes are now listed explicitly in the unions that used to cover them through `InterventionOrProcedure` (`administered`, `changeAbundance`, `changeExpression`, `containedIn`, `influence`, `interact`, `treatedBy`).

Removed the local declarations of `skos:exactMatch` (as an annotation property, which contradicts the imported SKOS and breaks OWL 2 DL), `skos:broaderTransitive` and `skos:inScheme` (unused); SKOS terms now come only from `owl:imports`.

Removed `obo:OMIT_0003283`, an orphan resource used in no axiom, whose only content was an LLM-generated definition (`llama3-8b-8192`) and a stray "OMIT Match" comment.

Replaced `rdfs:isDefinedBy "…NCIT_C62638"` (a string literal) on `gutbrain:Sentence` with `skos:exactMatch obo:NCIT_C62638`, like the other classes.

Fixed the ontology header: `rdfs:label` is now the title instead of "Author: Samuel Piron" (he is already a `dcterms:creator` via ORCID 0009-0006-5558-4295); `owl:versionInfo` is the string `"1.0"` instead of an `xsd:decimal`; `dcterms:license` is the CC BY 4.0 IRI, and the copyright and funding text moved to `dcterms:rights`.