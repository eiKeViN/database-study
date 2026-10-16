# Production knowledge systems: a primary-source reading path

Research date: 16 October 2026. Scope: mature operational knowledge systems; no domain-specific application proposal. The immediate deliverable is a prioritized set of case objects and reading entrypoints, with deeper comparative notes retained below.

## Selection and reading order

1. **Wikidata/Wikibase**: best first case for separating entities, claims, contextual qualification, source attribution and application-facing views. Read its data model before looking at its database. Then read the production scaling reports to see what the elegant logical model costs operationally.
2. **Gene Ontology (GO), situated in OBO**: strongest counterpoint for domain-expert semantics, evidence-bearing annotation, rigorous release discipline and changes to the meaning of terms. Read the annotation specification and obsoletion policy together. This case is particularly good at exposing why a persistent identifier alone cannot ensure stable meaning.
3. **Uber Databook**: best compact engineering case for integrating heterogeneous operational sources, schema ownership, ingestion, event logs and multiple serving indexes. Two first-party accounts document the evolution rather than only the final architecture. Treat its 2020 report as a dated production snapshot, not a verified description of Uber in 2026.
4. **GitLab Handbook**: useful boundary case showing how a mature document knowledge base operates without representing every sentence as a graph assertion. Read it after the other three to avoid equating “knowledge base” with “knowledge graph.” Its editorial and operational documentation is unusually inspectable, but it supplies little rigorous causal evidence about productivity.

For a short syllabus, use cases 1–3 as the core, with case 4 as a document-oriented comparison. These cases complement database courses; none is a substitute for a course on transactions, query optimization or distributed systems.

## 1. Wikidata/Wikibase: an assertion-centric public knowledge system

### Primary reading sequence

**W1. Wikibase DataModel/Primer**, living official documentation, accessed 2026-10-16.
https://www.mediawiki.org/wiki/Wikibase/DataModel/Primer
Read “Statements” and “Ranks.” The critical distinction is between describing an entity and recording a statement about it. A qualifier is part of the assertion’s meaning, not disposable display metadata. Divergent sources can coexist. Preferred/normal/deprecated ranks help choose useful views; they are not calibrated confidence probabilities. [W1]

**W2. Wikibase DataModel**, living technical specification.
https://www.mediawiki.org/wiki/Wikibase/DataModel
Read “Statements,” “Ranks of Statements,” and “ReferenceRecords.” Items and properties have identifiers independent of multilingual labels. A statement combines a principal property/value with qualifiers, references and rank. References can themselves contain structured bibliographic fields; absence of references is permitted by the model. Thus expressiveness is not an evidence-completeness guarantee. [W2]

**W3. RDF Dump Format**, living official specification.
https://www.mediawiki.org/wiki/Wikibase/Indexing/RDF_Dump_Format
Read statement representation, truthy statements and qualifiers. The full RDF representation retains statement nodes and their metadata, while direct best-rank assertions offer a simpler projection. A query author must choose the semantic loss intentionally. Data can refer to deleted items/properties, so consumers cannot assume perfect referential closure. [W3]

**W4. Property constraints portal**, living community documentation.
https://www.wikidata.org/wiki/Help:Property_constraints_portal
Constraints are guidance that permits exceptions rather than hard database integrity barriers. Their declarations are themselves statements on properties. This suits open-world contributions and legitimate anomalies; it leaves downstream applications responsible for deciding which violations matter. [W4]

**W5. Data access and merge guidance**, living documentation.
https://www.wikidata.org/wiki/Help:Data_access
https://www.wikidata.org/wiki/Help:Merging
Read revision-specific entity retrieval and the merge/unmerge sections. A concept identifier differs from the URL of a particular serialization. Consumers can request a specific entity revision. Merged identifiers should redirect and must not be repurposed. Undoing a merge may require addressing later bot substitutions, not merely reverting the original two records. The merge guide is labelled proposed guidance, so distinguish its documented procedures from formally adopted policy. [W5–W6]

**W7. March 2023 WDQS scaling update**, primary maintainer operational report.
https://www.wikidata.org/wiki/Wikidata:SPARQL_query_service/WDQS_backend_update/March_2023_scaling_update
The reported preceding 30-day window had update latency below ten minutes 98% of the time, with a roughly two-minute median. The same report describes query timeouts, occasional divergence from Wikidata, and increasingly fragile reloads. These are measured production observations, not a current SLA or an experimental performance comparison. [W7]

**W8. WDQS graph split**, official migration explanation recording the 9 May 2025 change.
https://www.wikidata.org/wiki/Wikidata:SPARQL_query_service/WDQS_graph_split
At the described scale, the graph exceeded 16 billion triples and grew by about one billion yearly; reloads required one to two months and could fail. Scholarly entities moved to a separate endpoint. Cross-partition queries need federation. Do not repeat its December 2025 legacy-endpoint availability as current in 2026. [W8]

**W9. Internal federation guide**, migration guidance.
https://www.wikidata.org/wiki/Wikidata:SPARQL_query_service/WDQS_graph_split/Internal_Federation_Guide
Read the recommendation to reconsider SPARQL for simple retrieval: known-entity fetching and elementary filters may be better served by entity or search APIs. A general graph query language is not necessarily the best access interface for every client. [W9]

### Questions to work through

- Can your query accidentally conflate a historical value with a current preferred value?
- Which facts disappear when the application switches from full statements to a simplified projection?
- Is entity identity preserved through a merge, and what happens to historical interpretation?
- Is the system bottleneck the authoritative edit store, the asynchronous projection, the query engine, or rebuilding that engine?

**Interpretive lesson:** public editability favors tolerating imperfect content and repairing it continuously. Query-serving infrastructure can become the limiting factor even when the authoring system remains usable. These are coupled but different engineering problems.

## 2. GO/OBO: expert annotations and controlled semantic evolution

### Primary reading sequence

**G1. GO annotations introduction**, official living documentation.
https://www.geneontology.org/docs/go-annotations/
A standard annotation links a gene product to a term via a relation. Validation requires valid identifiers, evidence and a reference, and disallows retracted-publication annotations. These rules constrain publication-ready data, not merely its visual presentation. [G1]

**G2. GAF 2.2 specification**, official exchange format.
https://geneontology.org/docs/go-annotation-file-gaf-format-2.2/
Study its seventeen fields: database/object identity, relation and negation, GO identifier, reference, evidence code, supporting identifiers, taxon, date, assigning group and extensions. Separate record identity, biological context and evidence lineage. The required relation field in 2.2 also illustrates why reading an older 2.1 parser specification is unsafe. [G2]

**G3. Evidence-code guide and external-resource guidelines**, official living documentation.
https://geneontology.org/docs/guide-go-evidence-codes/
https://geneontology.org/docs/guidelines-external/
Evidence codes characterize how an annotation was supported. They are not a universal confidence ordering. Electronic annotation is not individually manually reviewed; that does not mean the generating method lacks quality assessment. The external guidelines explicitly warn against inventing an evidence hierarchy and discuss problematic prediction patterns. [G3–G4]

**G5. GO-CAM overview**, official conceptual introduction, with a link to the 2019 original paper.
https://geneontology.org/docs/gocam-overview/
https://doi.org/10.1038/s41588-019-0500-1
Read the overview first: independent annotations are combined into contextual networks using defined relations from the Relation Ontology. This answers questions about how activities are connected that an unordered collection of gene–term associations cannot express. The paper is a primary follow-on reading, not independently re-evaluated in this research pass. [G5]

**G6. Ontology overview and OBO Principle 19**, official semantics/lifecycle documentation; Principle 19 was formally approved in 2025.
https://www.geneontology.org/docs/ontology-documentation/
https://obofoundry.org/principles/fp-019-term-stability.html
GO retains obsolete terms and identifiers, marks them obsolete and removes their relationships, with reasons and suggested replacements. OBO’s rule is more fundamental: substantially changing a term’s referents requires a new identifier. Exact replacement and an inexact alternative are different relations. Deprecation includes removing usages, preserving useful historical explanation, and communicating changes; changing a label is not an adequate migration. [G6–G7]

**G8. Taxon constraints**, official operational rule documentation.
https://geneontology.org/docs/taxon-constraints/
Taxon restrictions propagate through specified relation types, not indiscriminately through every edge. Incompatible annotations are filtered. Since September 2024, asserted or inferred restrictions are directly attached to affected terms in released ontology products described by the documentation, reducing consumers’ need to rediscover inheritance. The basic edition is an important exception. [G8]

**G9. Production pipeline repository and ontology source repository**, first-party operational documentation.
https://github.com/geneontology/pipeline
https://github.com/geneontology/go-ontology
Read functional branches, manual QC and finalization. Source commits are tested; snapshots and production releases have distinct purposes. Release bundles coordinate ontology, annotations and GO-CAM data. A manual checkpoint evaluates anomalies before finalization, and AmiGO deployment involves an explicit step. Documentation also admits scheduling problems. Do not mistake intended daily/monthly schedules for measured adherence. The ontology repository explicitly warns against treating its source checkout as the production download. [G9–G10]

**G11. The Gene Ontology knowledgebase in 2026**, consortium update, published online 18 December 2025, Nucleic Acids Research 54, D1779–D1792.
https://doi.org/10.1093/nar/gkaf1292
Accessible primary-paper copy:
https://escholarship.org/content/qt20v234dt/qt20v234dt_noSplash_475c0c1035b0ac4880d714f679ab0ab7.pdf
Read “Updates to the ontology” and “Discussion.” The authors report 768 terms added and 4,173 obsoleted in their comparison window; term merging stopped in 2022 in favor of explicit obsoletion/replacement. They describe deliberate top-down simplification after years of bottom-up growth. Over 1,500 GO-CAM pathway models are reported. These are resource-maintenance outcomes, not evidence that every annotation is correct or that all users obtain better results. A prose date in the comparison paragraph conflicts with its linked 2022 release; verify the release/table rather than copying that apparent typo. [G11]

### Questions to work through

- Which changes preserve meaning and which demand a new identifier?
- Does a model need per-assertion evidence, model-level evidence, or both?
- Can two independently valid component versions form an invalid combined release?
- Can removing vague or redundant terms improve the resource more than adding terms?

**Interpretive lesson:** semantic stability is an editorial commitment implemented through identifiers, change policies, validation and release engineering. Formal syntax alone does not provide it. Expert curation costs remain even when checks and imports are automated.

## 3. Uber Databook: a governed operational metadata system

### Primary reading sequence

**U1. Databook: Turning Big Data into Knowledge with Metadata at Uber**, Luyao Li and Lauren Tindal, 3 August 2018.
https://www.uber.com/au/en/blog/databook/
Read architecture and collection/storage separation. The team chose to materialize metadata rather than fetch every source on demand: source latency and source-system load made synchronous fan-out undesirable. Serving and collection processes were separated so ingestion disruption could leave stale-but-available results. Polling remained useful, while Kafka events supported freshness/lineage signals needing quicker propagation. Human descriptions supplied semantics that source extraction could not create. The central tradeoff is explicit freshness versus availability and source isolation, rather than a claim that one ingestion mechanism solves every need. [U1]

**U2. Turning Metadata Into Insights with Databook**, Sunheng Taing and Atul Gupte, 10 November 2020.
https://www.uber.com/py/en/blog/metadata-insights-databook/
Read Principles, Data Model, Event Log and Benefits. Domain experts propose schemas; a central committee reviews them and backward compatibility. Identified entity types differ from reusable value types. Generated Thrift/Protobuf/Avro definitions give a shared vocabulary. Persistent JSON records use MySQL; Kafka changes feed rebuildable derived stores, including Elasticsearch. The authors report eighteen months in production, hundreds of thousands of datasets and other assets, millions of columns, and new-entity onboarding under an hour instead of weeks. These self-reports are valuable deployment evidence but lack latency distributions, independent validation and a controlled productivity comparison. The article does not disclose Wikidata-like evidence ranks, detailed conflict-resolution policies or complete deletion semantics; do not invent them. [U2]

### Questions to work through

- Which metadata must be current immediately, and which can safely lag?
- Who owns semantic definitions across teams?
- Is an event log sufficient to rebuild a projection if schemas and transformation code also change?
- Which relationships are operational lineage and which are claims requiring independent evidence?

**Interpretive lesson:** graph-shaped information does not force a graph database as the sole persistent store. Vocabulary governance, replayable changes and access-pattern-specific projections can matter more than the storage label. This is an inference from the two engineering accounts, not a claim about current private infrastructure.

## 4. GitLab Handbook: document-level knowledge and operational ownership

### Primary reading sequence

**H1. Handbook usage**, living operational guidance.
https://handbook.gitlab.com/handbook/about/handbook-usage/
Read process changes, approval and peer review. Changes enter via merge requests; page ownership is resolved through CODEOWNERS, and significant changes receive peer review. The accountable owner decides the page’s content. This creates organizational authority and an inspectable change trail, rather than attaching a source/evidence record to every sentence. [H1]

**H2. Handbook frontmatter**, living schema documentation.
https://handbook.gitlab.com/docs/frontmatter/
Frontmatter provides a small structured envelope around prose. Controlled-document metadata is coupled to required ownership entries and approval/merge rules. The important pattern is differentiated review according to document risk rather than assuming every page requires identical governance. [H2]

**H3. Editing the Handbook**, living instructions.
https://handbook.gitlab.com/handbook/about/editing-handbook/
Read move/rename and deletion checklists. A displayed title differs from a path-derived URL. Moves require incoming-link, navigation and ownership updates plus a verified redirect; a local preview may not exercise production redirects. This is a document-specific identity migration problem. [H3]

**H4. Troubleshooting and maintenance**, living runbooks.
https://handbook.gitlab.com/handbook/about/editing-handbook/troubleshooting/
https://handbook.gitlab.com/docs/development/maintenance/
Checks distinguish newly broken links from historical debt. Maintenance instructions prioritize recurring broken destinations and keep linter configurations current. These tests catch structural failures, not incorrect policy or obsolete advice. The maintenance page includes transition-oriented language about allowing link jobs to fail, so do not generalize it into a claim that all broken links are currently blocked. [H4–H5]

**H6. Handbook Search**, last modified 11 August 2026.
https://handbook.gitlab.com/docs/search/
Public search uses Algolia; internal search uses Lunr indexing site content. The stated reason for the latter is avoiding external indexing of internal material without legal/security review. This is a concrete example of access boundaries shaping search architecture. Do not substitute the separate GitLab product-documentation search architecture for the Handbook’s. [H6]

**H7. Handbook Development and Changelog**, living operational sources.
https://handbook.gitlab.com/docs/development/
https://handbook.gitlab.com/handbook/about/changelog/
Redirect lifetimes are budgeted by context, with permanent exceptions for compliance. The changelog publishes the last hundred merge requests and included October 2026 changes when inspected, establishing active maintenance. Neither page establishes a numerical productivity improvement or an accuracy rate. [H7–H8]

### Questions to work through

- When is page-level provenance enough, and when does claim-level provenance become necessary?
- How does the system identify an accountable reviewer after a reorganization?
- Do link checks and search ranking conceal semantic staleness?
- How long should old URLs remain resolvable when the knowledge they pointed to is withdrawn?

**Interpretive lesson:** a mature document KB can achieve strong edit accountability and migration discipline while remaining semantically weak at the individual-claim level. That tradeoff can be appropriate; it is not merely an unfinished graph system.

## Side-by-side design tradeoffs

| Dimension | Wikidata/Wikibase | GO/OBO | Uber Databook | GitLab Handbook |
|---|---|---|---|---|
| Principal unit | Entity-associated assertion | Ontology term; evidenced annotation; contextual model | Typed operational asset | Owned document/page |
| Main correctness pressure | Heterogeneous public claims | Biological meaning and annotation validity | Cross-system interpretability | Current, authorized organizational guidance |
| Acceptance style | Contribution plus repair | Expert curation plus release validation | Governed integration | Reviewable document changes |
| Context | Statement qualifiers | Relations, negation, extensions, models | Source/asset relationships | Prose and document structure |
| History problem | Edits, merges, query-projection lag | Meaning change and synchronized versions | Ingestion and projection evolution | File, URL, ownership and policy evolution |
| Characteristic limit | Expressive queries become expensive | Expert review and semantic migration remain costly | Public details leave governance gaps | Structural checks do not prove prose accuracy |

Table entries are a synthesis of the case readings, not additional measured claims. Do not compare triple counts, annotation counts, asset counts and page counts as a common scale metric.

## Cross-case conclusions worth testing, not slogans

1. **Identity, evidence and serving are separate design decisions.** A stable name does not preserve meaning; a citation does not establish validity; a fast index does not establish freshness.
2. **Each simplification needs an explicit loss budget.** Best-rank projections, basic ontology editions, metadata search documents and page-level indexing each omit information. Ask which downstream question becomes unsafe after the omission.
3. **Governance must operate at the unit that can fail.** Properties, scientific terms, enterprise schema types and policy pages need different reviewers and different migration procedures.
4. **Versioning must include dependencies.** An isolated object revision may be insufficient when results depend on an ontology, annotations, imported relations, parser versions or rebuilt indexes.
5. **Maintenance is not only growth.** Obsoletion, repair, source retraction, ownership changes and query migrations are ordinary operating work. A model that cannot withdraw knowledge is incomplete.
6. **Separate three evidence levels.** Specifications establish intended semantics; runbooks establish operational mechanisms and anticipated failures; measured reports establish dated observations. None automatically proves end-user benefit.

## Compact Chinese insights

- 先学“知识以什么单位被编辑、验证和撤回”，再比较数据库产品。
- Wikidata 的 rank 是视图选择机制；GO 的 evidence code 是证据类型。二者都不应直接当作统一置信分数。
- 永久 ID 不等于永久含义；语义变化需要新概念、替代关系及迁移说明。
- 生产系统通常把编辑源、发布版本、搜索索引和查询服务分开；因此必须追踪延迟、重建能力和版本组合。
- GO 的维护实例说明，成熟知识库会主动删减和废弃概念，不能只用规模增长衡量进步。
- 文档知识库的强项是责任人、评审和历史；知识图谱的强项是细粒度关系及可计算语义。两者不是简单的先进与落后关系。
- 工程博客中的生产自报、正式规范和独立测评是三种不同证据，不应混用。

## Evidence ledger and limits

All links above are exact source URLs. Living pages were checked on 2026-10-16; a crawl date is not treated as publication date.

| Source IDs | Type/date | What the evidence supports | What it does not establish |
|---|---|---|---|
| W1–W3 | Official model/export specifications; living | Claim structure and export semantics | Reference completeness or truth |
| W4 | Community constraint documentation; living | Advisory validation design | Universal enforcement |
| W5–W6 | Access and merge documentation; living | Revision retrieval and identity repair procedures | Immutable whole-KB snapshots or fully automatic safe unmerge |
| W7 | Maintainer report, March 2023 | Explicit 30-day lag observations and known failure modes | Current latency or independent causal attribution |
| W8–W9 | Official 2025 migration docs | Split date, stated scale/rebuild pressures, required client changes | Availability of expired transitional endpoints in 2026 |
| G1–G4 | Official annotation specs/guidance; living | Required evidence/context and interpretation rules | Universal biological correctness |
| G5 | Official overview; original paper 2019 | Purpose and representation of connected activity models | Independent evaluation in this pass |
| G6–G8 | Official ontology/OBO policies; living, Principle 19 approved 2025 | Semantic identity and deprecation; relation-sensitive restrictions | Zero migration errors |
| G9–G10 | Maintainer source repositories; living | Tests, coordinated release process and human gates | Perfect schedule adherence |
| G11 | Consortium paper; online 2025-12-18 | Dated maintenance totals and model coverage | Current totals, controlled productivity gain, correction of the apparent date typo |
| U1 | Uber engineering report, 2018-08-03 | Historical collection/serving tradeoffs | Current Uber architecture |
| U2 | Uber engineering report, 2020-11-10 | Production self-reported architecture, scale and onboarding improvement | Independent benchmark; undocumented rank/conflict/deletion behavior |
| H1–H5 | Organization-maintained operational docs; living | Ownership, review, structural checks and migration process | Per-claim evidence or semantic accuracy rates |
| H6 | Search architecture doc; modified 2026-08-11 | Different public/internal indexing choices and stated reason | Search relevance benchmark |
| H7–H8 | Development guide and current changelog | Redirect policy and recent maintenance activity | Quantified business impact |

### Suggested exercise after the readings

Take one deliberately small invented example with two contradictory sources, a changed identifier, an obsolete classification and a withdrawn document. Describe how each case would preserve history, expose a default answer and inform downstream consumers. Then identify the facts that its native representation cannot express without convention or extension. This reveals the design boundary more clearly than a feature checklist.
