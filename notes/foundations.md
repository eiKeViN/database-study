# Foundations: a selected learning spine, with preserved research notes

Research checked: 16 October 2026. Scope: general database and knowledge-base design; no application to mathematics. The immediate deliverable is selection of learning materials, not a completed survey. Part I is the shortlist. Part II preserves findings for subsequent study. Recommendations, examples, and proposed exercises are my synthesis; statements about documents are sourced separately. No code or implementation was tested.

## Chinese takeaway

建议先以《Database System Concepts》第七版建立完整的数据库概念框架，再用 CMU 15-445 的已完成课程理解实现代价；Codd 与 Chen 的原论文用于回到设计动机。RDF、OWL、SHACL 等标准应作为后续“语义与互操作”专题，而不是替代数据库基础。先分清概念建模、逻辑模式、物理实现，再讨论知识图谱；尤其要分清“推理得到什么”与“数据必须填写什么”。这一阶段的目标是选定可信的学习主线，并明确每份材料解决和不解决的问题。

# Part I. Five selected learning objects

## 1. Main textbook: Database System Concepts, seventh edition

**Bibliographic/access facts.** Avi Silberschatz, Henry F. Korth, S. Sudarshan; McGraw-Hill; released March 2019; ISBN 9780078022159. The authors’ [book site](https://www.db-book.com/) identifies the edition and release. [Table of contents](https://www.db-book.com/toc-dir/toc.pdf), [preface](https://www.db-book.com/preface-dir/preface.pdf), and [chapter slides](https://www.db-book.com/slides-dir/index.html) are publicly readable. The full textbook has not been verified as legally open access; obtain it through a library or publisher. Public supplements are not equivalent to the full book.

**Exact route I recommend.** First pass: §§1.3–1.7; chapter 2; chapters 3–4, especially §3.6 and §4.4; chapter 6, especially §§6.1–6.9; chapter 7, initially §§7.1–7.3 and §§7.8–7.10. Next pass: chapters 13–16 and 17–19. Bridge to other representations: §8.1, §8.3, §10.6; distributed systems later in chapters 20–23. The numbering is verified against the seventh-edition contents, not another edition.

**Prerequisites.** The authors assume basic data structures, computer organization, and a high-level programming language. Their preface explicitly emphasizes intuitive explanations over formal proofs.

**Why select it.** My judgment: it provides the most coherent backbone here because design, query language, storage, optimization, and correctness live in one connected vocabulary. Read it to understand how a schema decision becomes an update rule, query, access path, and operational cost.

**What it does not replace.** A dedicated ontology-engineering treatment, enterprise editorial practice, production operations handbook, or current frontier research. Its age is acceptable for foundational concepts, but named technologies and contemporary performance claims need newer sources.

**Useful learning output.** Explain one small equipment-lending schema at three levels: domain concepts, tables/constraints, and indexes/transactions. Do not merely memorize normal-form names.

## 2. Implementation companion: CMU 15-445/645, Fall 2025

**Version/access.** Andy Pavlo’s [official archived course](https://15445.courses.cs.cmu.edu/fall2025/); [schedule](https://15445.courses.cs.cmu.edu/fall2025/schedule.html), updated 4 December 2025; [syllabus](https://15445.courses.cs.cmu.edu/fall2025/syllabus.html), updated 26 August 2025. The latest link resolves to Fall 2026, but I select the completed 2025 offering for a stable self-study sequence. Public slides and linked recordings are listed; sample lecture-slide PDFs opened successfully. Two lecture-note downloads timed out, so universal access to every asset is not claimed.

**Exact selection.** Lectures 01–02 for model/SQL; 03–06 for storage/memory; 07–10 for indexes; 11–16 for query execution and optimization; 17–20 for concurrency; 21–22 for logging/recovery; 23–24 for distributed systems. Sample inspected artifacts: [01 relational model](https://15445.courses.cs.cmu.edu/fall2025/slides/01-relationalmodel.pdf) and [15 optimization](https://15445.courses.cs.cmu.edu/fall2025/slides/15-optimization1.pdf).

**Prerequisites.** Officially CMU 15-213/513; C++17 programming/debugging is assumed for projects, which use BusTub. Reading/watching selected lectures does not require completing the full project sequence.

**Why select it.** My judgment: this exposes the cost hidden by a clean diagram. A relationship may be simple to describe while joins, cache behavior, index maintenance, and concurrent updates are difficult to implement.

**Omissions/caution.** This is a DBMS-internals course, not the primary resource for stakeholder discovery, conceptual modeling, ontology governance, or article-quality workflows. Do not let implementation projects consume the whole initial learning phase. Public course availability does not imply enrollment, grading, or unrestricted redistribution rights.

## 3. Founding motivation paper: E. F. Codd, 1970

**Exact work.** “A Relational Model of Data for Large Shared Data Banks,” Communications of the ACM 13(6), 377–387, June 1970. [DOI](https://doi.org/10.1145/362384.362685); [full-paper mirror inspected](https://db.dobo.sk/wp-content/uploads/2015/11/Codd_1970_A_relational_model.pdf). The mirror is not the publisher, but the content is the original primary paper. Read §§1.1–1.2, §1.4, §§2.2–2.3; reserve operation formalism for a slower pass.

**Prerequisites.** Understand tables, keys, and joins first. The historical notation should not be mistaken for modern SQL.

**Supported lesson.** The paper motivates independence of programs from physical organization and distinguishes logical redundancy from stored redundancy. It discusses consistency and explicitly recognizes that redundant storage can trade update/storage cost against query performance.

**Why select it.** My judgment: it prevents the mistaken history that relational databases were simply about arranging information in rectangular tables. The underlying design goal helps evaluate modern graph/document systems too.

**Omissions.** It is not a current DBMS implementation guide, modern SQL reference, or complete account of later normalization theory. Do not attribute the entire present-day normal-form hierarchy to this paper.

**Learning output.** State which equipment-lending queries must remain unchanged after adding an index or changing storage organization.

## 4. Conceptual-design paper: Peter P. Chen, 1976

**Exact work.** “The Entity-Relationship Model—Toward a Unified View of Data,” ACM TODS 1(1), 9–36, March 1976. [Publisher record](https://doi.org/10.1145/320434.320440); [full original paper inspected](https://weichselbraun.net/dbs/pdf/literature/chen.pdf); [MIT institutional working-paper record](https://dspace.mit.edu/entities/publication/5f4dcb94-111c-4368-8374-cec8842dee1d). MIT’s direct PDF failed in this session; do not claim that route was read.

**Read closely.** §2.1 on levels; §§2.2–2.3 on entities, relationships, roles, attributes, and information structure; §§3.1–3.2 on diagrams and the worked design process. Read §4 afterward for historical comparisons.

**Prerequisites.** Basic relational vocabulary; no implementation project needed.

**Supported lesson.** Chen separates conceptual information from access-path-dependent structures, and models semantic distinctions before selecting keys and relations. The paper acknowledges that representing something as an entity versus a relationship can depend on the application.

**Why select it.** My judgment: this is the best corrective to starting with a storage product and forcing the domain into its defaults. It also makes a useful comparison with ontology engineering without treating them as identical.

**Omissions.** Historical notation and system comparisons need context. It does not provide contemporary ontology semantics, migrations, governance, search relevance, or a modern distributed architecture.

**Learning output.** Decide whether a loan needs its own identity because it has a borrower, item, dates, return condition, and repeated occurrences.

## 5. Later semantic-interoperability unit: a deliberately bounded W3C reading packet

This is one learning unit composed of distinct specifications, not one book and not a demand to read every standard end to end.

**Core route.** [RDF 1.1 Concepts](https://www.w3.org/TR/2014/REC-rdf11-concepts-20140225/), §§1.1–1.8 and §§3–4; [RDF Schema 1.1](https://www.w3.org/TR/rdf-schema/), §§2–3; [OWL 2 Primer, second edition](https://www.w3.org/TR/owl2-primer/), §§2–5 and §§9–10; [SHACL Recommendation](https://www.w3.org/TR/2017/REC-shacl-20170720/), §§1.5, 2, 3.4–3.6, and cardinality/closed-shape portions of §4. These are free full documents. Dates: RDF/RDFS 25 February 2014; OWL second edition 11 December 2012; SHACL 20 July 2017.

**Prerequisites.** Entity identity, schema, keys, joins, and constraints; then RDF triples and basic class/property vocabulary. Formal semantics can wait until a concrete ambiguity demands it.

**Why select it.** My judgment: it provides an authoritative reference for exactly what an exchanged statement means and what validation does. It is the antidote to claims that any graph with labels has agreed semantics.

**Boundary.** These documents do not supply a storage architecture, operational SLA, truth verification, entity-resolution policy, or content-governance program. Normative specifications and informative primers have different purposes.

**Version warning.** Retrieved [RDF 1.2](https://www.w3.org/TR/2026/CR-rdf12-concepts-20260407/) is a Candidate Recommendation Snapshot dated 7 April 2026. Retrieved [SHACL 1.2 Core](https://www.w3.org/TR/2026/WD-shacl12-core-20261014/) is a Working Draft dated 14 October 2026. Treat these as a later frontier-watch branch; do not silently import their features into claims about the mature baseline.

## Selection judgment and order

For the immediate request, start with objects 1 and 4, use 3 to understand motivation, and consult 2 when asking implementation questions. Object 5 is a subsequent specialization. This selection is intentionally weighted toward databases proper. It is not a claim that relational storage should be chosen for every eventual system.

Defer tool selection and implementation. First decide whether the learner can explain: identity versus labels; entities versus events; conceptual versus logical versus physical design; missing versus false; constraints versus inference; and logical correctness versus operational correctness.

# Part II. Preserved foundation notes for later study

## A. Working distinctions, not a universal taxonomy

The following vocabulary is a practical synthesis for this study:

- **Database:** an organized data collection. A DBMS supplies mechanisms for managing it. A schema can be meaningful without making the system a knowledge-reasoning engine.
- **Knowledge base, formal-AI sense:** represented assertions and domain knowledge interpreted by some representation/reasoning discipline. Its boundary with ordinary data is contextual.
- **Knowledge base, enterprise/document sense:** reusable explanations, procedures, and answers, supported by metadata, search, maintenance, and audience controls. Do not infer formal logical semantics from the label “knowledge.”
- **Knowledge graph:** an entity/relationship-centered representation used to connect information. This working label alone specifies neither RDF nor OWL, neither inference nor correctness, neither a particular physical database nor comprehensive knowledge.
- **Ontology:** an explicit account of the vocabulary and commitments used to describe a domain. An ontology is not the entire running system.
- **Semantic Web:** a family of web-oriented representation/interoperability technologies and practices. It is not synonymous with all knowledge bases or all graphs.

A useful historical framing is Davis, Shrobe, and Szolovits, [“What Is a Knowledge Representation?” (1993)](https://groups.csail.mit.edu/medg/ftp/psz/k-rep.html): representation involves what stands for the world, which distinctions are accepted, how reasoning proceeds, computational utility, and human expression. This prevents evaluating a design solely by storage format.

Gruber’s [1993 ontology paper](https://tomgruber.org/writing/ontolingua-kaj-1993.pdf), especially §§1–2 and the sharing discussion, treats a shared ontology as a vocabulary/commitment interface, not a guarantee that every participant can answer every query. It explicitly leaves important consensus problems outside its solution.

For the document sense, the Consortium for Service Innovation’s [KCS v6 guide](https://library.serviceinnovation.org/KCS/KCS_v6/KCS_v6_Practices_Guide) is a primary practitioner source: capture, structure, reuse, improve, and content health are explicit parts of the method. It was released 21 April 2016; the landing page reports a static PDF update on 7 April 2025. A newer guide bearing a 2027 title appeared in search, so it is not silently substituted for this dated baseline. Organizational-experience guidance is not controlled evidence of universal effectiveness.

## B. Worked ordinary-domain example: equipment lending

All examples in this section are constructed illustrations, not empirical case-study findings.

Suppose a community workshop lends drills and projectors. It maintains inventory, manufacturer specifications, instruction manuals, and troubleshooting articles.

**Conceptual questions.** A physical drill is distinct from the product model. The model’s manual is distinct from the current PDF revision. A person is distinct from their borrower account. A loan is an event/relationship occurrence, not a permanent property of an item. “Power tools” is a browsing topic; it need not define a formal class used in eligibility rules.

**Logical alternatives.** A relational design might use Item(item_id, model_id), ProductModel(model_id, manufacturer_id), Loan(loan_id, item_id, borrower_id, checked_out_at, returned_at), and DocumentVersion(document_version_id, document_id, revision). A graph can represent the same distinctions with identified nodes and named edges. Neither representation repairs a wrong distinction automatically.

**Physical choices.** Indexing active loans by item, caching a joined inventory screen, and storing manuals in object storage are separate decisions. They should be evaluated against workload and update requirements rather than encoded as facts about what a drill is.

**Normalization illustration.** If every loan repeats the manufacturer’s current support phone, a phone change requires many edits. Separating manufacturer information can reduce that update anomaly. But a support number printed on a historical loan receipt may be a historical fact worth preserving. Therefore the question is not simply “Is this string duplicated?” It is “Does this value mean current manufacturer data or the value recorded at that event?” A denormalized read view can be useful if its source and refresh rules are explicit.

**Historical identity.** Replacing the workshop’s drill with another physical drill of the same model should create a new item identity. Renaming its display label should not. A repaired drill might retain its item identity, but the organization must define the boundary for component replacement. The data model cannot settle that policy without a domain decision.

## C. Identity and naming

[Web Architecture, Recommendation 15 December 2004](https://www.w3.org/TR/webarch/), especially §§2.2–2.3 and §3.5.1, addresses URI allocation, collisions, and persistence. One identifier used for both a physical item and its description creates ambiguity about such properties as creation date.

Design synthesis: keep stable identifiers separate from editable labels; record source-system identifiers in a namespace; define merge/split procedures; preserve redirects or deprecation mappings; distinguish document identity from revision identity. “Same display name,” “same manufacturer serial,” and “same real object” are separate tests. Their reliability depends on domain evidence.

RDF 1.1 provides a graph of triples, IRIs, literals, blank nodes, and datasets. Blank-node labels are local rather than durable portable identifiers. A named graph can separate data, but RDF does not prescribe that its name denotes the graph or supplies trustworthy provenance. The model is atemporal; temporal meaning requires an explicit vocabulary. Source: RDF Concepts §§1.5–1.7, 3.4, 4.

## D. Meaning, inference, and validation are separate contracts

**RDFS.** Domain and range statements support class-membership inference. They are not simply form-field restrictions. If `borrowedBy` has domain Loan and someone attaches it to an item, RDFS can classify the item as a Loan rather than report the intended data-entry error. Multiple domains mean membership in all of them, not an either/or choice. Source: [RDFS §§3.1–3.2](https://www.w3.org/TR/rdf-schema/).

**OWL.** Its open-world stance does not turn missing statements into false statements. Distinct names need not identify distinct individuals. The primer explicitly distinguishes this from common database assumptions. These are semantic choices, not claims that an OWL-backed application cannot impose stricter workflow requirements. Source: [OWL Primer §§2, 4.7](https://www.w3.org/TR/owl2-primer/).

**Constructed cardinality illustration.** Say each registered item has exactly one owning organization. Under [OWL 2 Direct Semantics §§2.2.3, 2.3](https://www.w3.org/TR/2012/REC-owl2-direct-semantics-20121211/), no stored owner need not contradict that axiom: an owner can exist without being named. Two named owners under an applicable maximum-one restriction can be interpreted as the same owner; explicitly distinct owners would instead conflict. This differs from counting two submitted field values. These examples assume the ontology has the stated axioms and no other relevant contradiction.

**SHACL.** The 2017 Recommendation validates a specified data graph against shapes, reports results, and can require cardinalities or restrict permitted properties. Full RDFS inference is not mandatory; requested unsupported entailment regimes must fail. Recursive-shape semantics are left to implementations. Source: [SHACL §§1.5, 3.4–3.6, 4](https://www.w3.org/TR/2017/REC-shacl-20170720/).

Design synthesis: write separate contracts for accepted input, inferred conclusions, stored assertions, and displayed answers. An input shape might require one owner IRI, while a domain ontology explains ownership. Decide whether validation sees raw submitted data, integrated data, or materialized inferences. Otherwise an inference can hide an omission that the submission process was supposed to catch.

**Query absence is another distinction.** SPARQL 1.1 §8 defines NOT EXISTS by absence of a graph-pattern match in the query context. A query finding no current-loan record is not, by itself, evidence that the workshop physically possesses the drill. Source: [SPARQL Query, Recommendation 21 March 2013](https://www.w3.org/TR/sparql11-query/). Completeness assumptions belong in the application’s data contract.

## E. Events, vocabularies, and provenance

**Multi-part facts.** The W3C [N-ary Relations Note, 12 April 2006](https://www.w3.org/TR/swbp-n-aryRelations/) describes introducing a relation instance with links to participants. In the workshop example, Loan L17 links borrower, item, checkout time, due time, and return condition. This avoids inventing disconnected “borrowed item” and “borrowed on date” statements whose pairing is ambiguous. The note is a modeling pattern, not a mandatory universal schema.

**Topics versus classes.** [SKOS Reference, Recommendation 18 August 2009](https://www.w3.org/TR/2009/REC-skos-reference-20090818/), §§1.3, 3, 5, 8, distinguishes concepts in knowledge-organization schemes from formal domain classes. SKOS broader links are not themselves transitive; broaderTransitive supports their closure. Cycles are not automatically inconsistent. Thus a workshop can require an acyclic navigation tree as its own editorial policy without claiming SKOS already enforces that policy. A browsing term “Repair” and a class of repair events serve different purposes.

**Provenance.** [PROV-O, Recommendation 30 April 2013](https://www.w3.org/TR/2013/REC-prov-o-20130430/), §§3.1–3.3, supplies entities, activities, agents, derivation, attribution, and qualified relations. Example: an extracted weight claim derives from manual revision 4, was generated by extraction run 23, and reviewed by an editor. This is a modeled lineage, not proof that the weight is correct or that the claimed reviewer really reviewed it. Trust, access control, signatures, and conflict resolution need additional mechanisms.

**Reasoning budget.** [OWL 2 Profiles, second edition, 11 December 2012](https://www.w3.org/TR/owl2-profiles/), §§1–4, trades expressivity for computational properties: EL emphasizes large terminologies, QL supports query rewriting for large instance datasets, RL supports rule-oriented processing. None is a runtime-latency guarantee for an arbitrary implementation or workload. Select features from required questions, then test the chosen engine.

## F. Design exercises to preserve for the next stage

These are proposed learning tests, not tasks already run:

1. Construct two records with the same label but different item identities. Explain why merging is wrong.
2. Construct two identifiers for one manufacturer. Decide what evidence permits merging and how to reverse a mistaken merge.
3. Explain why a loan event is not adequately captured by a permanent borrower edge.
4. Model current manufacturer contact information and the number printed on a historical receipt without conflating them.
5. Show a missing field that is acceptable under an ontology but fails an input constraint.
6. Show an inference that follows from a mistaken domain declaration. Distinguish a valid inference from a correct real-world conclusion.
7. Keep contradictory source claims separate, with source and time, before deciding what to publish as the preferred value.
8. Test whether a query’s “not found” result means false, unknown, inaccessible, not yet ingested, or outside the dataset.
9. Explain how a cached read model is refreshed after a source fact changes.
10. State which questions the chosen representation deliberately cannot answer.

A useful supplementary guide is Noy and McGuinness’s [Ontology Development 101](https://protege.stanford.edu/publications/ontology_development/ontology101.pdf), particularly §3 on scope and competency questions, §4 on class design, and §6 on naming. It describes an iterative, application-dependent method; it is a historical frame-oriented tutorial, not current OWL/SHACL semantics. The PDF itself did not expose a publication date in the retrieved text, so a commonly cited 2001 date is not treated here as independently verified metadata.

# Part III. Evidence ledger and limits

All sources below were opened as full documents or document pages; relevant sections were inspected through extracted text/search. This does not mean every page of every book, course, or standard was exhaustively read. Failed routes are identified above. No paywall was bypassed.

| Source | Evidence used | Limits |
|---|---|---|
| DB System Concepts author site, March 2019 edition | Edition, ISBN, public supplements | Not full-book access |
| Book contents and preface | Exact chapter route, assumed background | Selection is our recommendation |
| CMU Fall 2025 schedule/syllabus | Lecture sequence, projects, prerequisites | No claim of watching every video |
| Codd 1970, §§1–2 | Data independence, redundancy, consistency | Historical original; mirror hosting |
| Chen 1976, §§2–3 | Conceptual levels, entities/relations, design | Historical notation; full paper via mirror |
| Davis et al. 1993 | Multiple roles of representation | Conceptual analysis, not benchmark |
| Gruber 1993, §§1–2 | Ontology as shared commitment/interface | No completeness or consensus guarantee |
| Web Architecture 2004 | Identifier allocation and collision | Governance still needed |
| RDF Concepts 2014 | Graph/dataset/blank-node scope | Does not specify storage or trust |
| RDFS 2014 | Domain/range inference | Not an input validator |
| OWL Primer 2012 | Open world; names need not differ | Informative introduction |
| OWL Direct Semantics 2012 | Existential/cardinality interpretation | Illustrations assume stated axioms |
| SHACL 2017 | Validation, entailment options, recursion limit | Engine/dialect choices matter |
| SPARQL Query 2013 | Dataset-relative absence tests | Absence is not world completeness |
| N-ary Relations 2006 | Relation-instance modeling pattern | Working Group Note, not mandatory rule |
| SKOS 2009 | Concept schemes, hierarchy semantics | Editorial constraints remain local |
| PROV-O 2013 | Lineage vocabulary | Provenance claims need verification |
| OWL Profiles 2012 | Expressivity/computation tradeoffs | No operational performance guarantee |
| RDF 1.2 April 2026 / SHACL 1.2 October 2026 | Current retrieved maturity labels | Evolving, not mature-baseline assumptions |
| KCS v6 2016, maintained landing page | Document-KB lifecycle framing | Practitioner method; no causal efficacy claim |
| Ontology Development 101 | Iterative scope and competency-question method | Historical tutorial; PDF date unverified |

## Open questions for choosing the next learning phase

- Is the goal conceptual literacy, designing a schema, building a DBMS, or operating a knowledge service? The materials differ sharply in effort and payoff.
- Which decisions must be backed by accepted records, and which may tolerate partial knowledge?
- Will the eventual system manage documents, structured assertions, or both? What are the authoritative sources for each?
- Who can declare two identifiers equivalent, and who can reverse that declaration?
- What must survive changes: identifiers, meanings, queries, document links, or all of them?
- Are inference, validation, and retrieval needed separately? Which precise examples justify each?
- What practical case study should follow the foundation: a transaction system, a document-support KB, or a public integrated graph? Do not pick one merely because it uses fashionable technology.

## Bottom line

The strongest initial plan is a small, coherent learning spine with explicit boundaries: textbook for the system, Chen for conceptual design, Codd for motivation, CMU for implementation costs, then a tightly scoped standards branch for meaning and interoperability. The preserved notes identify later questions; they do not commit the user to a particular architecture.
