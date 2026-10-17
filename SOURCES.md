# Source index

Selected learning spine and primary-source entry points, checked for the first research cut on **16 October 2026**. This index is a navigation aid; the linked local notes contain finer-grained references, inspected sections, evidence ledgers, and limitations. A listing is not a full-reading claim. The transactions and Zanzibar close-reading records were added on **17 October 2026**.

## 1. Core books and courses

- **Silberschatz, Korth, and Sudarshan, Database System Concepts, 7th edition (2019).** [Authors' site](https://www.db-book.com/), [contents](https://www.db-book.com/toc-dir/toc.pdf), [slides](https://www.db-book.com/slides-dir/index.html). Main conceptual spine: relational model, SQL, conceptual design, normalization, then execution and transactions. The complete textbook was not verified as open access. [Reading plan](notes/foundations.md).
- **CMU 15-445/645, Fall 2025.** [Completed course archive](https://15445.courses.cs.cmu.edu/fall2025/), [database course directory](https://db.cs.cmu.edu/courses/). Implementation companion for storage, indexes, query execution, concurrency, and recovery. Use the completed offering as a coherent route; it does not replace conceptual modeling. [Selection](notes/foundations.md).
- **Kleppmann and Riccomini, Designing Data-Intensive Applications, 2nd edition (2026).** [Publisher](https://www.oreilly.com/library/view/designing-data-intensive-applications/9781098119058/cover.html). Systems tradeoffs, evolution, derived data, and end-to-end correctness. Chapter references in the notes refer to this edition, not the first. [Learning guide](notes/lifecycle.md).
- **Hogan et al., Knowledge Graphs (2021).** [Free book site](https://kgbook.org/). Representation, deductive and inductive techniques, creation, and applications. Read after the initial database foundations. [Foundation discussion](notes/foundations.md).
- **Abiteboul, Hull, and Vianu, Foundations of Databases (1995).** [Authors' free text](https://webdam.di.ens.fr/Alice/). Optional theory reference; not required cover-to-cover for the first pass.
- **Readings in Database Systems, 5th edition (2015).** [Editors' PDF](https://www.redbook.io/pdf/redbook-5th-edition.pdf). Optional historical and research reading guide; its age should inform how its systems coverage is used.

## 2. Modeling and integrity

- **Codd (1970), A Relational Model of Data for Large Shared Data Banks.** [DOI](https://doi.org/10.1145/362384.362685). Data independence and the original relational proposal. Do not attribute the full later BCNF theory to this paper. [Close-reading lesson](notes/01-modeling-and-integrity.md).
- **Chen (1976), The Entity-Relationship Model—Toward a Unified View of Data.** [DOI](https://doi.org/10.1145/320434.320440). Conceptual entities, relationships, roles, identification, and representation. [Close-reading lesson](notes/01-modeling-and-integrity.md).
- **PostgreSQL 18 documentation.** [Constraints](https://www.postgresql.org/docs/18/ddl-constraints.html), [partial indexes](https://www.postgresql.org/docs/18/indexes-partial.html), [CREATE VIEW](https://www.postgresql.org/docs/18/sql-createview.html). Concrete implementation boundaries, including NULL behavior and view updatability; not claims about all SQL systems. [Lesson](notes/01-modeling-and-integrity.md).

- **PostgreSQL 18 transactions and maintenance.** [Isolation](https://www.postgresql.org/docs/18/transaction-iso.html), [explicit locking](https://www.postgresql.org/docs/18/explicit-locking.html), [serialization retries](https://www.postgresql.org/docs/18/mvcc-serialization-failure-handling.html), [materialized-view refresh](https://www.postgresql.org/docs/18/sql-refreshmaterializedview.html), [logical decoding](https://www.postgresql.org/docs/18/logicaldecoding-explanation.html). See [Module 2](notes/02-transactions-and-derived-views.md) and its [section-level evidence ledger](notes/02-source-evidence-ledger.md). Reasoned schedules, not executed experiments.

## 3. Knowledge representation and stable standards

- **Davis, Shrobe, and Szolovits (1993), What Is a Knowledge Representation?** [Primary text](https://groups.csail.mit.edu/medg/ftp/psz/k-rep.html). Representation as multiple commitments and roles.
- **Gruber (1993), A Translation Approach to Portable Ontology Specifications.** [Author-hosted paper](https://tomgruber.org/writing/ontolingua-kaj-1993.pdf). Shared vocabulary and conceptual commitments.
- **W3C RDF 1.1 / RDFS.** [RDF 1.1 Concepts](https://www.w3.org/TR/rdf11-concepts/), [RDFS](https://www.w3.org/TR/rdf-schema/). Graph representation and entailment.
- **W3C OWL 2.** [Primer](https://www.w3.org/TR/owl2-primer/). Open-world reasoning; cardinality is not a required-field form validator.
- **W3C SHACL (2017).** [Recommendation](https://www.w3.org/TR/2017/REC-shacl-20170720/). Validation of a selected data graph against shapes.
- **W3C SKOS.** [Reference](https://www.w3.org/TR/skos-reference/). Controlled vocabularies; broader relations are not interchangeable with subclass axioms.
- **W3C PROV-O (2013).** [Recommendation](https://www.w3.org/TR/2013/REC-prov-o-20130430/). Provenance vocabulary, not a guarantee of truth or an automatic retraction engine.

See [Foundations](notes/foundations.md) for version cautions. RDF 1.2 and SHACL 1.2 drafts encountered in the review are not silently substituted for the mature baselines above.

## 4. Production cases and lifecycle mechanisms

- **Wikidata/Wikibase.** [Data model primer](https://www.mediawiki.org/wiki/Wikibase/DataModel/Primer), [constraints](https://www.wikidata.org/wiki/Help:Property_constraints_portal), [WDQS graph split](https://www.wikidata.org/wiki/Wikidata:SPARQL_query_service/WDQS_graph_split). Qualified statements, references, ranks, advisory checks, and serving tradeoffs.
- **Gene Ontology and OBO.** [Annotations](https://www.geneontology.org/docs/go-annotations/), [evidence codes](https://geneontology.org/docs/guide-go-evidence-codes/), [term stability](https://obofoundry.org/principles/fp-019-term-stability.html), [release pipeline](https://github.com/geneontology/pipeline). Expert curation, semantic changes, and coordinated releases.
- **Uber Databook.** [2018 engineering account](https://www.uber.com/au/en/blog/databook/), [2020 engineering account](https://www.uber.com/py/en/blog/metadata-insights-databook/). Dated reports of metadata collection, schema governance, logs, and derived serving indexes; not verification of current private infrastructure.
- **GitLab Handbook.** [Usage and governance](https://handbook.gitlab.com/handbook/about/handbook-usage/), [editing](https://handbook.gitlab.com/handbook/about/editing-handbook/), [search](https://handbook.gitlab.com/docs/search/). Document-level ownership and review as a legitimate mature knowledge-system design.
- **XTDB temporal documentation.** [Time in XTDB](https://docs.xtdb.com/about/time-in-xtdb.html). Concrete valid-time versus system-time distinctions, without a product recommendation.
- **Zanzibar (USENIX ATC 2019).** [Paper](https://www.usenix.org/system/files/atc19-pang.pdf). Content and authorization consistency; historical performance is not a universal service promise. [Close-reading case](notes/zanzibar-content-authorization-consistency.md) and [source ledger](notes/zanzibar-sources.md), added 17 October 2026.

Comparative analysis: [Production systems](notes/production-systems.md). Operational consequences: [Lifecycle engineering](notes/lifecycle.md).

## 5. Evaluation and frontier papers

- **DBSP (PVLDB 2023).** [Original paper](https://www.vldb.org/pvldb/vol16/p1601-budiu.pdf). Incremental view maintenance. [2025 journal extension DOI](https://doi.org/10.1007/s00778-025-00922-y) was checked at metadata level; do not imply the extension was fully read. [Guide](notes/lifecycle.md).
- **LOTUS (PVLDB 2025).** [Paper](https://www.vldb.org/pvldb/vol18/p4171-patel.pdf). Semantic operators and optimization relative to reference algorithms. Operator-level statistical guarantees are not factual-truth guarantees. [Reading note](notes/frontier-readings.md).
- **SemBench, v2 (16 March 2026).** [Versioned text](https://arxiv.org/html/2511.01716v2). Semantic query-engine workloads and evaluation. Treated here as a preprint; venue acceptance was not verified. [Reading note](notes/frontier-readings.md).
- **EMERGE, v2 (7 April 2026).** [Versioned text](https://arxiv.org/html/2507.03617v2). Knowledge-base maintenance evaluation, with important gold-coverage and temporal/literal limits. [Evaluation discussion](notes/evaluation-gaps.md).
- **ALCE (EMNLP 2023).** [Paper record](https://aclanthology.org/2023.emnlp-main.398/). Separate answer quality and citation support rather than conflating the two.
- **RAGTruth (ACL 2024).** [Paper](https://aclanthology.org/2024.acl-long.585.pdf). Support from supplied evidence is not an independent test of source truth.
- **CRAG, inspected v1 (7 June 2024).** [Versioned text](https://arxiv.org/html/2406.04744v1). Static benchmark slices; not live-update, authorization, or organizational-productivity evaluation.
- **Wikidata quality study (2021).** [Paper record](https://arxiv.org/abs/2107.00156). Why removal is an imperfect error proxy; preserve the denominator of each reported result.

The [evaluation memo](notes/evaluation-gaps.md) contains additional primary references and proposed tests. The [synthesis](notes/synthesis.md) distinguishes mature guarantees, deployed patterns, frontier evidence, and questions that remain open within this bounded review.

## 6. Schema evolution and stable identity

Added **17 October 2026**. [Module 3](notes/03-schema-evolution-and-stable-identity.md) works through a mixed-version label migration, write-intent and stale-backfill hazards, rename versus changed meaning, exact replacement, splits, and historical queries. All schedules and acceptance checks are reasoned, unexecuted examples.

- **GitLab migration guidance.** [Avoiding downtime in migrations](https://docs.gitlab.com/development/database/avoiding_downtime_in_migrations/). Phased column changes and old-client dependencies, not a universal zero-downtime guarantee.
- **PostgreSQL 18 ALTER TABLE.** [Reference](https://www.postgresql.org/docs/18/sql-altertable.html). Staged constraint validation and locking; not proof of semantic migration correctness.
- **OBO and GO semantic-change policies.** [OBO term stability](https://obofoundry.org/principles/fp-019-term-stability.html), [GO documentation](https://geneontology.org/docs/ontology-documentation/), and [the GO 2026 update paper](https://doi.org/10.1093/nar/gkaf1292). Distinguish historical alternate-ID merges from the reported later obsoletion/replacement procedure.

The [Module 3 evidence ledger](notes/03-schema-evolution-evidence.md) records inspected sections, source-version limits, and the GO policy reconciliation.
