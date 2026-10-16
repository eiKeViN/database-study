# Targeted frontier readings for the database learning shortlist

Checked 16 October 2026. Recommendation: add LOTUS as one frontier original systems paper; add SemBench only if a second slot is available. Keep these after relational models, physical execution, query optimization, transactions, and storage fundamentals. Neither is a substitute for a database textbook or mature operational documentation.

## 1. LOTUS: semantic operators and statistically constrained optimization

Liana Patel et al. **Semantic Operators and Their Optimization: Enabling LLM-Based Data Processing with Accuracy Guarantees in LOTUS.** PVLDB 18(11), 4171–4184, 2025. DOI 10.14778/3749646.3749685.

Primary full text: https://www.vldb.org/pvldb/vol18/p4171-patel.pdf

Why select it: a direct bridge from established database concepts to a concrete frontier. It treats semantic filter, join, grouping, ranking, and aggregation as declarative operators with alternative physical implementations.

Core learning objects: model–data independence; logical versus physical operators; reference-algorithm semantics; approximate execution with statistical constraints; model cascades and embedding proxies; cost/latency/quality tradeoffs. Read §§2–3 before the performance graphs.

Evidence: original implemented system with experiments on fact-checking, classification, search, and topic analysis. Open-source artifacts are linked in the paper. Its evidence is stronger than a vision-only essay but remains workload-specific.

Critical limitation: “accuracy guarantees” concern individual optimized operators relative to a chosen reference algorithm. They are not guarantees that the reference LLM tells the truth or that an arbitrary composed pipeline is correct. Large reported speedups use particular baselines; do not present the maximum as typical end-to-end benefit.

中文：重点学习“如何把模糊的语义任务变成可优化的算子”，以及“保证到底是相对于真实答案，还是相对于参考算法”。

## 2. SemBench: measuring semantic query engines as systems

Jiale Lao et al. **SemBench: A Benchmark for Semantic Query Processing Engines.** First posted 3 November 2025; inspected latest v2 dated 16 March 2026. Treat as a versioned preprint: its PDF-style header still contains placeholder PVLDB metadata, so publication acceptance is not verified here.

Primary latest version: https://arxiv.org/html/2511.01716v2
Version record: https://arxiv.org/abs/2511.01716
Artifact site linked by authors: https://sembench.org

Why select it: complements a system’s self-evaluation with an explicit cross-system workload and evaluation design. Five scenarios and 55 queries combine traditional relational processing with semantic operators across text, images, and audio.

Core learning objects: workload coverage; isolated operators versus multi-operator queries; scale factors; ground truth constructed from labeled datasets; query-specific quality measures alongside latency, monetary cost, memory, and scalability.

Limits: datasets combine existing labeled collections and generated metadata; benchmark coverage is not production representativeness. Results are snapshots of rapidly changing systems. Query capability differences must be reported, not hidden by averages over only successfully supported queries. Version warning: v1 included a medical scenario; v2 uses Cars, so do not mix scenario descriptions or results across versions.

中文：要同时问“答案是否对、花了多少钱、用了多久、哪些查询根本不支持”，不能只比较一个平均分。

## Optional alternatives, not additional mandatory reading

- **Palimpzest**, CIDR 2025, https://www.vldb.org/cidrdb/2025/palimpzest-optimizing-ai-powered-analytics-with-declarative-query-processing.html . A good alternative to LOTUS for learning declarative AI workloads and cost-based model/prompt selection. It is an original prototype systems paper; empirical quality optimization is distinct from LOTUS’s reference-relative operator guarantees. The two need not both occupy a short initial syllabus.
- **The Seattle Report on Database Research**, 2020, author-hosted PDF https://www.cs.umd.edu/~abadi/papers/seattle-report.pdf . An agenda-reading option for broader perspective on data governance, context, provenance, reproducibility, and heterogeneous data. Useful as a research map, not a current 2026 frontier census and not empirical evidence that proposed solutions work.

## How these fit the larger reading sequence

1. Foundations explain what a database promises and how data/query independence works.
2. Implementation courses explain physical operators, indexes, optimization, transactions, and recovery.
3. A knowledge-graph synthesis explains representation, entities, rules, context, and integration.
4. LOTUS asks what changes when operators invoke imperfect models.
5. SemBench asks how to compare those systems without confusing model accuracy, query support, and execution cost.

A later research phase can connect this to contextual fidelity, provenance-aware maintenance, temporal updates, and human governance. The broader evidence memo is preserved separately; these two readings are the immediate curated recommendation.
