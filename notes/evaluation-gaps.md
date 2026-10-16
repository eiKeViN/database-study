# Knowledge-base quality: evaluation, demonstrated effects, and remaining research gaps

Research checked 16 October 2026. Bounded first-round evidence review; this is not an exhaustive survey or a claim that any proposed research question has no prior work. No domain-specific mathematics mapping is attempted. Exact primary-source links and versions are supplied below. Proposed evaluation designs are recommendations from this synthesis, not validated results from the cited papers.

## Main judgment

Knowledge-base maturity should mean dependable behavior over a declared scope, workload, time interval, and access policy. Neither a large triple count nor high retrieval or answer scores establishes that. Mature components exist for storage, indexing, constraints, versioning, provenance, and human review. The harder remaining question is how to combine these components so that useful answers remain correct, appropriately qualified, traceable, and maintainable as sources and permissions change.

中文要点：知识库的成熟度，不应由“收录了多少知识”定义，而应由“在什么范围、什么时间、对什么用户、支持什么任务，以及出错后能否发现、追溯和修复”定义。

## Five evidence-backed findings

### 1. A link-prediction leaderboard is not a completeness audit

Dettmers et al., AAAI 2018, demonstrated that inverse-relation leakage in WN18 and FB15k was severe enough for a simple rule model to attain state-of-the-art results. Their work motivated evaluation on harder variants. The demonstrated result is about benchmark validity, not a universal indictment of embeddings. Nevertheless, recovering hidden edges in a fixed graph cannot establish coverage of facts absent from that graph, unseen entities, or future updates. Random splits can reward memorizing redundant structure rather than filling operational gaps. [Primary paper, published 25 April 2018](https://ojs.aaai.org/index.php/AAAI/article/view/11573)

Knowledge Vault offers a complementary systems lesson: supervised fusion of multiple extraction channels and graph-derived priors was feasible at web scale. But its local closed-world labeling assumes that, once a subject–predicate pair has known values, alternative unrecorded values can be labeled incorrect. That expedient is potentially problematic for incomplete multivalued relations. Calibrated probabilities are useful only relative to the labeling process and evaluation distribution. [Dong et al., KDD, August 2014, full paper](https://www.cs.ubc.ca/~murphyk/papers/kv-kdd14.pdf)

Design implication: separately report discovery coverage, independently checked precision, novel-entity performance, and future-snapshot performance. Do not call MRR or Hits@k “knowledge completeness.”

### 2. Consistency checks identify work; they do not decide what is true

The 2021 study of Wikidata analyzes deletion, deprecation, and community-defined constraints. Its authors explicitly note that removals are neither necessary nor sufficient indicators of factual error: representation changes and stylistic normalization also cause removals. They found 61.5% of removed dates were replaced by semantically equivalent formats. For a missing symmetric relation, either the reciprocal fact is missing or the existing assertion is wrong; merely adding the inverse can propagate an error. Constraints themselves can be outdated or too narrow. These are observations from a functioning large collaborative KB, not hypothetical toy failures. [Ilievski et al., first submitted 30 June 2021, full study](https://arxiv.org/html/2107.00156)

Design implication: distinguish syntax, schema adherence, logical consistency, source support, and world correctness. Count successful error resolution, not just violations eliminated. A repair that satisfies a rule by inventing a fact should fail evaluation. Provenance and adjudication are needed to choose among repairs.

中文要点：一致性是“系统内部不打架”，真实性是“对外部世界说得对”。把所有冲突消掉，可能只是把错误变得更整齐。

### 3. Retrieval helps, but static aggregate accuracy hides important failure slices

CRAG contains 4,409 questions, five domains, multiple temporal regimes, and mock web/KG retrieval. In its 2024 evaluation, straightforward RAG reached roughly 44% accuracy compared with at most roughly 34% for the evaluated LLM-only systems; the best evaluated commercial systems produced completely correct, hallucination-free answers for about 63% of questions. These are historical benchmark results, not current product rankings. Its human score distinguishes perfect, acceptable, missing, and incorrect responses, penalizing incorrect responses more than abstention. Its search candidates are frozen and questions are partly template-generated; “end-to-end” therefore does not reproduce unrestricted live search, operational refresh, or enterprise authorization. [Yang et al., 7 June 2024 version, methods and appendix](https://arxiv.org/html/2406.04744v1)

FreshLLMs separately demonstrates the importance of fast-changing questions and false premises, using more than 50,000 human judgments. Search-augmented prompting improved the studied systems, but evidence order and amount affected correctness. [Vu et al., first submitted 5 October 2023](https://arxiv.org/abs/2310.03214)

Design implication: measure quality conditional on query type, age, entity popularity, answerability, and retrieval evidence availability; track abstention alongside errors.

### 4. A citation is neither sufficient support nor independent truth verification

ALCE separates answer correctness, fluency, and citation quality. Its 2023 experiments found even the best evaluated systems lacked full citation support around half the time on ELI5. Its methods matter: passage-level evidence, sentence-level statements, automated entailment checks, and separate answer-coverage measures. Its QAMPARI recall-5 deliberately counts five correct examples as full recall, illustrating that a reasonable metric for “give examples” is misleading for “give every member.” Thus apparent completeness depends on the question’s contract. Automated entailment is itself fallible, and an entailed statement can repeat a false or outdated source. [Gao et al., EMNLP, December 2023](https://aclanthology.org/2023.emnlp-main.398.pdf)

RAGTruth adds nearly 18,000 model responses with manual case- and word-level hallucination annotations, demonstrating that source-grounded generation still requires direct checking for unsupported or contradictory claims. Its labels concern the relationship to supplied evidence, not a universal audit of that evidence’s truth. [Niu et al., ACL 2024](https://aclanthology.org/2024.acl-long.585.pdf)

Design implication: evaluate source authority/currentness, claim support, answer sufficiency, and downstream user success separately. Citation presence alone is a poor proxy for all four.

### 5. Updating an existing KB is a different task from extracting more triples

EMERGE explicitly evaluates existing facts, additions, new entities, linking into an existing graph, and deprecation. Its April 2026 version contains 233K passages aligned with 1.45 million edits across seven yearly snapshots. Table 3 reports EDC+ with GPT-5.1 completeness of 40.2 for Add and 34.8 for Deprecate; semantically soft recall scores are substantially higher. These numbers are benchmark-specific, not precision estimates. The paper avoids precision/F1 because gold knowledge is incomplete. Curation uses LLM judgments checked against 500 triples annotated by two experts. Explicit limitations include Wikipedia/Wikidata timing mismatch, omission of literal-valued relations, selected hyperlink-grounded text, and lack of full start/end validity intervals. [Zaporojets et al., v2, 7 April 2026](https://arxiv.org/html/2507.03617v2)

Design implication: promising evidence that lifecycle-aware evaluation is becoming concrete; insufficient evidence for safe autonomous updates. Production readiness still requires harmful-edit precision, rollback, multi-step error accumulation, and downstream query tests.

中文要点：抽取一条看似正确的新知识，只完成了维护工作的一小部分；还要判断它对应谁、替代什么、何时有效，以及哪些已有结论需要随之改变。

## Quality dimensions and how to measure them

The following is a proposed integrated evaluation specification. It combines familiar measurement families with operational tests; it is not a claim that one established standard contains this exact list.

1. **Correctness.** Sample assertions stratified by relation, source, age, confidence, and importance; independently adjudicate against suitable evidence. Report precision, uncertainty intervals, error severity, and calibration. A random sample alone may miss rare but expensive errors. Agreement with an existing KB is a silver standard, not ground truth.

2. **Completeness within declared scope.** Define eligible entities, properties, times, and required task answers. Compare with an independently constructed reference set where feasible; distinguish entity, property, relation, evidence, and answer-set coverage. Unknown denominator means absolute completeness is not measurable. Relative completeness versus similar entities can prioritize investigation but inherits those entities’ biases.

3. **Consistency.** Count violations per applicable constraint, severity, and affected entity; test whether rules themselves remain appropriate. Audit repairs against evidence. Zero violations could reflect weak constraints, missing facts, or aggressive deletion.

4. **Freshness and temporal validity.** Record event-valid time, publication time, ingestion time, and last verification separately. Measure delay from authoritative change to correct query behavior, stale-answer rate, and historical-query accuracy. Recently ingested old content is not fresh knowledge; overwriting old facts can improve present answers while destroying history.

5. **Provenance.** Measure the fraction of consequential claims with recoverable source passages, version identifiers, transformation records, and reviewer or extractor information. Test lineage traversal after correction or deletion. Distinguish a reference URL from a complete derivation trace. Provenance coverage is not source credibility.

6. **Contextual fidelity.** Test retention of qualifiers: time, jurisdiction, population, measurement method, units, negation, uncertainty, and attribution. Use paired examples whose wording is almost identical but context changes the correct answer. Exact triple matching alone can overlook stripped qualifiers.

7. **Retrieval utility.** Use Recall@k, precision, nDCG, evidence sufficiency, and multi-hop evidence coverage as diagnostic measures. Then hold the generator fixed and measure answers with actual versus oracle retrieval. A relevant passage need not contain decisive evidence; a correct answer may come from model memory and hide retrieval failure.

8. **Maintenance effort.** Measure expert minutes per verified useful change, review queue age, false-positive alert burden, recomputation cost, update latency, and rollback effort. Include debugging and rejected suggestions. Automated extraction volume can rise while net maintenance productivity falls.

9. **Access control and deletion.** Test allowed and forbidden queries under real identity boundaries, permission revocations, shared caches, summaries, derived nodes, embeddings, and previous answers. Measure unauthorized exposure and revocation propagation delay as first-class outcomes. Treat permissions as hard constraints rather than an average score that can be offset by better accuracy.

10. **End-user effect.** In controlled tasks measure completion success, independently verified decision quality, time spent checking, errors carried into subsequent work, and appropriate reliance. Satisfaction and fluent answers are secondary; users can prefer confident wrong answers.

## Practical tradeoffs and misleading proxies

- More evidence can improve recall but increase conflicting, outdated, duplicated, or unauthorized material. Evaluate marginal useful evidence, not document count.
- Canonicalization reduces duplicates but a false merge contaminates many facts. False splits hurt recall; false merges may be much more costly. Use cluster-level and downstream impact measures alongside pairwise entity-resolution F1.
- Aggressive freshness can import unverified breaking reports. Strong verification slows incorporation. Evaluate a quality–delay frontier under domain-specific consequence levels.
- Preserving every conflict aids auditability but increases query ambiguity and review cost. A single “best value” improves simplicity but can hide legitimate disagreements.
- Large schemas preserve context but increase extraction difficulty and human effort. Small schemas simplify ingestion but can irreversibly discard distinctions.
- Higher abstention may improve accuracy among answered questions while making the system useless. Report coverage–error curves and task completion, not conditional accuracy alone.
- A high model-judge correlation on one benchmark does not guarantee correct ranking under a new language, task, or adversarial source. Human audit should cover disputed and high-cost cases, not just a random easy subset.
- Triple growth, tokens indexed, citations per answer, clicks, and benchmark scores each measure something. None alone measures maintained knowledge usefulness.

## Open improvement spaces: engineering versus scientific uncertainty

### A. Context and time

**Known engineering work:** retain qualifiers; version sources; represent validity intervals; support historical snapshots and event-time versus ingestion-time queries.

**Research question:** Which minimal context must survive extraction and compression to preserve correctness across a changing query distribution? When does a change mean correction, expiration, differing perspective, or additional valid detail?

**Proposed test:** assemble temporally ordered source bundles with amendments, late reports, retrospective corrections, and conflicting authorities. Compare flattened triples, qualified claims, and source-first retrieval under equal storage and retrieval budgets. Score present and historical answers, qualifier omission, update delay, and contradiction handling. Keep future documents unavailable to earlier queries.

### B. Conflict and uncertainty

**Known engineering work:** retain competing claims, source identity, confidence, and review status; avoid automatic destructive resolution.

**Research question:** How should a system combine dependent sources and uncertainty without treating repeated copies as independent confirmation? When should it present disagreement rather than choose a value?

**Proposed test:** controlled source-copying networks plus naturally disputed facts. Evaluate probability calibration, false consensus, source sensitivity, appropriate abstention, and usefulness of explanations. Vary authority and timestamp independently so the model cannot succeed using “newest wins.”

### C. Continual evolution and entity identity

**Known engineering work:** persistent IDs, aliases, merge/split logs, incremental indexing, idempotent updates, rollback.

**Research question:** Can jointly learned extraction, linking, and schema adaptation improve accumulated KB utility under novelty without accumulating self-reinforcing mistakes?

**Proposed test:** replay an unseen stream with new entities, renamed organizations, ambiguous names, mergers, and schema changes. Compare independent extraction, graph-aware updates, and human-approved updates. Report false merges/splits, edit precision, restoration accuracy, historical query regressions, and expert cost over the entire stream, not only the final snapshot.

### D. Human governance at scale

**Known engineering work:** ownership, review queues, escalation, audit trails, service targets, and bounded automatic actions.

**Research question:** Which review-allocation strategy most improves consequential user outcomes per unit expert effort, while avoiding neglected long-tail groups and relations?

**Proposed test:** randomize review queues across uncertainty, source novelty, graph-impact, and actual query-demand prioritization. Fix reviewer hours. Blindly adjudicate accepted and rejected edits, and measure downstream mistakes prevented, reviewer fatigue, and distribution of unresolved errors. Observational correction logs alone cannot establish a strategy’s causal benefit.

### E. Provenance-aware retrieval and source deletion

**Known engineering work:** deletion propagation through deterministic indexes and explicit derivations; cache invalidation; permission checks before retrieval and delivery.

**Research question:** How can support be tracked through generative summaries and fused claims sufficiently well to revoke evidence, permissions, or confidence without rebuilding everything? How can a system distinguish “no longer authorized to use this source” from “this fact is false”?

**Proposed test:** remove or revoke a source after index, summary, and graph construction; include claims with one versus multiple independent supporting sources. Measure residual forbidden content, unsupported surviving assertions, unnecessary removal of still-supported claims, rebuild cost, and latency. This is primarily a systems correctness requirement; efficient faithful propagation through lossy generative transformations adds a research component. It is not solved merely by deleting one vector-store entry.

### F. Query-conditioned knowledge quality

**Known engineering work:** route queries, measure workload slices, monitor failures, and maintain task-specific evaluation sets.

**Research question:** Can maintenance prioritize what is useful now without making future or minority workloads worse? Can the system reliably identify when apparently complete evidence is incomplete for a particular question?

**Proposed test:** train maintenance policies on one workload, evaluate under shifted demand and held-out query families. Include exhaustive-set questions, counterfactual premise checks, multi-hop evidence, and permissions. Score marginal task improvement per accepted update, worst-slice quality, and failure under distribution shift. A workload-weighted scalar may help allocation, but publish the underlying dimensions.

## What the evidence does and does not establish

The inspected papers establish that useful individual capabilities exist and expose specific measurement failures. They do not establish that any universal KB architecture is best, that GraphRAG always beats document retrieval, or that benchmark improvements translate into organizational productivity. This strand did not identify a convincing randomized field study establishing end-to-end maintenance ROI across organizations. That is an evidence gap in this review, not proof that no such study exists.

A defensible next evaluation has three layers: assertion/source audits; full-lifecycle replay with temporal and permission changes; controlled user tasks with verified outcomes and costs. Use the same frozen corpus and generator for architecture comparisons, disclose extraction and indexing budgets, include a strong lexical/hybrid baseline, and separate oracle-retrieval from real-retrieval runs. Publish failure categories and uncertainty intervals rather than only a winning average.

## Source ledger and interpretation warnings

- Dettmers et al., *Convolutional 2D Knowledge Graph Embeddings*, AAAI 2018, publisher date 25 April 2018. https://ojs.aaai.org/index.php/AAAI/article/view/11573 . Original leakage demonstration; historical static KGC setting.
- Dong et al., *Knowledge Vault: A Web-Scale Approach to Probabilistic Knowledge Fusion*, KDD, 24–27 August 2014. https://research.google/pubs/knowledge-vault-a-web-scale-approach-to-probabilistic-knowledge-fusion/ ; full paper https://www.cs.ubc.ca/~murphyk/papers/kv-kdd14.pdf . Original industrial research system, not a present service-quality audit.
- Ilievski et al., *A Study of the Quality of Wikidata*, 2021. https://arxiv.org/abs/2107.00156 ; inspected full text https://arxiv.org/html/2107.00156 . Observational community and constraint indicators, not universal factual ground truth.
- Yang et al., *CRAG—Comprehensive RAG Benchmark*, first posted 7 June 2024; NeurIPS 2024 Datasets and Benchmarks. Inspected version https://arxiv.org/html/2406.04744v1 ; proceedings https://proceedings.neurips.cc/paper_files/paper/2024/file/1435d2d0fca85a84d83ddcb754f58c29-Paper-Datasets_and_Benchmarks_Track.pdf . Frozen retrieval and historical models; avoid presenting numbers as current SOTA.
- Vu et al., *FreshLLMs: Refreshing Large Language Models with Search Engine Augmentation*, first posted 5 October 2023. https://arxiv.org/abs/2310.03214 . Supplementary temporal QA evidence; primary abstract inspected, not all result tables.
- Gao et al., *Enabling Large Language Models to Generate Text with Citations*, EMNLP, December 2023. https://aclanthology.org/2023.emnlp-main.398/ ; full paper https://aclanthology.org/2023.emnlp-main.398.pdf . Inspect metrics, especially recall-5, before reusing completeness claims.
- Niu et al., *RAGTruth*, ACL 2024; preprint first posted 31 December 2023. https://aclanthology.org/2024.acl-long.585.pdf ; https://arxiv.org/abs/2401.00396 . Human source-faithfulness labels; supplementary corpus evidence, not a current-model comparison.
- Zaporojets et al., *EMERGE*, v1 4 July 2025, v2 7 April 2026. Use https://arxiv.org/html/2507.03617v2 and https://arxiv.org/abs/2507.03617v2 . The v1 counts (376K passages, 1.25M edits, ten snapshots) differ from v2 and must not be mixed. April 2026 revision inspected including metrics, Table 3, annotation appendix, and limitations. Publication venue was not verified here; describe as the versioned preprint.

All source-based summaries above are intentionally brief; the evaluation framework and proposed experiments are this memo’s independent synthesis. Questions needing a second literature pass include access-control-aware retrieval guarantees, deletion through generative derivatives, source-dependence calibration, and causal field evidence on governance ROI.
