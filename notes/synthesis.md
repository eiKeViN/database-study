# Reliable knowledge and database design: a first research map

Research cut: 16 October 2026. A bounded synthesis of the six accompanying research memos, not an exhaustive survey, full-textbook reading claim, implemented system or benchmark. No product stack, repository schema or mathematics application is selected. Source observations, interpretation and unanswered questions are distinguished below.

## 中文执行摘要

第一轮研究的主要认识是：成熟系统的可靠性来自几种不同的承诺，而不是一种数据库或一种“知识图谱架构”。

- **数据库正确性**：合法状态、引用关系、并发更新和故障恢复是否受控。规范化能处理特定依赖下的更新异常，不能证明记录符合现实。
- **知识语义与证据**：记录的是谁的说法、在什么条件和时间成立、如何支持、何时应保留分歧。来源链接、置信分数和逻辑一致性都不能单独证明真实性。
- **编辑治理**：谁能定义概念、合并实体、批准修改和撤回内容；改变含义时如何通知使用者。Wikidata 的开放协作、GO 的专家发布和 GitLab 的文档负责人机制面对的是不同问题。
- **发布与检索**：查询看到哪个版本、哪些信息被简化、落后多久、权限撤销是否已生效。主库提交成功，不等于搜索结果和生成答案已经更新。

这些是分析维度，不要求建立四套服务。文档系统可以把许多责任放在页面、元数据和评审流程里；需要逐条证据的系统才可能值得承担细粒度断言和依赖追踪的成本。

成熟实践已经提供约束、版本、来源记录、增量更新和人工审核等工具。尚缺充分证据的是：在持续变化、来源冲突和模型生成内容共同存在时，如何以可接受成本稳定改善用户结果。下一步应围绕明确失败案例开展有限深读与验证，而不是继续扩充工具清单。

## 1. Four contracts, not four mandatory services

### Stored-state and execution correctness

**Source view.** Codd's motivation includes independence from physical organization and access paths. Chen distinguishes conceptual entities and relationships from their representation. Modern normalization theory gives conditional guarantees about redundancy, reconstruction and dependencies. The lending lesson demonstrates a lossless, dependency-preserving BCNF decomposition under its stated dependencies; its training example illustrates the genuine BCNF/dependency-preservation tradeoff. [Codd 1970](https://doi.org/10.1145/362384.362685), [Chen 1976](https://doi.org/10.1145/320434.320440), [Database System Concepts chapter 7](https://www.db-book.com/slides-dir/PDF-dir/ch7.pdf).

**Interpretation.** Conceptual meaning, logical constraints and physical access deserve separate decisions. A key or transaction can protect represented state without proving correct entity identification, accurate sources or complete event reporting. A snapshot dependency that each item determines one model does not prohibit changing that model between snapshots. The modeling proofs establish no performance result.

### Knowledge meaning, evidence and conclusions

**Source view.** Davis and colleagues connect representation to world commitments, reasoning, computational utility and human expression. Gruber treats ontology as a shared vocabulary/commitment interface, leaving consensus problems unresolved. RDFS domain/range support inference rather than simply rejecting misplaced fields. OWL's open-world interpretation differs from record-completeness checks. SHACL validates a selected graph against shapes. PROV-O expresses provenance without proving source truth or implementing automatic retraction. [Davis et al. 1993](https://groups.csail.mit.edu/medg/ftp/psz/k-rep.html), [Gruber 1993](https://tomgruber.org/writing/ontolingua-kaj-1993.pdf), [RDFS](https://www.w3.org/TR/rdf-schema/), [OWL Primer](https://www.w3.org/TR/owl2-primer/), [SHACL 2017](https://www.w3.org/TR/2017/REC-shacl-20170720/), [PROV-O 2013](https://www.w3.org/TR/2013/REC-prov-o-20130430/).

**Interpretation.** “Valid input,” “entailed conclusion,” “supported claim” and “true statement about the world” are distinct assessments. An application may combine them; none replaces the others. Representation must distinguish entity from label, occurrence from enduring relation, source assertion from endorsed conclusion, and missing from false.

### Editorial authority and governance

**Source view.** Wikidata permits heterogeneous statements and advisory constraints. GO combines evidence-bearing annotations, semantic-change policy and coordinated release processes. GitLab uses accountable document owners and reviewable changes. [Wikidata constraints](https://www.wikidata.org/wiki/Help:Property_constraints_portal), [GO annotations](https://www.geneontology.org/docs/go-annotations/), [GO pipeline](https://github.com/geneontology/pipeline), [Handbook usage](https://handbook.gitlab.com/handbook/about/handbook-usage/).

**Interpretation.** Someone still decides which distinctions matter, which sources are relevant and who may approve a repair. These cases face different contribution models and consequences. Page-level governance is not merely an unfinished claim-level knowledge graph.

### Published retrieval and answers

**Source view.** Wikibase's full statement representation differs from simplified best-rank direct assertions. Uber's historical Databook accounts separate collection, persistence and serving indexes. GitLab's search documentation distinguishes public and internal indexing for disclosure reasons. [Wikibase RDF format](https://www.mediawiki.org/wiki/Wikibase/Indexing/RDF_Dump_Format), [Uber 2018](https://www.uber.com/au/en/blog/databook/), [Uber 2020](https://www.uber.com/py/en/blog/metadata-insights-databook/), [Handbook search](https://handbook.gitlab.com/docs/search/).

**Interpretation.** Every published view has an information-loss budget, freshness boundary, audience and intended workload. A search index or generated summary may be derived from accepted information without being an adequate sole record of it. This does not require every system to deploy several databases.

## 2. What the mature cases establish, and what they do not

### Wikidata: plural assertions and deliberate projections

Qualifiers, references and ranks separate content from default display preference. Rank is not a calibrated probability; a reference field does not guarantee adequate evidence. The March 2023 maintainer report describes the preceding 30 days: update latency below ten minutes 98% of the time and about two minutes at the median, alongside query timeouts and divergence. These are historical observations, not a current SLA. The documented 9 May 2025 graph split changed client queries across partitions. Expressive authoring does not remove serving and rebuild limits. [Primer](https://www.mediawiki.org/wiki/Wikibase/DataModel/Primer), [March 2023 report](https://www.wikidata.org/wiki/Wikidata:SPARQL_query_service/WDQS_backend_update/March_2023_scaling_update), [graph split](https://www.wikidata.org/wiki/Wikidata:SPARQL_query_service/WDQS_graph_split).

### GO/OBO: persistent identifiers need semantic-change policy

Evidence codes characterize support methods rather than a universal confidence hierarchy. GO preserves obsolete identifiers and explanations while removing active relationships. OBO Principle 19 requires a new identifier for substantially changed referents. Coordinated releases address compatibility between ontology and annotation versions. The documented mechanisms establish maintenance practice, not zero migration errors or universal scientific correctness. [Evidence guidance](https://geneontology.org/docs/guide-go-evidence-codes/), [term stability](https://obofoundry.org/principles/fp-019-term-stability.html), [ontology documentation](https://www.geneontology.org/docs/ontology-documentation/), [pipeline](https://github.com/geneontology/pipeline).

### Uber Databook: governance and separate serving paths

The 2018 account explains materializing metadata to avoid source fan-out latency and load, permitting stale-but-available serving during collection disruption. The 10 November 2020 account describes reviewed schemas, persisted records, a change log and derived indexes. It reports eighteen months in production and new-entity onboarding in under an hour rather than weeks. This is a dated author-reported improvement without a controlled comparison or latency distribution. It does not establish current 2026 architecture, complete deletion semantics or a universal conflict policy. [2018 account](https://www.uber.com/au/en/blog/databook/), [2020 account](https://www.uber.com/py/en/blog/metadata-insights-databook/).

### GitLab Handbook: page-level knowledge can be mature

Ownership, review, redirects and link maintenance address real failure modes without formalizing every sentence. Structural tests cannot establish whether advice remains substantively correct. No quantified productivity or accuracy gain follows from the inspected runbooks. The case helps identify when fine-grained claim modeling may not justify its cost. [Usage](https://handbook.gitlab.com/handbook/about/handbook-usage/), [editing and migration](https://handbook.gitlab.com/handbook/about/editing-handbook/).

These systems cannot be ranked by triples, annotations, assets or pages: their units, users, workloads and correctness pressures differ.

## 3. Seven actionable design questions

### 1. What can be independently wrong, changed or withdrawn?

Physical objects, recurring events, source versions, qualified assertions and policy pages have different lifecycles. Fine-grained claims enable selective correction but cost extraction, identity management and editorial effort. Page ownership is economical when the whole page is the appropriate review unit.

**Evidence:** Chen, Wikibase statements and Handbook governance support different legitimate units. **Decision:** define identity-preserving changes versus new objects or versions. **Check:** renames, repeat events and a mistaken merge must not collapse distinct facts.

### 2. Which rules reject input, infer facts or flag review?

Keys, foreign keys, SHACL constraints, ontology axioms and community warnings are not interchangeable. A strict transaction record and a body of integrated claims may need different acceptance policies.

**Evidence:** PostgreSQL documents enforced constraints, Wikidata advisory constraints, and W3C distinct inference/validation contracts. **Decision:** assign each rule a scope, consequence and exception process. **Check:** missing, conflicting and exceptional records receive intended outcomes rather than invented “repairs.” [PostgreSQL 18 constraints](https://www.postgresql.org/docs/18/ddl-constraints.html).

### 3. What context must survive for the intended questions?

Time, branch, edition, units, negation and attribution can be part of meaning. Align these before declaring conflict. A preferred answer can coexist with alternatives.

**Evidence:** Wikibase qualifiers and GO annotation fields make context operational. **Contested choice:** richer assertions versus source-first prose retrieval; this review establishes no universal winner. **Decision:** list questions made unsafe by each simplification. **Check:** paired examples differing in one qualifier produce appropriately different answers. [GAF 2.2](https://geneontology.org/docs/go-annotation-file-gaf-format-2.2/).

### 4. Who may resolve disagreement or change meaning?

Separate extraction confidence, identity confidence, source relevance, review status and truth. Copied sources are not necessarily independent corroboration. False-merge versus false-split costs depend on downstream consequences.

**Evidence:** GO, Wikidata and Databook place authority at different levels; ranks and evidence codes are not universal confidence scores. **Decision:** identify reviewers, escalation triggers and reversible decisions. **Check:** verified useful changes and prevented downstream errors per expert hour, including rejected proposals and repair effort. No reviewed source proves an optimal review-allocation strategy.

### 5. What must “updated” mean to readers?

Distinguish effective time, publication, capture, authoritative commit and query visibility. Corrected historical information differs from what the system knew then. Coherent releases can delay freshness; independent refresh can expose mixed versions.

**Evidence:** GO releases, Uber collection/serving separation and documented search refresh behavior. **Decision:** define version/freshness contracts and permitted lag. **Check:** delayed corrections and failed projections remain distinguishable from fully propagated updates. [Search visibility](https://www.elastic.co/docs/manage-data/data-store/near-real-time-search), [temporal distinctions](https://docs.xtdb.com/about/time-in-xtdb.html).

### 6. What survives retraction, revocation and erasure?

A missing URL concerns availability; retraction changes support; revocation changes permission; erasure limits retention. These are neither uniformly “false” nor uniformly “delete everything.”

**Evidence:** PROV-O supplies vocabulary; Zanzibar demonstrates content/permission consistency. Neither solves end-to-end deletion through generated summaries. **Decision:** state permitted retention and how serving is blocked while cleanup completes. **Check:** remove one of two independent supports, revoke access during generation, and inspect caches and derivatives. [Zanzibar 2019](https://www.usenix.org/system/files/atc19-pang.pdf).

### 7. What would demonstrate useful improvement?

Separate constraint adherence, world correctness, context, retrieval sufficiency, answer support, maintenance cost, user outcomes and access compliance. Unknown denominators prevent absolute completeness claims.

**Evidence:** Wikidata quality research shows removal is an imperfect error proxy; ALCE separates answers and citations; EMERGE separates maintenance operations. **Decision:** a small evaluation contract for steady-state queries and changes. **Check:** temporally held-out lifecycle replay and verified user tasks under disclosed budgets. Access is a hard condition, not a score averaged against fluency. [Wikidata study](https://arxiv.org/abs/2107.00156), [ALCE](https://aclanthology.org/2023.emnlp-main.398/), [EMERGE v2](https://arxiv.org/html/2507.03617v2).

## 4. Two ordinary end-to-end examples

These are invented illustrations and proposed acceptance tests, not implemented systems or observed results.

### A. Conflicting branch hours, late correction and removal

1. A bakery has North and South branches. On April 5, the evidence available to the system says North closes at 18:00 on weekdays. A directory repeats it. Branch identity is separate from the business name.
2. On April 10, an official notice says North's closing time became 17:00 on April 1, excluding public holidays. Retain the revision where allowed, its exact support and scope. Effective time is April 1; the system learned it April 10.
3. North's ordinary weekday values conflict; South and holidays fall outside the notice. An explicit editorial policy may prefer the applicable official notice. That preference does not follow from a key constraint or “newest wins.” If authenticity or applicability is unclear, preserve uncertainty.
4. After acceptance and publication, the current ordinary-weekday answer is 17:00. A stale index showing 18:00 is a propagation issue or declared lag. An April 5 question must distinguish corrected historical hours from the answer supportable on April 5. The latter cannot use future evidence.
5. If the URL disappears, mark availability; do not infer the hours false. If the notice is retracted, withdraw its support and reassess. Independent valid evidence may sustain the claim. If content must be erased, remove prohibited copies and derivatives; retain only permitted audit metadata and admit limited historical replay.
6. Test that South and holidays do not inherit the change, answers cite applicable evidence, retractions reach summaries/caches, and historical queries distinguish effective from known time.

**Tradeoff:** temporal/contextual detail enables safer answers but raises representation and review costs. A simpler directory may choose coarse history, provided it does not promise unsupported historical answers.

### B. An evolving equipment guide reaches indexes and answers

1. A workshop's guide revision 3 says Model P supports Adapter X. Parsing preserves section context and table headers. Accepted evidence feeds keyword search, perhaps semantic search, and a help summary. These are possible projections, not a required stack.
2. Revision 4 says only hardware revision B supports X; revision A requires Y. Dropping the hardware qualifier makes even perfectly synchronized indexes wrong. Meaning must be repaired before faster propagation helps.
3. Distinguish product family, hardware revision and physical item. Existing item records may not reveal hardware revision; leave it unknown rather than defaulting to B. Review accepts the qualified claim and supersedes revision 3's blanket recommendation for current use.
4. Track affected search documents and summaries. A coordinated-release option publishes required derivatives together; an independent-refresh option declares lag or temporarily withholds affected answers. An index write acknowledgment alone does not establish search visibility.
5. “Does my Model P support X?” now requires hardware revision, or an explanation of both cases. An unconditional yes citing revision 4 still fails contextual fidelity. Cached answers need dependency and audience checks.
6. Reprocessing after a parser change differs from retry duplicates. Rebuilds require retained authorized inputs and transformation versions, not merely the existence of a log. If revision 4 becomes restricted, block its text and restricted derivatives while cleaning stale projections. Public evidence may justify a separate public derivation, not automatic reuse of a restricted summary.
7. Test lost table headers, duplicate/out-of-order events, a failed projection, rebuild, and revocation during generation. Measure correct qualified answers and propagation delay rather than ingestion success alone.

**Tradeoff:** lineage enables selective repair, but lossy generated summaries may not retain precise support dependencies. Conservative invalidation and regeneration can be safer than falsely precise incremental edits.

## 5. Evidence strength and empirical limits

**Mature, bounded foundations:** conditional normalization results, version-pinned constraint behavior, normative representation/validation contracts, implemented versioning/indexing mechanisms and documented editorial processes. These establish different guarantees; maturity does not imply universal necessity.

**Deployed patterns, limited causal evidence:** the production cases demonstrate viability and expose maintenance failures. Runbooks and maintainer accounts rarely isolate each choice's causal contribution to accuracy or productivity. Intended release schedules are not measured adherence.

**Frontier systems:** DBSP addresses incremental query maintenance, not source credibility or arbitrary generated-text updating. LOTUS optimizes semantic operators; its guarantees concern individual operators relative to a reference algorithm, not factual truth or arbitrary pipeline correctness. SemBench v2, 16 March 2026, is a versioned preprint with a cross-system workload; capability gaps and representativeness remain important. [DBSP 2023](https://www.vldb.org/pvldb/vol16/p1601-budiu.pdf), [LOTUS 2025](https://www.vldb.org/pvldb/vol18/p4171-patel.pdf), [SemBench v2](https://arxiv.org/html/2511.01716v2).

Specific evaluation findings must retain their own denominators and versions:

- The **2021 Wikidata study** found **61.5% of removed dates** replaced by semantically equivalent formats in its studied data. The denominator is removed dates, not all facts or errors. [Study](https://arxiv.org/html/2107.00156).
- **CRAG, inspected 7 June 2024 version**, evaluates frozen retrieval and historical systems. Its QA results are not live refresh, authorization or organizational productivity measures. [CRAG v1](https://arxiv.org/html/2406.04744v1).
- **ALCE 2023** and **RAGTruth 2024** assess support from supplied evidence, not an independent guarantee that this evidence is true or current. [ALCE](https://aclanthology.org/2023.emnlp-main.398/), [RAGTruth](https://aclanthology.org/2024.acl-long.585.pdf).
- **EMERGE v2, 7 April 2026**, studies additions/deprecations over yearly snapshots, with incomplete gold knowledge and limited temporal/literal coverage. Completeness scores are not harmful-edit precision; v1/v2 counts cannot be mixed. [EMERGE v2](https://arxiv.org/html/2507.03617v2).

No scores across these incompatible tasks are compared. This synthesis ran no SQL engine, comparative benchmark or user study. Its proposed checks remain unperformed.

## 6. Scope clarifications and review results

1. **Universal architecture wording narrowed.** The lifecycle model applies to evidence-bearing, multi-source services. Claim objects, immutable artifacts and manifests are optional mechanisms, not requirements for every document KB.
2. **Retention limits immutability.** Historical preservation applies only where permission and retention allow it. Erasure can limit replay; an immutable log does not remove this conflict.
3. **Identity cost is conditional.** False merges are especially dangerous in the contact/permission examples; they are not universally more costly than false splits.
4. **Modeling corrections incorporated.** Required names need NOT NULL under the lesson's assumptions; snapshot dependencies do not enforce immutable membership; the 3NF definition now uses singleton RHS. Independent audit found keys, decompositions and counterexamples valid under stated assumptions. No execution is claimed.
5. **GO numeric window unresolved.** The production memo flags a prose-date/link discrepancy in the 2026 update paper. Added/obsoleted totals are omitted here rather than silently assigning a comparison interval. Qualitative policy lessons use independent documentation.
6. **Apparent contradictions are often scope differences.** Open-world inference can coexist with closed submission requirements; advisory contributions with strict release gates; persistent identifiers with changed labels. Substantially changed referents are different from changed labels.
7. **Access and maturity labels retained.** Textbook supplements/contents are not full-book reading. Full 2023 DBSP text and 2025 journal-extension metadata are distinct evidence. EMERGE and SemBench remain versioned preprints here. Retrieved RDF 1.2/SHACL 1.2 drafts are not imported into mature-baseline claims.

## 7. Unknowns and a bounded continuation

Unsettled by this round, not claimed absent from all prior literature:

- Minimum context needed through extraction/compression under changing query demand.
- Dependence-aware source combination and when explicit disagreement helps most.
- Faithful support/access propagation through generated summaries at acceptable rebuild cost.
- Review allocation that prevents consequential errors without neglecting low-volume workloads.
- Accumulation and repair cost of identity mistakes and semantic migrations over long update streams.
- Causal improvement in verified user outcomes relative to full maintenance cost.

Choose at most **two next close readings**:

1. **EMERGE v2, 7 April 2026:** task construction, metrics, Table 3, annotation checks and limitations. Stop when a one-page explanation distinguishes additions, deprecations, linking and novelty; explains why its scores do not establish edit precision; and defines one small lifecycle replay exposing a missing capability. If the existing memo already answers this, consolidate it rather than reread mechanically. [Text](https://arxiv.org/html/2507.03617v2).
2. **Zanzibar, USENIX ATC 2019:** §2, §§3.1–3.2 and relevant evaluation definitions. Stop when permission removal followed by content update can be traced; its guarantee and assumptions stated; and the boundary with derived summaries explained. Do not select a product or transfer its historical performance numbers. [Paper](https://www.usenix.org/system/files/atc19-pang.pdf).

These close specific gaps in lifecycle evaluation and authorization consistency. The next decision is whether a small scoped experiment would help, not whether to keep expanding a reading tree or declare universal architectural mastery.
