# Proposal: reuse existing taxonomy vocabularies for taxa and ranks in the Gut-Brain Ontology

*Draft for discussion with UNIPD · Graphwise · 2026-10-16*

**Scope:** the taxonomic part of GBO 1.0 (`gutbrain:Domain`, `SuperKingdom`, `Phylum`, `Class`, `Order`, `Family`, `Genus`, `genomics:Species`) and the agreed move to `:Taxon`, `:hasTaxonomicRank`, `:broaderTaxon` and `:broaderRank`.

## Summary

GBO currently models ranks as a chain of subclasses (`Species ⊑ Genus ⊑ Family ⊑ … ⊑ Domain`). Under that model, a genus such as *Bifidobacterium* used in a relation whose domain is `genomics:Species` is inferred to be a species, and through the chain, every other rank too. The agreed fix is to make every taxon an instance of `:Taxon`, attach its rank as a value, and build the tree with a parent-taxon property.

That pattern already exists in several established vocabularies. We propose that GBO reuse their rank individuals and align its properties with them, rather than minting its own `:Species`, `:Genus` and so on.

## 1. What already exists

| Source | Rank values | Taxon → rank | Taxon → parent |
|---|---|---|---|
| [TAXRANK](https://obofoundry.org/ontology/taxrank.html) (OBO Foundry) | individuals, e.g. `TAXRANK:0000006` species, `TAXRANK:0000005` genus | — | — |
| [NCBITaxon](https://obofoundry.org/ontology/ncbitaxon.html) | `NCBITaxon:species`, `NCBITaxon:genus`, … (TAXRANK cross-references them) | `ncbitaxon:has_rank` | `rdfs:subClassOf` |
| [Darwin Core](https://dwc.tdwg.org/terms/) | strings ("species", "genus") | `dwc:taxonRank` | `dwc:parentNameUsageID` |
| [Wikidata](https://www.wikidata.org/) | items, e.g. `wd:Q7432` species, `wd:Q34740` genus | `wdt:P105` (taxon rank) | `wdt:P171` (parent taxon) |
| [SKOS](https://www.w3.org/TR/skos-reference/) | — | — | `skos:broader` / `skos:broaderTransitive` |

The TAXRANK IDs that matter for GBO, checked against the current `taxrank.obo` on 2026-10-16:

| GBO class today | TAXRANK | NCBITaxon cross-reference |
|---|---|---|
| — (missing in GBO) | `TAXRANK:0000037` domain | — |
| `gutbrain:Domain` (NCIt match: Superkingdom) | `TAXRANK:0000022` superkingdom | `NCBITaxon:superkingdom` |
| `gutbrain:SuperKingdom` (NCIt match: Kingdom) | `TAXRANK:0000017` kingdom | `NCBITaxon:kingdom` |
| `gutbrain:Phylum` | `TAXRANK:0000001` phylum | — |
| `gutbrain:Class` | `TAXRANK:0000002` class | `NCBITaxon:class` |
| `gutbrain:Order` | `TAXRANK:0000003` order | `NCBITaxon:order` |
| `gutbrain:Family` | `TAXRANK:0000004` family | `NCBITaxon:family` |
| `gutbrain:Genus` | `TAXRANK:0000005` genus | `NCBITaxon:genus` |
| `genomics:Species` | `TAXRANK:0000006` species | `NCBITaxon:species` |

TAXRANK also has the intermediate ranks (subgenus, subfamily, strain, …) that real microbial lineages need. GBO doesn't model them today.

### Defects in the current rank classes that reuse removes

- **Kingdom is missing, and the upper ranks are mislabelled.** In GBO, `Phylum` goes straight to `SuperKingdom`, which goes to `Domain`. But the definition on `gutbrain:SuperKingdom` describes the five kingdoms (Monera, Protista, Plantae, Fungi, Animalia), and its `skos:exactMatch` is `NCIT_C45276`, which NCIt labels **Kingdom**. Likewise, `gutbrain:Domain` matches `NCIT_C62706`, which NCIt labels **Superkingdom**. The labels are shifted one level up. In practice GBO has Kingdom and Superkingdom under the wrong names, and no class means Domain. With TAXRANK, each level has its own identifier: domain `0000037`, superkingdom `0000022`, kingdom `0000017`.
- **A class named `Class`.** `gutbrain:Class` reads like `owl:Class` or `rdfs:Class` in Turtle, in SPARQL, and in any tool that shows local names or labels: "`?x a :Class`" looks like a metaclass query. Once ranks are values, the taxonomic class rank is simply `TAXRANK:0000002` (label "class"), and GBO no longer has a term with that local name. If GBO must keep a local term, it should be named `ClassRank` or `TaxonomicClass`.

## 2. Proposal

1. **Use TAXRANK individuals as the rank values.** `:hasTaxonomicRank` points to `TAXRANK:…` IRIs. GBO does not define its own rank individuals. The current rank classes are deprecated, and each one gets a `skos:exactMatch` to its TAXRANK term so that existing data can be migrated.
2. **Align the rank property.** Declare `:hasTaxonomicRank` with `skos:exactMatch` to `wdt:P105` and `skos:closeMatch` to `dwc:taxonRank`. Darwin Core's version takes a string, which is why it's only a close match.
3. **Make `:broaderTaxon` a sub-property of `skos:broader`.** Then `skos:broaderTransitive` answers "everything under Firmicutes" without a custom property path. Add `skos:exactMatch wdt:P171`.
4. **Link taxa to NCBITaxon, don't copy it.** Each taxon instance in the GutBrainIE graph gets a `skos:exactMatch` to its `NCBITaxon_…` IRI, as `gutbrain:Human` already does with `NCBITaxon_9606`. The lineage can then be loaded from NCBITaxon, so annotators don't have to enter it by hand.
5. **Keep `:broaderRank` as the only GBO-specific piece.** TAXRANK has no ordering between ranks, so GBO states the order itself (species → genus → … → domain) between the TAXRANK individuals. SHACL can then check that a taxon's broader taxon has a *higher* rank. That tolerates intermediate ranks and NCBI "no rank" nodes. A rule like "every species must have a genus as its parent" would reject them.

## 3. Example

*Bifidobacterium longum* up to Bacteria. The `ex:` instance IRIs are illustrative.

```turtle
@prefix gutbrain: <https://w3id.org/hereditary/ontology/gutbrain/schema/> .
@prefix obo:      <http://purl.obolibrary.org/obo/> .
@prefix skos:     <http://www.w3.org/2004/02/skos/core#> .
@prefix rdfs:     <http://www.w3.org/2000/01/rdf-schema#> .
@prefix wdt:      <http://www.wikidata.org/prop/direct/> .
@prefix ex:       <https://example.org/gutbrain/> .

# schema
gutbrain:broaderTaxon rdfs:subPropertyOf skos:broader ;
    skos:exactMatch wdt:P171 .
gutbrain:hasTaxonomicRank skos:exactMatch wdt:P105 .

obo:TAXRANK_0000006 gutbrain:broaderRank obo:TAXRANK_0000005 .   # species < genus
obo:TAXRANK_0000005 gutbrain:broaderRank obo:TAXRANK_0000004 .   # genus < family

# data
ex:BifidobacteriumLongum a gutbrain:Taxon ;
    rdfs:label "Bifidobacterium longum" ;
    gutbrain:hasTaxonomicRank obo:TAXRANK_0000006 ;
    gutbrain:broaderTaxon ex:Bifidobacterium ;
    skos:exactMatch obo:NCBITaxon_216816 .

ex:Bifidobacterium a gutbrain:Taxon ;
    rdfs:label "Bifidobacterium" ;
    gutbrain:hasTaxonomicRank obo:TAXRANK_0000005 ;
    gutbrain:broaderTaxon ex:Bifidobacteriaceae ;
    skos:exactMatch obo:NCBITaxon_1678 .
```

To find every taxon under a given phylum:

```sparql
SELECT ?t WHERE { ?t skos:broader+ ex:Actinomycetota }
```

With a reasoner that applies `rdfs:subPropertyOf` and SKOS, `?t skos:broaderTransitive ex:Actinomycetota` returns the same taxa.

## 4. Open points

- **Domain vs. superkingdom.** In 2025 NCBI replaced its "superkingdom" rank with "domain". So for bacteria, the top of the lineage should be `TAXRANK:0000037` domain, then phylum, with kingdom used only where NCBI assigns one. Whether GBO needs superkingdom at all is open.
- **Phylum renames.** NCBI now uses the new prokaryote phylum names (Bacillota for Firmicutes, Actinomycetota for Actinobacteria). Corpus mentions use the old ones. The linked NCBITaxon node keeps both names as synonyms, so this can be resolved during entity linking.
- **Relations typed with `genomics:Species`.** Their domains and ranges should change to `:Taxon`. Where a relation really only applies to species, that becomes a SHACL constraint on the rank, not an OWL range.
