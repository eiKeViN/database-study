# Reliable knowledge-base lifecycle and engineering

Research date: 2026-10-16. Scope: general knowledge-base operation, without a mathematics mapping or implementation changes. This is an engineering design memo, not a benchmark of a proposed implementation. Recommendations and hypothetical examples below are our synthesis; the evidence ledger explicitly identifies what external sources establish.

## Executive conclusion

For an evidence-bearing, multi-source KB service, it is useful to distinguish (1) source artifacts or permitted references, (2) contextual claims and identity decisions, and (3) published, query-oriented projections. These are design separations, not mandatory objects or services for every KB. A document-first handbook may instead manage applicability, ownership, revision, and review through page metadata and editorial workflow, without a claim-level ontology. Retain captured artifacts only where permissions and retention rules allow. A search index, embedding collection, or readable summary is generally a projection, not the sole record of knowledge.

The central operational requirement is to make changes explainable and safely propagating. An update is incomplete until affected claims, derived answers, permission checks, and user-visible projections reach a declared state. Database atomicity does not imply index freshness; reproducible history does not authorize continued disclosure after permission revocation.

## 1. Separate representation choices from storage choices

Representation decides what the system means:

- What counts as the same entity, as opposed to a similar name or an organizational successor?
- Is an assertion a source's statement, a curator's accepted conclusion, or a machine inference?
- Which qualifiers make it applicable: time, location, jurisdiction, product edition, population, measurement method?
- What does absence mean: unknown, intentionally withheld, not applicable, or explicitly false?
- Which evidence and selection rule justify an answer?

Storage decides how this meaning is maintained and served: relational tables versus a graph, immutable objects versus update-in-place, transaction boundaries, inverted/vector indexes, partitioning, caching, replication, and retention. A graph database does not automatically solve identity, provenance, temporal semantics, or conflict. A relational system can encode these well. Conversely, an excellent storage engine cannot repair an underspecified claim.

For a service requiring explicit claim-level evidence and reproducibility, a candidate logical model contains entities; permitted source versions or references; assertions with context; evidence links; revision/supersession/retraction events; curation decisions; derivations; and publication manifests. Select only the separations required by its contracts; this is not a universal minimal model. These can initially live in one database plus object storage. Separate services are a scaling choice, not a prerequisite for maturity.

## 2. Ingestion: preserve distinctions before optimizing retrieval

Recommended pipeline:

1. Capture source identity, authorized access scope, retrieval time, source-reported publication time, media type, content digest, and immutable bytes or permitted snapshot. Distinguish a URL from the particular content retrieved there.
2. Parse into a versioned intermediate representation, preserving headings, tables, units, footnotes, page/section locations, and source spans. Record parser/OCR versions and failure flags.
3. Canonicalize reversible presentation differences, such as Unicode normalization or explicit unit conversion, while retaining the original representation. Never silently infer a timezone, merge legal entities, or collapse semantically different quantities.
4. Extract candidate claims, including negation, modality, conditions, and attribution. Preserve exact support spans, not merely a link to a long document.
5. Resolve identities and validate constraints; quarantine ambiguous or malformed candidates.
6. Commit accepted changes and their derivation/decision records, then update projections.

Failures and trade-offs:

- Deduplicating by URL loses source revisions; deduplicating by bytes alone conflates copies with different access restrictions or independent publication contexts. Content hashes should identify artifacts, not replace source identity and access policy.
- Aggressive text cleaning can turn “not eligible” into “eligible,” detach table headers, or drop an exception. Better parsing and spot review cost more but prevent precise-looking false claims.
- Retrying extraction can create duplicate support and spuriously raise confidence. Use idempotency keys derived from source version, extraction specification, and extraction unit; distinguish retry from deliberate reprocessing with a new model.
- Full ingestion before publication gives a coherent release but delays freshness. Incremental publication reduces latency while requiring an explicit partial-coverage state.

## 3. Entity resolution must be reversible

Use stable opaque entity IDs, source-native IDs with namespaces, and separately versioned aliases and match decisions. Names and URLs are mutable attributes. A changed address is usually a revision of an existing entity; a newly incorporated business with the same trade name may be a new entity.

Prefer authoritative identifiers where available; otherwise combine multiple features and retain match evidence. Probabilistic record linkage is an established implemented approach, exemplified by Splink [S8]. It does not justify treating every high score as identity certainty.

Recommended controls: high-precision automatic matching for well-understood cases, an uncertain zone for review, and explicit non-match evidence. Preserve source record IDs through merges so that an erroneous merge can be split and downstream references reassigned. Record the linkage model, thresholds, feature version, and reviewer.

A false merge can contaminate later facts, while a false split can reduce recall, duplicate results, distort counts, or hide relationships. Their relative costs are consequence-dependent: contact and permission use cases often favor conservative merging, whereas counting or fraud detection may make false splits especially costly. Thresholds should reflect these downstream consequences, not only pairwise accuracy. Transitive closure is particularly hazardous: A matching B and B matching C can combine records even when A and C conflict. Cluster-level constraints, sampled review, and reversible merge events cost storage and reviewer time but reduce irreversible contamination.

## 4. Claims, trust, and disagreement

Store competing assertions rather than overwriting them. Group potential conflicts only after aligning subject, predicate, units, temporal interval, and other qualifiers. “Office closes at 17:00” and “holiday service closes at 15:00” need not conflict.

Model source authority by domain and claim type. A retailer is authoritative about its listed offer, less so about an independent safety assessment. Separate authenticity, relevant expertise, recency, extraction confidence, and corroboration. A single universal trust score obscures these differences. Ten copied articles are not ten independent witnesses; retain source derivation/copy relationships where known.

A current-answer policy should be versioned and explainable: for example, prefer the applicable official policy revision over an older FAQ, while showing an unresolved conflict if dates or authority are ambiguous. The system may choose a preferred display value without deleting alternatives. Wikidata demonstrates separate statement qualifiers, references, and ranks [S2]; it is evidence for representational viability, not proof that a particular selection policy is correct.

Confidence must be scoped: confidence in extraction is not confidence in truth, and confidence in identity is not confidence in the predicate. Rules should allow “insufficient evidence” rather than force a winner.

## 5. Identity, revisions, and time are different axes

Use at least three identity levels: real-world entity ID, logical source/document ID, immutable artifact or claim revision ID. Revision chains preserve the fact that the same subject/document changes; hashes identify exact content but not enduring identity.

Store effective/valid time separately from system/record time. A correction learned today can apply to last month. Source publication time and retrieval time are additional metadata, not substitutes for either. Use explicit timezone, granularity, inclusive/exclusive interval conventions, and unknown boundaries; do not fabricate second-level precision for a statement known only by year.

XTDB documents the distinction between valid time and system time and supports querying them separately [S3]. This establishes a concrete implementation option; it does not mean every KB needs a bitemporal database. A simpler revision ledger suffices when only current state and coarse audit history matter. Full bitemporality adds interval handling, larger histories, more elaborate queries, and more tests. It is valuable when “what was true then?” and “what did we believe then?” are materially different questions.

Historical reproducibility also requires recording policy/schema/model versions. Re-running an old query over old facts with today's conflict-selection rules can produce a different answer even if the data snapshot is unchanged.

## 6. Provenance and derivation are operational infrastructure

For each accepted claim, retain links to source version and exact evidence span; extraction activity and software/model version; normalization/identity decisions; curator; and status transitions. For a derived result, retain the specific input claim revisions and transformation specification. PROV-O supplies interoperable concepts for entities, activities, agents, derivation, revision, attribution, and invalidation [S1]. It does not itself prove truth or implement retraction propagation.

Distinguish evidence provenance (“why might I believe this?”), computational lineage (“which inputs and code produced this?”), and decision history (“who accepted or overrode it?”). A citation list alone covers none of these completely.

Maintain reverse dependencies so a source correction can identify affected assertions, summaries, and indexes. Retraction is not always cascading deletion: if a conclusion has two genuinely independent derivations and one remains valid, retain the conclusion with reduced support. This requires representing alternative supporting sets; merely storing an undifferentiated list of parent documents can over-delete or under-delete. Start with conservative invalidation and recomputation if fine-grained support tracking is too costly.

## 7. Transactions and coherent publication

Recommended commit unit: the new claim revisions, source references, identity changes, retractions, and an outbox/change event belong in one authoritative transaction. Otherwise a crash can publish a claim without evidence or update the database without notifying the indexer. Consumers must be idempotent; a retry is expected, not exceptional.

For multi-store publication, build a release manifest containing the authoritative commit or source-offset vector, schema version, extraction/curation policy versions, index generations, validation report, and publication time. Build required projections, validate, then atomically switch a release pointer. Readers pin that release for the duration of a query. This is a design recommendation analogous to, but broader than, Iceberg's table snapshot/metadata publication mechanism [S4]. Iceberg does not automatically make a graph, text index, and vector index mutually atomic.

Trade-off: coherent release publication increases latency and may temporarily duplicate storage. Independently updating projections is cheaper and fresher but can yield a mixed answer assembled from mutually inconsistent revisions. Support both only with explicit serving contracts: strict snapshot, bounded-stale, or current-best-effort. Never label best-effort “consistent” without saying consistent with what.

Concurrent curators should submit changes against a base revision, with conflict detection rather than last-writer-wins. Retrying a stale update must revalidate assumptions, especially after identity merges or schema changes.

## 8. Incremental maintenance and freshness

Ingestion progress, authoritative commit, projection update, and search visibility are separate milestones. Expose watermarks/offsets for each, plus last successful refresh, lag, and failed partitions. Multi-source KBs usually have a vector of source progress, not one globally meaningful “last updated” timestamp.

Debezium's PostgreSQL connector documents consistent initial capture followed by committed row changes and snapshot/stream collision handling [S5]. Elastic documents that indexed changes become searchable after refresh [S6]. Materialize documents incrementally maintained query results, including insert/update/delete processing [S7]. These are concrete mechanisms with different boundaries; none makes a fact fresh if its upstream source has not changed or has not been read.

For derived results, invalidate by dependency and input revision. Incremental joins or rules can have large fan-out; one entity merge or schema reinterpretation may require a broad rebuild. A practical strategy uses incremental updates for ordinary changes, full recomputation for semantic migrations or highly connected changes, and periodic reconciliation to detect drift.

Materialization trades write-time compute and retained state for read latency. More indexes increase memory, maintenance costs, and deletion work. Keep a small set of measured high-value projections rather than materializing every possible inference. In particular, embedding freshness includes source content, chunking, model version, and filter metadata. A vector that still exists after its source is revoked is not safe merely because the main database is correct.

## 9. Schema evolution and deprecation

Separate representational compatibility from semantic compatibility. Renaming a field is unlike changing its meaning. Replacing “price” with “price excluding tax in local currency” requires a new predicate or an explicit migration with units and scope; a successful SQL migration proves only structural success.

Use stable field/predicate IDs, versioned schemas and validation rules, declared migration mappings, and a deprecation window. Iceberg's field IDs and schema evolution demonstrate how physical identity can survive renames [S4]. SHACL supplies explicit graph validation constraints [S9]. Neither determines whether a new domain meaning is scientifically or operationally justified.

Prefer expand-migrate-contract: add the new representation, backfill with provenance, compare old/new query behavior, update consumers, then retire the old contract. Preserve lossy mappings as lossy, and leave unknown values unknown. Dual writing risks divergence and requires reconciliation; read adapters avoid duplicate writes but add query complexity. Retaining every historical schema indefinitely increases operational burden, so define supported read/migration windows while preserving archival interpretability where needed.

## 10. Permissions and deletion are part of semantics

Apply authorization to raw documents, claims, evidence links, derived summaries, search results, citations, exports, logs, and caches. A public conclusion does not authorize disclosure of a confidential supporting source. Conversely, deriving a summary from restricted text does not make it public.

A conservative derived-artifact policy requires permission for all inputs actually used; where the same fact is independently supported by public evidence, generate a distinct public derivation. This is safer than inheriting the least restrictive label. Tenant boundaries should affect entity resolution too: a shared global index can reveal the existence of private entities even if their content is hidden.

Permission checks must be current enough to respect revocations and causal order. Zanzibar's 2019 paper demonstrates authorization designed around causally ordered ACL/content changes at production scale [S10]. Its reported performance is historical evidence for that deployment, not a guarantee for a new KB.

Recommendation: filter candidates before model context construction, verify permission at delivery, invalidate entitlement-sensitive caches, and maintain an urgent deny/tombstone overlay while slower index cleanup completes. This adds request latency and availability dependencies. Historical-data queries must still obey current access rights unless a separately authorized audit policy permits otherwise; replaying old ACLs is not automatically legitimate.

Distinguish source disappearance, source retraction, permission revocation, and required erasure. A vanished URL does not falsify a claim. A retraction changes evidential status. Revocation stops serving content. Erasure may require removing retained copies and derived material, compromising perfect replay. Retain only permitted audit metadata and state the resulting reproducibility limit. “Immutable history” must not become an excuse for retaining prohibited content forever.

Treat retrieved text as untrusted data: instructions inside sources must not grant ingestion agents permission to change the KB, access other sources, or exfiltrate information. Extraction credentials should not include publication or permission-administration authority.

## 11. Human and agent curation, audit, and reproducibility

Agents should propose claims and changes within a typed change set. Human review is most valuable for high-blast-radius merges, authoritative-source conflicts, sensitive publication, schema changes, and retractions; routine high-confidence changes can be sampled. Review queues should show evidence spans, before/after context, affected dependents, and the reason the proposal was flagged. Asking reviewers to inspect whole documents for every assertion creates a bottleneck and encourages rubber-stamping.

Record proposer, reviewer, decision, rationale, timestamps, base revision, and resulting revision. Distinguish factual status from workflow status: reviewed is not synonymous with true. Maintain separation of duties for sensitive changes, with rollback or compensating change events.

For an auditable answer, capture query, release, authorized evidence IDs, retrieval configuration, model identifier, prompt/template version, output, and selection policy. Exact regeneration may remain impossible with nondeterministic or unavailable hosted models; retaining the actual historical output and inputs establishes what happened without falsely promising bit-for-bit reproducibility. Logs themselves need access controls and retention limits.

## 12. Three ordinary end-to-end examples

### A. A shop changes its opening hours

On April 10 an official notice says weekday closing moved from 18:00 to 17:00 effective April 1. A directory still says 18:00. Store the new source revision and a claim valid from April 1, recorded April 10. Preserve the older belief and the directory's competing assertion. The current view prefers the applicable official notice; an audit of an answer given April 5 can reproduce the then-known 18:00 answer. If the notice specifies one branch or a holiday exception, that scope belongs in the claim, not just in prose hidden behind a link. Avoid a global overwrite of all branches.

Trade-off: two temporal axes and context increase query complexity, but prevent incorrectly describing the April 5 answer as based on evidence not yet received.

### B. Two employees share the name Alex Chen

An old team page names Alex Chen as the support contact; a new HR record identifies a different Alex Chen as finance lead. Name-only merging incorrectly combines roles and permissions. Stable employee IDs keep the people separate. If a faulty merge already happened, a recorded merge decision and preserved source IDs allow a split, followed by re-evaluation of role claims, summaries, contact suggestions, and search aliases. An official correction retroactive to a transfer date is recorded as a correction, not as proof the system knew it earlier.

Trade-off: conservative matching leaves duplicates and review work, but avoids high-cost contamination and access mistakes.

### C. A supplier guide disappears or is withdrawn

A KB says a device works with Adapter X because guide version 3 says so. If the URL returns 404, mark source availability; do not silently convert compatibility to false. If the supplier retracts that passage, invalidate its support and recompute dependent recommendations. Independent valid test evidence may still support compatibility. If access to the guide is revoked, block its text, snippets, vectors, and summaries at serving time while removing stale projections. If retention is prohibited, remove captured content too and mark old answers as not fully replayable.

Trade-off: dependency-aware recomputation preserves justified facts but is more complex than delete-all. A deny overlay reduces disclosure latency at the cost of extra checks. Source-removal tests must include cached answers and exported artifacts under the system's control, not only the primary record.

## 13. Acceptance tests and operational measurements

Test the intended guarantees, not only retrieval relevance:

- Replay duplicate and out-of-order events: one accepted revision, no duplicated evidence, no resurrected deletion.
- Crash between authoritative commit and projection update: change is eventually applied once in effect, with measurable lag.
- Concurrent edits and a conflicting merge: stale assumptions trigger review or revalidation.
- Retroactive correction: historical belief and corrected historical truth remain distinguishable.
- Revoke access during generation: no restricted text or citation is delivered.
- Remove one of two independent supports: supported conclusion remains; unsupported derivatives disappear or are flagged.
- Migrate a predicate's meaning: unsupported translations remain unknown, and old queries receive a documented compatibility behavior.
- Rebuild from retained inputs: compare semantic query results and evidence coverage; measure discrepancies rather than assuming replay correctness.

Track false merges/splits on labeled samples, unsupported-claim rate, evidence-span correctness, conflict-resolution review accuracy, deletion/revocation propagation latency, projection-lag percentiles, orphaned derivations, rebuild cost, failed-source coverage, and reviewer effort. Set thresholds by consequence and use case; no universal freshness or accuracy target is defensible. A mature KB also reports what it could not ingest or validate, so apparent completeness is not mistaken for actual coverage.

## Chinese summary

成熟知识库的核心不是“把资料放进图数据库或向量库”，而是把来源、带语境的断言、身份判断、审核决定和面向查询的派生结果分开管理，并让每次更新能够解释、追踪和传播。

1. 先保留证据再规范化：来源地址不等于来源版本；原文、表格语境、单位、否定词和精确证据位置不能在清洗中丢失。
2. 实体身份不能由名字决定。合并要有依据、阈值和可撤销记录；错误合并与错误拆分的相对代价取决于用途，应按下游后果设定阈值。
3. 冲突应在时间、地点、适用对象等条件对齐后判断。可信度、抽取置信度和身份匹配置信度不是同一回事。
4. 区分事实生效时间与系统获知时间，才能同时回答“当时真实情况是什么”和“当时系统知道什么”。
5. 数据库提交、索引更新、搜索可见和权限生效是不同边界。用版本清单和进度水位定义一致性，不能用一个“最后更新时间”掩盖滞后。
6. 来源失效、内容撤回、权限撤销和必须删除是四种不同事件。需要依赖追踪、支持集重算、及时拦截与后续清理。
7. 模式升级不仅是字段改名，还可能改变语义；必须保留迁移依据、兼容窗口和无法无损转换的情况。
8. 人工审核应集中在高影响变更；代理负责可审查的建议。审计能够说明当时发生了什么，但不能无条件保证旧模型输出可精确重现。

这些机制在正确性、延迟、存储、计算和维护复杂度之间有实际代价。应按风险逐步引入，不必从第一天就部署复杂的多数据库架构。

## Source and evidence ledger

All URLs below were consulted on 2026-10-16. Living documentation is identified as such; it should be pinned to a release or archived capture for implementation decisions. No external source is claimed to validate the complete architecture proposed here.

- S1. W3C, PROV-O, Recommendation 2013-04-30. https://www.w3.org/TR/2013/REC-prov-o-20130430/ (also https://www.w3.org/TR/prov-o/). Source establishes vocabulary for entities, activities, agents, derivation, revision, attribution, and invalidation. Evidence type: normative interoperability model. Inference: use this conceptual separation for reproducible KB pipelines. It does not establish factual reliability, permission propagation, or automatic truth maintenance.
- S2. Wikidata Help:Ranking, living help page, https://www.wikidata.org/wiki/Help:Ranking ; Help:Qualifiers, https://www.wikidata.org/wiki/Help:Qualifiers/en-gb . Source establishes normal/preferred/deprecated ranks and contextual qualifiers. Evidence type: deployed-system data-model guidance. Inference: preserve alternatives and independently select a display preference. No numerical accuracy claim is inferred.
- S3. XTDB, Time in XTDB, current living documentation, https://docs.xtdb.com/about/time-in-xtdb.html . Source explains system-time and valid-time distinctions and separate historical questions. Evidence type: documented database capability, not independent performance evaluation. Inference: use dual temporal semantics when delayed or retroactive corrections matter. Product documentation version not pinned; do not confuse with XTDB v1 documentation.
- S4. Apache Iceberg specification, living specification consulted as published on research date, https://iceberg.apache.org/spec/ . Sections Overview, Optimistic Concurrency, Schema Evolution, Identifier Field IDs, Table Metadata, and Snapshot Retention Policy. Source establishes atomic metadata replacement, committed snapshots, optimistic validation, stable schema field IDs, and retention-dependent physical deletion. It explicitly leaves row-identifier uniqueness enforcement to engines/providers. Evidence type: storage format semantics. Inference: release manifests and stable semantic IDs are useful patterns; cross-store atomicity and semantic migrations remain application responsibilities. The page spans format versions including v4 material; no claim that every engine supports every described version.
- S5. Debezium PostgreSQL connector, stable URL serving documentation labeled 3.7 on research date, https://debezium.io/documentation/reference/stable/connectors/postgresql.html . Source describes snapshot-to-log capture, committed change events, incremental snapshot collision handling, and possible duplicates in documented circumstances. Evidence type: implementation/operational documentation. Inference: maintain idempotent consumers and explicit capture boundaries. Do not extrapolate a universal exactly-once end-to-end guarantee.
- S6. Elastic, Near real-time search, living documentation, https://www.elastic.co/docs/manage-data/data-store/near-real-time-search . Source explains refresh-mediated visibility, with a default periodic refresh for recently searched indices. Evidence type: documented serving behavior. Inference: acknowledgment of a write is not sufficient evidence of search visibility; explicit visibility contracts are needed. Do not treat the described default interval as a universal freshness SLA.
- S7. Materialize, Views and Arrangements, living documentation, https://materialize.com/docs/fundamentals/concepts/views/ and https://materialize.com/docs/fundamentals/concepts/arrangements/ . Sources document incrementally maintained indexed/materialized views, durability differences, and data/time/diff update streams handling insertions and deletions. Evidence type: implemented capability. Inference: derivatives can be maintained incrementally, but costs and source lag must be measured. “Fresh” relative to ingested inputs is not proof of completeness or real-world currency.
- S8. Splink, Probabilistic vs Deterministic linkage, living documentation, https://moj-analytical-services.github.io/splink/topic_guides/theory/probabilistic_vs_deterministic.html ; project overview https://moj-analytical-services.github.io/splink/index.html . Source describes probabilistic entity resolution using the Fellegi–Sunter model and linkage without shared unique identifiers. Evidence type: implemented method. Inference: support evidence-based matching and uncertain decisions; the source does not establish a safe threshold for this KB or make pairwise linkage equivalent to verified global identity.
- S9. W3C, Shapes Constraint Language, Recommendation 2017-07-20, https://www.w3.org/TR/2017/REC-shacl-20170720/ ; publication announcement https://www.w3.org/news/2017/shapes-constraint-language-shacl-is-now-a-w3c-recommendation/ . Source establishes RDF graph validation against explicit shapes. Evidence type: normative validation language. Inference: version constraints alongside data and migrations. Passing constraints does not establish source truth or domain adequacy.
- S10. Pang et al., Zanzibar: Google's Consistent, Global Authorization System, USENIX ATC 2019, pp. 33–46, July 2019. https://www.usenix.org/conference/atc19/presentation/pang ; paper https://www.usenix.org/system/files/atc19-pang.pdf . Source reports causally ordered authorization amid ACL/content changes and its then-production deployment: trillions of ACLs, millions of authorization requests per second, under 10 ms 95th-percentile latency, above 99.999% availability over three years. Evidence type: authors' production report. Inference: authorization consistency deserves its own contract; these historical numbers are not present-day claims or expected performance for a new design.

## Priority addendum: five essential learning objects, in order

These are a curated lifecycle track, not a recommendation to read every linked source in the ledger. Learning outcomes and exercises are our recommendations; exact titles, editions, and covered topics were checked against primary pages on 2026-10-16.

### 1. The architectural spine: Designing Data-Intensive Applications, second edition

Martin Kleppmann and Chris Riccomini, O'Reilly, February 2026, ISBN shown by publisher: 9781098119058. Verified publisher contents: https://www.oreilly.com/library/view/designing-data-intensive-applications/9781098119058/cover.html . This is the published second edition, not an early-release draft or the 2017 first edition. Chapter numbering differs from the first edition.

Read first: chapter 1, especially systems of record versus derived data; chapter 5, Encoding and Evolution; chapter 12, especially Change Data Capture, State/Streams/Immutability, Reasoning About Time, and Fault Tolerance; chapter 13, especially Observing Derived State, the end-to-end argument, and Timeliness and Integrity. Use chapter 4's materialized-view, full-text, and embedding sections as reference.

Learning purpose: understand which guarantees belong to authoritative storage, update transport, and derived serving, before choosing a graph or vector product. Output: draw one ingestion-to-answer pipeline, label every independently failing boundary, and define what “updated” means at each boundary.

Omission: this is not a complete treatment of evidence provenance, entity identity, or factual conflict adjudication. Do not read it as a turnkey KB design. Publisher metadata and table of contents were inspected; this research pass did not read the entire book.

### 2. A minimal provenance model: W3C PROV-O

Recommendation dated 2013-04-30: https://www.w3.org/TR/2013/REC-prov-o-20130430/ . Read §3.1 Starting Point Terms, §3.2 Expanded Terms, and §3.3 Qualified Terms; use §4 as a lookup, not a cover-to-cover exercise.

Learning purpose: distinguish the artifact, the activity that transformed it, and the responsible agent; distinguish revision, derivation, attribution, and invalidation. Output: describe one source correction and the affected summary using explicit source-version and transformation links.

Omission: PROV-O does not rank truth, enforce access, or automatically retract derived conclusions. Start with a small useful subset; implementing the entire vocabulary before a concrete use case adds complexity without assured benefit.

### 3. Temporal semantics made concrete: XTDB, Time in XTDB

Current official living documentation: https://docs.xtdb.com/about/time-in-xtdb.html . Read the System time and Valid time sections, then Bitemporality in XTDB and its three historical-question categories. Treat the documentation as an executable-model tutorial rather than independent vendor-performance evidence.

Learning purpose: stop conflating effective dates, retrieval dates, and database-record dates. Output: represent a shop-hours correction learned April 10 but effective April 1; show the difference between a corrected April 5 answer and the answer actually supported by the database on April 5.

Omission: time dimensions do not themselves decide which contradictory source to prefer. Historical access also remains governed by authorization policy. Exact implementation syntax and defaults should be pinned to the XTDB release chosen later.

### 4. A production consistency case study: Zanzibar

Pang et al., USENIX ATC 2019, pp. 33–46. Landing page: https://www.usenix.org/conference/atc19/presentation/pang . Open paper: https://www.usenix.org/system/files/atc19-pang.pdf . Read §2 for the model/consistency contract, §3.1 Storage and §3.2 Serving for the mechanisms, and §4 for measured behavior and measurement definitions.

Learning purpose: see why permissions must be causally coordinated with content and why caching cannot be separated from correctness. Output: trace “remove colleague access, then add confidential content” and identify why an old entitlement cache is unsafe.

Omission: this is one historical Google deployment, not a modern off-the-shelf KB recommendation. The published workload and metrics must not be extrapolated to a new service. It addresses authorization, not truth or extraction quality.

### 5. Frontier continuation: DBSP and automatic incremental view maintenance

Start with Budiu et al., PVLDB 16(7), 1601–1614, 2023: https://www.vldb.org/pvldb/vol16/p1601-budiu.pdf . Read §1, §3, §4 (especially §4.4 costs), then §6 for recursive-query maintenance. The paper explains general incrementalization rather than requiring hand-written maintenance rules for each query; its cost discussion also exposes retained-state and planning complications.

Then consult the distinct extended journal article, published 2025-05-08: Budiu et al., The VLDB Journal 34, article 39, https://doi.org/10.1007/s00778-025-00922-y . Publisher search metadata verified title/date and identifies this as a later version; full journal text was not inspected in this pass, so no exact journal-section mapping or claims about its additions are asserted. Do not relabel the 2023 result as a new 2025 discovery.

Learning purpose: understand the research frontier behind maintaining derived results after inserts, updates, and deletions. Output: explain how deleting one input affects an aggregate or recursive derived relation, and identify when retained state or change fan-out makes recomputation preferable.

Omission: this does not solve semantic truth maintenance, source credibility, permissions, or arbitrary model-generated summaries. “Incrementalizable” is not a blanket performance guarantee.

### What to postpone

Use Iceberg's Overview/Optimistic Concurrency/Schema Evolution sections only when studying snapshot publication and stable physical field IDs; use Debezium's Snapshots/Streaming changes sections when building CDC; use Elastic's near-real-time documentation when specifying search freshness. These are excellent engineering references but poor starting points for the conceptual lifecycle. Splink becomes essential when actual identity-linkage data and error costs are available. SHACL becomes essential when a concrete graph contract needs validation. This order avoids mistaking a product's mechanism for the complete knowledge model.
