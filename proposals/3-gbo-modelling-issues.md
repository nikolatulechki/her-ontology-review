# Proposal: modelling issues in the Gut-Brain Ontology

*Draft for discussion with UNIPD · Graphwise · 2026-10-16*

**Scope:** the entity, relation and annotation layers of GBO 1.0 (`https://w3id.org/hereditary/ontology/gutbrain/1.0/`). The taxonomy ranks and the bibliographic properties are covered by separate proposals ([taxon-rank-reuse.md](taxon-rank-reuse.md), [semopenalex-publication-migration.md](semopenalex-publication-migration.md)) and are not repeated here.

## Summary

A review of GBO 1.0 found two kinds of problems. Clear errors (a class carrying another class's definition, substances classified as activities, conflicting SKOS declarations, an LLM-generated orphan term, a literal used as an IRI, a malformed ontology header) have already been fixed in the review copy; the README lists each change. This document collects the remaining issues, which need a modelling decision from UNIPD rather than a mechanical fix.

The most important ones for the T4.6 / D4.10 knowledge graph are in section 3: the annotation layer cannot store the GutBrainIE offsets, and extracted relations carry no provenance, so the graph cannot say which paper or sentence asserted a relation.

## 1. Relation definitions disagree with their axioms

For several relations the comment, the name and the domain/range describe three different things. Annotators and query writers read the comment; reasoners read the axioms.

| Property | Comment says | Axioms say |
|---|---|---|
| `changeAbundance` | bacteria or microbiome → disease | Disease ∪ Intervention ∪ Chemical ∪ Drug ∪ Food → Species ∪ Microbiome, i.e. the opposite direction |
| `changeExpression` | bacteria or microbiome change a gene | the domain also includes Intervention, Chemical, Drug, Food and DietarySupplement |
| `locatedIn` | an organism located in the microbiome | AnatomicalSite ∪ Chemical ∪ Mention ∪ Microbiome → AnatomicalSite ∪ Organism ∪ Sentence |
| `interact` | "what can influence disease, disorder or finding" | different unions on each side; a disease can interact with a drug but not the reverse; not symmetric |
| `usedBy` | techniques used on biological entities | Animal ∪ Human ∪ Microbiome → BiomedicalTechnique, so the data reads "Human usedBy Technique" |
| `affect`, `isA` | both: "a relationship between a disease and another disease" | identical text and identical domain/range; nothing tells them apart |

**Proposal.**

1. For each relation, take the GutBrainIE annotation guidelines as the reference and rewrite the comment, then fix the domain and range to match it. Where the corpus uses a relation in both directions, say so in the comment.
2. Rename `usedBy` to `uses` with domain BiomedicalTechnique, or swap its domain and range.
3. Declare `interact` and `comparedTo` as `owl:SymmetricProperty` and give `interact` the same union on both sides.
4. Define `isA` as a subsumption between diseases (or replace it with `skos:broader` on disease concepts) and `affect` as a causal or comorbidity relation, with distinct comments.
5. Fix the typos ("relatioship" three times, "te disease") and the label of `affect`, which has no language tag.

## 2. Overloaded properties: text layer and biology layer mixed

`partOf` and `locatedIn` are each used for two unrelated things:

- `partOf`: Sentence → PaperAbstract/PaperTitle (text), and Gene → Genome, Species → Microbiome (biology).
- `locatedIn`: Mention → Sentence (text), and Microbiome → AnatomicalSite (biology).

Because the unions are so wide, the domain and range constrain almost nothing: `partOf` accepts Metabolite partOf Chemical, which is really a subclass relation, and even Sentence partOf Genome, since Sentence is in the domain and Genome in the range. The mention-in-sentence relation is also stated twice, once by `containedIn` and once by `locatedIn`.

`containedIn` has a second problem: its comment says an entity is contained in a mention. A mention *refers to* an entity; it does not contain it. It also has no range.

**Proposal.**

1. **Biology:** reuse the OBO Relation Ontology: `part of` (`obo:BFO_0000050`) and `located in` (`obo:RO_0001025`), or make the GBO properties sub-properties of them.
2. **Text:** keep a small set of document properties: sentence in title/abstract, mention in sentence. Declare `composedOf` and the text use of `partOf` as `owl:inverseOf` each other, or keep only one direction.
3. **Mention to entity:** replace the entity-side use of `containedIn` with a property from mention to entity, aligned with NIF (`nif:taIdentRef`) or Web Annotation (`oa:hasBody`).

## 3. The annotation layer cannot hold the GutBrainIE data

### 3.1 Offsets

GutBrainIE entities come with `start_idx`, `end_idx` and `location` (title or abstract). GBO has `hasMentionText` and `taggedAs` on `Mention`, and nothing for the offsets. The SemOpenAlex proposal keeps the gold title and abstract precisely because the offsets point into them; without offset properties that reason has nothing to attach to.

**Proposal.** Add offset properties on `Mention`, preferably by reusing NIF:

```turtle
@prefix nif: <http://persistence.uni-leipzig.org/nlp2rdf/ontologies/nif-core#> .
@prefix ex:  <https://example.org/gutbrainie/> .

ex:mention-36532064-a-0-22 a gutbrain:Mention ;
    gutbrain:hasMentionText "Bifidobacterium longum" ;
    nif:beginIndex 0 ;
    nif:endIndex 22 ;
    nif:referenceContext ex:abstract-36532064 ;
    nif:taIdentRef ex:BifidobacteriumLongum .
```

### 3.2 Provenance of relations

Relations such as `changeAbundance` or `influence` link two entity instances directly. Once two papers report the same pair, or one reports the opposite effect, the graph cannot tell which paper, sentence or annotator asserted what. The comment on `hasMentionText` already speaks of "relation mentions", but no class or property represents one.

**Proposal.** Represent each extracted relation as a node that points to its subject, predicate, object and evidence. Three options, in order of preference for GraphDB:

1. **RDF-star:** annotate the triple itself with its evidence. GraphDB supports it natively.
   ```turtle
   << ex:BifidobacteriumLongum gutbrain:influence ex:Depression >>
       gutbrain:assertedIn ex:sentence-36532064-a-3 ;
       gutbrain:annotatedBy ex:expert_7 .
   ```
2. **An n-ary `RelationMention` class** with `subject`, `object`, `relationType` and `evidence` properties. Works in plain RDF and in Ontop mappings.
3. **Nanopublications**, if the KG is to be published as citable assertions.

This connects to the open question on `paperAnnotator` in the SemOpenAlex proposal (section 8, item 5): whatever represents the annotator, PROV or otherwise, should attach to the relation, not only to the paper.

### 3.3 `taggedAs`

`taggedAs` stores the entity category as a string ("drug", "bacteria", "DDF"), which repeats the `rdf:type` of the linked entity and can drift from it. Its comment says it applies to a mention or a paper, but its domain is only `Mention`. Proposal: derive the tag from the entity's type, or keep it only as the raw corpus label and document it as such.

## 4. Class hierarchy and definitions

- **Substances.** Since the clear-error fix, Food, Drug and Chemical are no longer interventions, but they are not related to each other either. Drug, Metabolite and Neurotransmitter should be subclasses of Chemical, which matches their CHEBI mappings. An intervention can then be linked to the substance it administers instead of being one.
- **Microbiota and Microbiome** are unrelated classes, although `partOf` allows Species → Microbiota and the relations `isLinkedTo`, `impact` and `producedBy` use only Microbiome. Decide whether Microbiota is a part of a Microbiome (the OHMI definition suggests so) and say so in an axiom.
- **Techniques under Assay.** `StatisticalTechnique` and `BiomedicalTechnique` are subclasses of `genomics:Assay` (OBI_0000070). In OBI an assay measures a material entity, which a statistical method does not. `BiomedicalTechnique` also maps to NCIT_C15188, which NCIt labels **Biomedical Research**, and its definition ("Research concerned with…") describes a field of research, not a technique. Both need a new parent (for example OBI "planned process" or EDAM "operation") and a correct mapping.
- **Definition sources.** Microbiota's definition is credited to Wikipedia but the class maps to NCIT_C93019; Metabolite's is credited to NCIT but the class maps to CHEBI_25212. The definition should come from the mapped term.
- **No disjointness.** The file has no `owl:disjointWith`. Basic disjointness (organisms vs. substances vs. processes vs. text) would have made the Food/Gene and substance/intervention errors show up as inconsistencies.
- **Unused classes.** `ExperimentalResult`, `Specimen` and `SpecimenCollectionProcess` appear in no relation. Connect them to the rest of the model or leave them out until they are needed.

## 5. Terms owned by other modules

This file gives labels, definitions and mappings to terms in the BTO and HERO-Genomics namespaces (`bto:Gene`, `bto:AnatomicalSite`, `bto:InterventionOrProcedure`, `bto:DiseaseDisorderOrFinding`, `genomics:Assay`, `genomics:Species`) without importing those modules. If the copies drift from the originals, merging the modules gives two definitions per term. Proposal: `owl:imports` the modules (or a MIREOT-style extract of the terms used) and drop the local copies. A detail: `bto:AnatomicalSite` is labelled "Anatomic Site" here.

## 6. Naming and annotation conventions

- **Generic names on narrow properties.** `relatedTo` (Neurotransmitter → Microbiome only), `impact` (Drug → Microbiome only) and `target` read as general-purpose relations. Either give them specific names or make them general and constrain each use with SHACL, as proposed for the taxon ranks.
- **Labels.** Object property labels are camelCase (`"changeAbundance"`). Use readable labels ("change abundance") and keep the camelCase in the IRI.
- **Definitions.** Definitions are in `rdfs:comment` with an inline "[Definition Source: X]". Use `skos:definition` (or IAO_0000115) for the text and `dcterms:source` with an IRI for the source, so both can be queried.
- **Literal ranges.** `rdf:PlainLiteral` is valid OWL 2 but poorly supported by RDF tools. Use `xsd:string`, or `rdf:langString` for natural-language text.
- **Header metadata.** Add `dcterms:created`, `dcterms:modified` and `dcterms:issued`, and `vann:preferredNamespacePrefix "gutbrain"` with `vann:preferredNamespaceUri`.
- **Leftover declaration.** `rdfs:Class a owl:Class` has no effect and can be removed.

## 7. Decisions for UNIPD

1. Which source defines each relation's direction and meaning: the GutBrainIE annotation guidelines (recommended) or the current comments?
2. Reuse RO for biological part-of and located-in (recommended), or keep GBO-specific properties.
3. Provenance model for relations: RDF-star (recommended for GraphDB), an n-ary `RelationMention` class (simplest for Ontop), or nanopublications.
4. Adopt NIF for mentions and offsets (recommended), or add GBO-specific offset properties.
5. Import BTO and HERO-Genomics into GBO, or keep GBO self-contained and stop redefining their terms.
6. Timing: as with the other proposals, document the changes in D3.10 (internal review at the end of November 2026). Items in sections 1, 4 and 6 fit a 1.1 release; sections 2, 3 and 5 change how data is produced and fit 2.0.
