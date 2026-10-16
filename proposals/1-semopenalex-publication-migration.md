# Proposal: reuse SemOpenAlex for the bibliographic part of the Gut-Brain Ontology

*Draft for discussion with UNIPD · Graphwise · 2026-10-15*

**Scope:** the publication-related terms of GBO 1.0 (`https://w3id.org/hereditary/ontology/gutbrain/1.0/`). Entities, relations and the taxonomy are out of scope.

## Summary

GBO describes the papers of the GutBrainIE corpus with its own class, `gutbrain:Paper`, and four string-valued properties: `paperId`, `paperAuthor`, `paperJournal` and `paperYear`. The same information already exists as linked data in [SemOpenAlex](https://semopenalex.org/), the RDF version of [OpenAlex](https://openalex.org/) built by KIT (AIFB) and metaphacts, together with the [converters](https://github.com/metaphacts/semopenalex/tree/main/transformation-scripts) that turn OpenAlex records into RDF. There is no reason for GBO to maintain a parallel bibliographic model.

We checked the whole GutBrainIE 2025 corpus. All 1,647 PMIDs resolve to an OpenAlex work: 1,644 directly in the published SemOpenAlex graph, and the remaining 3 through the OpenAlex API.

We therefore propose that GBO stop describing publications. Each `gutbrain:Paper` gets one link to its SemOpenAlex work, the four bibliographic properties are deprecated, and GBO keeps only what is specific to the corpus: the annotated title and abstract text, sentences, mentions, the annotator and the collections.

## 1. Why SemOpenAlex

- **The conversion already exists.** SemOpenAlex documents how every OpenAlex work attribute maps to RDF ([mapping table](https://github.com/metaphacts/semopenalex/blob/main/transformation-scripts/README.md)) and publishes the scripts that apply it. The output is online: a public SPARQL endpoint at `https://semopenalex.org/sparql`, dereferenceable IRIs, about 26 billion triples, released under CC0 (Färber et al., [*SemOpenAlex: The Scientific Landscape in 26 Billion RDF Triples*](https://doi.org/10.1007/978-3-031-47243-5_6), ISWC 2023). A home-grown PubMed-to-RDF converter, with its own author and journal disambiguation, would redo this work.
- **Identifiers instead of strings.** GBO keeps the author list as one string and the journal as a name. SemOpenAlex provides ordered authorships, authors with ORCID iDs, affiliations, journals as entities with an ISSN-L, DOIs, PubMed's MeSH indexing, citations (`cito:cites`), topics and funders.
- **Cleaner data.** The gold journal string matches the SemOpenAlex journal name in only 80% of documents (for example "Frontiers in immunology" against "Frontiers in Immunology"). With journal IRIs the comparison is no longer needed.
- **Standard vocabularies.** SemOpenAlex reuses Dublin Core Terms, FaBiO, CiTO, DataCite and PRISM wherever they fit, so GBO gets SPAR alignment without designing it.
- **Same technology.** SemOpenAlex ships a GraphDB preload setup, and HEREDITARY's federated stack runs GraphDB with Ontop.
- **No privacy concern.** Bibliographic metadata is public and CC0, so loading it next to the corpus does not touch HEREDITARY's rule that sensitive data stays in the local stores.
- **Less to maintain.** Four properties, their datatypes and their documentation leave GBO.

## 2. What we checked

Checked on 2026-10-15 against the [GutBrainIE 2025 full collection](https://zenodo.org/records/16845409), the OpenAlex API and the public SemOpenAlex endpoint. The queries are in the appendix.

| Check | Result |
|---|---|
| Unique PMIDs: platinum 111, gold 208, silver 499, bronze 749, dev 40, test 40 | 1,647, none in two collections |
| Found in SemOpenAlex by `fabio:hasPubMedId` | 1,644 |
| Found through the OpenAlex API | 1,644 |
| Found by at least one of the two | 1,647 (100%) |
| Author count equal to the gold author list | 1,631 of 1,644 |
| Publication year equal to the gold year | 1,412 of 1,644 |
| Journal name equal to the gold journal, ignoring case | 1,312 of 1,644 |
| Title identical to the gold title | 44 of 1,644 |
| Abstract available in SemOpenAlex | 1,290 of 1,644 |
| Abstract identical to the gold abstract | 305 of 1,290 |

What the numbers mean:

- **The two lookups complement each other.** PMIDs 32566740, 34172092 and 38353184 are found only through the live OpenAlex API. PMIDs 37604976, 39078158 and 39310761 are found only in SemOpenAlex; the live API does not return them for those PMIDs. One PMID, 34841505, maps to two duplicate SemOpenAlex works (W4286950022 and W3215257356).
- **Identity is reliable even though titles are rarely identical.** 1,006 titles differ only by the trailing period PubMed adds, 510 only in punctuation, spacing or case, and 4 only by HTML markup. The remaining 80 include truncated subtitles, escaped entities (`&amp;`) and appended noise such as "Table 1.".
- **Years differ for 14% of documents.** SemOpenAlex gives an earlier year in 210 cases, 196 of them by one year, which is consistent with online-first against issue dates, and a later year in 22.
- **The corpus is not only research articles.** SemOpenAlex types 911 of the works as articles and 714 as reviews; the other 20 are preprints, editorials or letters.

## 3. Target model

GBO keeps one node per corpus document, meaning the title and abstract as they were annotated, and links it to the publication. Everything bibliographic comes from SemOpenAlex. The example is PMID 36532064 from the dev set; the GBO-side instance IRIs (`ex:`) are illustrative.

Prefixes used in the examples:

```turtle
@prefix gutbrain: <https://w3id.org/hereditary/ontology/gutbrain/schema/> .
@prefix soa:      <https://semopenalex.org/ontology/> .
@prefix dcterms:  <http://purl.org/dc/terms/> .
@prefix dcmitype: <http://purl.org/dc/dcmitype/> .
@prefix fabio:    <http://purl.org/spar/fabio/> .
@prefix datacite: <http://purl.org/spar/datacite/> .
@prefix prism:    <http://prismstandard.org/namespaces/basic/2.0/> .
@prefix foaf:     <http://xmlns.com/foaf/0.1/> .
@prefix dbo:      <https://dbpedia.org/ontology/> .
@prefix owl:      <http://www.w3.org/2002/07/owl#> .
@prefix rdfs:     <http://www.w3.org/2000/01/rdf-schema#> .
@prefix skos:     <http://www.w3.org/2004/02/skos/core#> .
@prefix sh:       <http://www.w3.org/ns/shacl#> .
@prefix xsd:      <http://www.w3.org/2001/XMLSchema#> .
@prefix ex:       <https://example.org/gutbrainie/> .
```

Today, following the GBO 1.0 property definitions:

```turtle
ex:paper-36532064 a gutbrain:Paper ;
    gutbrain:paperId 36532064 ;
    gutbrain:paperAuthor "Maria L Elkjaer; Lukas Simon; Tobias Frisch; …" ;
    gutbrain:paperJournal "Frontiers in immunology" ;
    gutbrain:paperYear "2022"^^xsd:gYear ;
    gutbrain:paperAnnotator "expert_7" ;
    gutbrain:hasTitle ex:title-36532064 ;
    gutbrain:hasAbstract ex:abstract-36532064 .
```

Proposed. GBO asserts only this:

```turtle
ex:paper-36532064 a gutbrain:Paper ;
    dcterms:source <https://semopenalex.org/work/W4310558192> ;
    gutbrain:paperAnnotator "expert_7" ;
    gutbrain:hasTitle ex:title-36532064 ;
    gutbrain:hasAbstract ex:abstract-36532064 .
```

The rest is loaded from SemOpenAlex (abridged, values as published):

```turtle
<https://semopenalex.org/work/W4310558192> a soa:Work ;
    fabio:hasPubMedId "https://pubmed.ncbi.nlm.nih.gov/36532064" ;
    datacite:doi "https://doi.org/10.3389/fimmu.2022.1043579" ;
    dcterms:title "Hypothesis of a potential BrainBiota and its relation to CNS autoimmune inflammation" ;
    fabio:hasPublicationYear 2022 ;
    prism:publicationDate "2022-12-02"^^xsd:date ;
    soa:workType "article" ;
    dcterms:creator <https://semopenalex.org/author/A5032641208> ;    # first author shown
    soa:hasAuthorship <https://semopenalex.org/authorship/W4310558192A5032641208> ;
    soa:hasPrimaryLocation <https://semopenalex.org/location/W43105581921> .

<https://semopenalex.org/authorship/W4310558192A5032641208>
    soa:position 0 ;
    soa:hasAuthor <https://semopenalex.org/author/A5032641208> ;
    soa:hasOrganization <https://semopenalex.org/institution/I177969490> .

<https://semopenalex.org/author/A5032641208>
    dbo:orcidId "https://orcid.org/0000-0002-8596-142X" .

<https://semopenalex.org/location/W43105581921>
    soa:hasSource <https://semopenalex.org/source/S2595292759> .

<https://semopenalex.org/source/S2595292759>
    foaf:name "Frontiers in Immunology" ;
    fabio:hasIssnL "1664-3224" .
```

Why a separate GBO node instead of using the work IRI directly:

- The gold title and abstract differ from SemOpenAlex's (section 2), and entity offsets point into the gold text. On a single node, one resource would carry two different titles.
- The annotator and the collection describe the corpus document, not the publication. If a later edition re-annotates a paper, the two annotations stay apart.
- GBO asserts nothing on IRIs it does not own, so reloading SemOpenAlex can never conflict with GBO data.

Using the work IRI as the paper would also work for the 2025 corpus, where no PMID appears twice, and saves one node per paper. It is listed as a decision in section 8.

The link uses `dcterms:source`, "a related resource from which the described resource is derived", which GBO already declares. If GBO later adopts PROV for annotation provenance, `prov:wasDerivedFrom` is the equivalent.

## 4. Changes to the ontology

### 4.1 Term by term

| GBO 1.0 term | Action | Replacement |
|---|---|---|
| `gutbrain:paperId` | deprecate | `fabio:hasPubMedId` on the work |
| `gutbrain:paperAuthor` | deprecate | `dcterms:creator` (an `soa:Author`); `soa:hasAuthorship` with `soa:position`, `soa:hasAuthor`, `soa:hasOrganization` |
| `gutbrain:paperJournal` | deprecate | `soa:hasPrimaryLocation/soa:hasSource` (an `soa:Source` with `foaf:name`, `fabio:hasIssnL`) |
| `gutbrain:paperYear` | deprecate | `fabio:hasPublicationYear`, `prism:publicationDate` |
| none | add | `dcterms:source` from `gutbrain:Paper` to `soa:Work` |
| `gutbrain:Paper`, `gutbrain:PaperTitle`, `gutbrain:PaperAbstract` | keep, redefine | none; they hold the annotated text |
| `gutbrain:hasTitle`, `hasAbstract`, `hasTitleText`, `hasAbstractText` | keep | none |
| `gutbrain:paperAnnotator` | keep | none; annotation provenance |
| `gutbrain:PaperCollection`, `gutbrain:contains` | fix | `dcmitype:Collection`, `dcterms:hasPart` |

### 4.2 Change list

1. **Deprecate `paperId`, `paperAuthor`, `paperJournal` and `paperYear`.** Mark them `owl:deprecated true` with `dcterms:isReplacedBy` in 1.1 and delete them in 2.0. For example:

   ```turtle
   gutbrain:paperJournal owl:deprecated true ;
       dcterms:isReplacedBy soa:hasPrimaryLocation ;
       skos:editorialNote "Use soa:hasPrimaryLocation/soa:hasSource on the soa:Work linked with dcterms:source."@en .
   ```

2. **Add the link and constrain it with SHACL.** Every `gutbrain:Paper` has exactly one `dcterms:source`, and it is an `soa:Work`. GBO types `dcterms:source` as an annotation property, which cannot appear in an OWL restriction, so the constraint goes into a shapes file. This also follows the SHACL direction discussed for the taxonomy ranks.

   ```turtle
   gutbrain:PaperShape a sh:NodeShape ;
       sh:targetClass gutbrain:Paper ;
       sh:property [
           sh:path dcterms:source ;
           sh:class soa:Work ;
           sh:minCount 1 ;
           sh:maxCount 1
       ] .
   ```

3. **Redefine `gutbrain:Paper`.** Its definition, "an article reporting original research, published in a peer-reviewed journal", does not describe what the class holds, and 714 of the corpus works are reviews. Proposed: "A document of the GutBrainIE corpus: the title and abstract of a publication as they were annotated. The publication itself is the linked `soa:Work`." The IRI and label stay.

4. **Redefine `gutbrain:PaperTitle` and `gutbrain:PaperAbstract` as the annotated text.** Replace the generic NCIT definitions with "the title (abstract) exactly as annotated; entity offsets refer to this text". `hasTitleText`, `hasAbstractText`, `Sentence`, `composedOf` and `partOf` stay as they are.

   The gold text has to stay in GBO because GutBrainIE entity offsets (`start_idx`, `end_idx`, `location`) point into it, and SemOpenAlex holds a different string. The converter rebuilds abstracts from OpenAlex's inverted index joined with single spaces, and its `clean()` function strips double quotes, backslashes, tabs and line breaks from titles and abstracts. In the corpus, 160 SemOpenAlex abstracts start with a stray "Abstract", 354 works have no abstract, and titles lack PubMed's trailing period.

5. **Fix the paper collection.** `gutbrain:contains` has `rdfs:domain skos:Collection` although the documentation says `PaperCollection`, and `gutbrain:PaperCollection` appears in no axiom. `skos:Collection` groups SKOS concepts, not documents. Proposed: make `gutbrain:PaperCollection` a subclass of `dcmitype:Collection`, deprecate `gutbrain:contains` in favour of `dcterms:hasPart`, and create one instance per GutBrainIE collection (platinum, gold, silver, bronze, dev, test).

6. **Reference SemOpenAlex terms instead of importing the ontology.** GBO uses SemOpenAlex terms only as annotation values (`dcterms:isReplacedBy`) and in the shapes file, so it needs no `owl:imports`. Importing the SemOpenAlex ontology would also break OWL 2 DL, which does not allow one property to be both an annotation property and a data or object property:

   | Property | GBO declares it as | SemOpenAlex declares it as |
   |---|---|---|
   | `dcterms:abstract`, `dcterms:license`, `dcterms:publisher` | annotation property | datatype property |
   | `dcterms:created`, `dcterms:modified` | annotation and datatype property | datatype property |

   GBO already gives 12 Dublin Core properties both types, because it inlines the DCMI schema. Removing that inlined schema, as we propose for the `dc:` declarations, solves both problems.

7. **Version.** Items 1 to 6 add terms or deprecate them, so they fit a 1.1 release. Deleting the deprecated terms makes 2.0.

## 5. Changes to the data and the ingestion pipeline

1. **Resolve PMIDs to works.** Look up `fabio:hasPubMedId` in SemOpenAlex. Its value is the PubMed URL as a string, `"https://pubmed.ncbi.nlm.nih.gov/36532064"`, not the integer GBO stores. If that fails, ask the OpenAlex API (`https://api.openalex.org/works/pmid:36532064`); the work IRI is `https://semopenalex.org/work/` followed by the OpenAlex ID, which is the rule the converter uses. When a PMID maps to more than one work, take the one the OpenAlex API returns: for 34841505 that is W3215257356. Publish the lookup table (PMID, OpenAlex ID, lookup source, date) with the corpus.
2. **Load the bibliographic subgraph.** CONSTRUCT the works, authorships, authors, locations and sources from the SemOpenAlex endpoint into a dedicated named graph in GraphDB, separate from GBO data so it can be dropped and reloaded. For works missing from the snapshot, fetch the OpenAlex JSON and apply the SemOpenAlex works mapping. The published script is a batch job over the entire OpenAlex dump: `semopenalex-works.py` downloads `data/works/` from the OpenAlex S3 bucket as soon as it starts and runs 16 worker processes. For a handful of works we would run its per-record mapping on a small JSONL file from the API instead. We recommend mirroring rather than federating live queries to semopenalex.org, because we have no availability guarantee for the public endpoint.
3. **Stop producing the deprecated triples.** Remove `paperId`, `paperAuthor`, `paperJournal` and `paperYear` from the ingestion and OBDA mappings, and add `dcterms:source` from the lookup table.
4. **Keep the gold text untouched.** `hasTitleText` and `hasAbstractText` keep coming from the corpus files.
5. **Re-resolve on every corpus release.** OpenAlex IDs change when OpenAlex merges duplicate records, and SemOpenAlex is a static snapshot: the example work was last modified on 2025-01-21.

## 6. Documentation and queries

- **Rewrite the "Paper Modeling" section of the GBO documentation.** It already disagrees with the ontology file in three places: the domain of `contains` (documentation: `PaperCollection`; file: `skos:Collection`), the datatype of `paperYear` (documentation: `xsd:integer`; file: `xsd:gYear`), and how sentences attach (documentation: to the paper through `partOf`; file: `partOf` from `Sentence` to `PaperTitle` or `PaperAbstract`).
- **Queries that read the deprecated properties gain one hop.** For example, journal names (prefixes as in section 3). In GBO 1.0:

  ```sparql
  SELECT ?paper ?journal WHERE {
    ?paper a gutbrain:Paper ;
           gutbrain:paperJournal ?journal .
  }
  ```

  Proposed:

  ```sparql
  SELECT ?paper ?journal ?issnL WHERE {
    ?paper a gutbrain:Paper ;
           dcterms:source ?work .
    ?work soa:hasPrimaryLocation/soa:hasSource ?source .
    ?source foaf:name ?journal .
    OPTIONAL { ?source fabio:hasIssnL ?issnL }
  }
  ```

## 7. Caveats about SemOpenAlex

- **Where the data and the ontology disagree, follow the data.** DOIs are published with `datacite:doi` as strings (`"https://doi.org/…"`), while the ontology and its SHACL shapes declare `prism:doi` with `xsd:anyURI`. Publication dates use `prism:publicationDate` in the data but `dcterms:date` in the mapping README. ORCID iDs are strings under `https://dbpedia.org/ontology/orcidId`, with `https` rather than DBpedia's own `http` namespace, so IRIs must be copied exactly as published.
- **Snapshot age.** The repository was last updated on 2025-02-21 and the ontology (version 4.0.0) on 2024-12-18. Works published after the snapshot need the OpenAlex API path above.
- **Licensing.** The SemOpenAlex ontology and dataset are CC0, like OpenAlex itself. The GitHub repository has no license file, so before vendoring or adapting the scripts we should ask KIT and metaphacts to add one.
- **Publication year.** SemOpenAlex's year differs from the gold year for 232 documents; see the decisions below.

## 8. Decisions for UNIPD

1. Keep a GBO document node linked to the work (recommended), or use the SemOpenAlex work IRI as the paper itself.
2. Take SemOpenAlex's publication year as the bibliographic year. The extraction task does not depend on it, and the gold year remains in the corpus files.
3. Mirror the subgraph in HEREDITARY's GraphDB (recommended), or federate queries to semopenalex.org.
4. Timing: document the deprecations in D3.10 (internal review at the end of November 2026), build the T4.6 / D4.10 knowledge graph on the new model, and delete the deprecated terms in GBO 2.0 afterwards.
5. Out of scope here: `paperAnnotator` mixes people and a system (`expert_1` to `expert_7`, `student_A`, `student_B` and `distant`). Modeling it with PROV (`prov:wasAttributedTo` a `prov:Person` or a `prov:SoftwareAgent`) belongs in the review of the annotation layer.

## 9. Rollout

1. Agree on the model with UNIPD before the D3.10 internal review.
2. UNIPD releases GBO 1.1: deprecations, link, definitions, shapes and the collection fix.
3. Graphwise builds the PMID lookup table and the SemOpenAlex subgraph for the 1,647 documents; the lookup is already prototyped (section 2).
4. UNIPD updates the ingestion and OBDA mappings and the queries.
5. Both validate the GBO data together with the loaded SemOpenAlex graph against the shapes, so that every `gutbrain:Paper` has exactly one `soa:Work`.
6. UNIPD releases GBO 2.0 without the deprecated terms.

## 10. Related: MeSH IRIs

GBO links classes to MeSH with NCBI web identifiers: `mesh:` expands to `https://www.ncbi.nlm.nih.gov/mesh/`, so Genome is `mesh:68016678`. SemOpenAlex indexes works with NLM's MeSH RDF IRIs, where Genome is `http://id.nlm.nih.gov/mesh/D016678`. Switching GBO's `skos:exactMatch` targets to the NLM IRIs would make GBO classes joinable with the MeSH indexing of the corpus works.

## Appendix: reproducing the check

The corpus is `GutBrainIE_Full_Collection_2025.zip` from Zenodo record [16845409](https://zenodo.org/records/16845409); the PMIDs are the keys of the JSON files under `Annotations/` and `Test_Data/`.

SemOpenAlex lookup, run in batches of 150 PMIDs against `https://semopenalex.org/sparql`:

```sparql
PREFIX fabio: <http://purl.org/spar/fabio/>

SELECT ?pmid ?work WHERE {
  VALUES ?pmid {
    "https://pubmed.ncbi.nlm.nih.gov/36532064"
    "https://pubmed.ncbi.nlm.nih.gov/34841505"
  }
  ?work fabio:hasPubMedId ?pmid .
}
```

OpenAlex API, up to 50 PMIDs per request:

```text
https://api.openalex.org/works?filter=pmid:36532064|34841505&select=id,ids
```
