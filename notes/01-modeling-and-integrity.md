# Module 1: Modeling and integrity before technology selection

16 October 2026. A bounded close-reading lesson, using an invented community-workshop lending service. No application to mathematics, no production implementation, and no performance benchmark.

## 中文要点

- 先说清“记录什么事实、这些事实在什么时间成立”，再决定表、主键和索引。
- 函数依赖是对所有合法数据状态的业务约束，不是从几行样本里猜出的规律。
- 无损分解保证原有事实可重建；依赖保持保证原有函数依赖可由分表后的局部约束共同保证。两者不是同一个性质。
- 本例可以同时做到 BCNF、无损、依赖保持；不能因此推断所有问题都能兼得。
- 规范化不会解决实体认错、词义变化、来源冲突或历史语义。SQL 约束也不会验证现实世界的真实性。

## 1. What the close reading establishes

**Codd’s motivation.** In [Codd 1970, §§1.1–1.2](https://db.dobo.sk/wp-content/uploads/2015/11/Codd_1970_A_relational_model.pdf), the concern is independence from ordering, indexing, and access paths, not merely displaying tables. His §1.4 normalization removes nested/non-simple domains; it should not be presented as the entire later theory of 3NF and BCNF. Sections 2.2–2.3 distinguish redundancy, consistency, and physical representation. The article is the original primary text, accessed through a public mirror; [publication DOI](https://doi.org/10.1145/362384.362685).

**Chen’s conceptual contribution.** [Chen 1976, §§2.1–2.3 and 3.1–3.2](https://weichselbraun.net/dbs/pdf/literature/chen.pdf) separates domain entities/relationships from their representation and access paths. Roles and relationship attributes carry meaning. Identification may require one attribute, a combination, or another entity’s identifying relationship. This is an original paper, not a modern notation manual; [publication DOI](https://doi.org/10.1145/320434.320440).

**Modern teaching bridge.** The official [Database System Concepts 7e chapter 6 slides](https://www.db-book.com/slides-dir/PDF-dir/ch6.pdf) distinguish conceptual, logical, and physical design (6.4–6.7), then discuss cardinalities, participation, keys, and translation to relations (6.24–6.51). The PDF title metadata incorrectly says chapter 7 in the retrieval output; its visible first-page text and contents identify chapter 6. We follow the contents.

**Structural definitions.** A functional dependency X→Y means equal X values require equal Y values in every legal relation instance. A candidate key is a minimal set determining all attributes. BCNF requires every nontrivial dependency’s determinant to be a superkey. Precisely, 3NF requires that for every implied nontrivial dependency X→A with a single attribute on the right, X is a superkey or A is prime (belongs to some candidate key). Binary decomposition is lossless when the shared attributes determine one component; dependency preservation concerns what all projected dependencies together imply. These definitions and results appear in the official [chapter 7 slides](https://db-book.com/slides-dir/PDF-dir/ch7.pdf), especially 7.13–7.18, 7.23–7.29, and 7.55–7.57. The following applications are worked out independently for our example.

## 2. Decide what one fact means

Our workshop lends physical tools. We make these explicit design assumptions:

1. Each loan occurrence concerns exactly one physical item and one borrower account.
2. An item belongs to one product model. This membership is stable for this simplified module.
3. Each model has one current display name; each borrower account has one current display name. Names are not unique.
4. A loan has an identifier, checkout time, and due time. The same borrower can borrow the same item repeatedly.
5. A borrower account is our entity; deduplicating real people across accounts is outside this model.
6. The initial proof concerns one snapshot of null-free, set-valued relations. Historical names, absent values, SQL duplicate rows, and optional returns are addressed separately.

Assumption 3 allows a name to change between database states. It does not require one timeless name. These are proposed requirements, not facts supplied by a real workshop.

**Conceptual E-R view.** Item, ProductModel, BorrowerAccount, and LoanOccurrence are the relevant entity types. LoanOccurrence participates with exactly one item and one account; an item or account can have many loan occurrences. A simple borrower–item association would collapse repeat loans between the same pair. Representing the occurrence explicitly preserves them.

**Logical view.** Choose attributes, relations, keys, and constraints implementing those facts.

**Physical view.** Choose indexes and access paths supporting actual operations. A B-tree is not a new kind of item, and changing an index should not change which loans occurred.

## 3. Begin with a plausible bad table

LoanSheet(loan_id, item_id, model_id, model_name, borrower_id, borrower_name, checked_out_at, due_at)

Example rows, with dates abbreviated within one year:

| Loan | Item | Model | Current model name | Borrower | Current borrower name | Out | Due |
|---|---|---|---|---|---|---|---|
| L1 | I1 | M1 | Cordless drill | B1 | Ana | Oct 1 | Oct 3 |
| L2 | I1 | M1 | Cordless drill | B2 | Ben | Oct 5 | Oct 7 |
| L3 | I2 | M1 | Cordless drill | B1 | Ana | Oct 8 | Oct 10 |

Anomalies under the stated meaning:

- **Update:** changing M1’s current display name requires three rows to agree. Changing only L1 leaves one model with conflicting current names.
- **Insertion:** a new item with no loan cannot be represented without fabricating or omitting loan data.
- **Deletion:** deleting L3, the only recorded loan of I2, also removes the only representation of I2 from this table.

The error is mixing independently maintained facts into a loan-shaped record. Repeated text alone is not the definition of a problem: a name printed on a historical receipt could legitimately be copied and retained as an event-specific fact.

## 4. Derive keys from declared dependencies

Use these dependencies, with no additional unstated uniqueness rules:

- loan_id → item_id, borrower_id, checked_out_at, due_at
- item_id → model_id
- model_id → model_name
- borrower_id → borrower_name

Starting from loan_id yields the whole row: its item gives the model and model name; its borrower gives the borrower name. Thus loan_id is a candidate key. Every key must contain loan_id because none of the other listed dependencies can derive it. It is the only candidate key under this dependency set.

item_id is not a loan key: L1 and L2 share it. borrower_name is not an account key merely because this sample contains two distinct names. Even (item_id, borrower_id, checked_out_at) is not assumed unique: clock precision, import errors, and event rules need a deliberate policy before that combination becomes a key.

Choosing a surrogate loan_id does not make the other dependencies disappear. It gives each row a handle; model_name can still disagree across rows for the same model.

## 5. A decomposition with two separate guarantees

Proposed relations:

- ProductModel(model_id, model_name), key model_id
- Item(item_id, model_id), key item_id
- BorrowerAccount(borrower_id, borrower_name), key borrower_id
- Loan(loan_id, item_id, borrower_id, checked_out_at, due_at), key loan_id

**Why it is lossless for the original represented facts.** Split model_id/model_name from LoanSheet, retaining model_id in the remainder. The shared model_id determines the complete model component. Then split borrower_id/borrower_name, whose shared borrower_id determines that component. Finally split item_id/model_id, whose shared item_id determines the item component. Each binary step satisfies the lossless criterion; performing them successively preserves reconstruction of every legal original LoanSheet instance.

A direct argument makes the guarantee concrete. Each original loan finds exactly its model name through its item and exactly its account name through its borrower. Shared identifiers cannot introduce a second competing name in a legal instance. Joining the projections neither loses a loan nor invents a different combination for it.

**Why it preserves dependencies.** Each declared dependency is now checkable wholly inside one relation. Their combined implications still include all dependencies originally listed. This is a logical property; it is not a measured statement about validation latency.

**Why each component is BCNF here.** In each component every nontrivial dependency implied by the stipulated set has a determinant containing that component’s key. This conclusion is relative to the declared dependency set; additional real business rules can change the analysis.

**Important boundary.** The proof starts from projections of an existing valid LoanSheet. It does not prove that arbitrarily edited separate tables will remain mutually consistent: references must be maintained. The new schema can additionally represent an item with no loan, something the original table could not express. This extension is a benefit of the conceptual redesign, not a fact recovered by the lossless-join theorem.

## 6. Two ways to decompose badly

### A. Lose associations, even though all column values survive

For loan–borrower–item facts, suppose we retain only LoanBorrower(loan_id, borrower_id) and BorrowerItem(borrower_id, item_id).

From L1/B1/I1 and L3/B1/I2, the projections join into four rows: the two correct rows plus L1/B1/I2 and L3/B1/I1. Values have survived, but the original pairings have not. “Lossy” can therefore mean spurious reconstructed facts, not just missing rows.

### B. Preserve reconstruction but lose local enforcement

Consider the smaller schema LoanItemModel(L, I, M) with L→I and I→M. Decompose it into A(L,I) and B(L,M).

This decomposition is lossless: common L determines both components. Both components are BCNF. But their local dependencies do not enforce I→M.

Counterexample after separate edits:

- A contains (L1,I1) and (L2,I1).
- B contains (L1,M1) and (L2,M2).

Each table has unique L. Their join gives the same item two models, violating the original rule. Matching the loan identifiers between tables does not fix this. Enforcement requires a cross-table condition or a better representation. The preferred decomposition A(L,I), Item(I,M) preserves the rule locally and is also lossless.

This example shows a bad decomposition choice, not an unavoidable tradeoff in the main lending schema.

## 7. Where a genuine BCNF tradeoff can appear

A small extension in the same workshop illustrates why 3NF is not simply a mistake. Suppose TrainingAssignment(borrower, skill, trainer) has exactly these rules:

- trainer → skill: every trainer teaches one skill in this simplified program.
- borrower, skill → trainer: each borrower has at most one assigned trainer per skill.

Candidate keys are (borrower, skill) and (borrower, trainer). Every attribute belongs to a candidate key. The relation is 3NF but not BCNF because trainer determines skill without determining borrower.

Splitting into TrainerSkill(trainer, skill) and BorrowerTrainer(borrower, trainer) is lossless and BCNF. However, assigning B1 to T1 and T2, with both trainers teaching S1, satisfies each component’s local dependencies yet violates the original one-trainer-per-borrower-skill rule. No proper subrelation contains all three attributes needed for that rule; the projected dependencies cannot recover it.

The choices are to retain the 3NF relation and enforce its dependencies, or decompose and explicitly maintain the cross-relation rule. Dependency preservation is about local FD entailment, not an assurance that each FD has a single convenient SQL keyword. This illustrates the general BCNF/dependency-preservation limitation discussed in chapter 7, rather than a recommendation to force this artificial training policy onto a real organization.

## 8. Translate into PostgreSQL constraints carefully

Documentation baseline: PostgreSQL **18**, retrieved 16 October 2026. These are documented behaviors, not results of executing this schema.

Suggested implementation choices:

- PRIMARY KEY on each chosen identifier.
- NOT NULL on required model_id, item_id, borrower_id, model_name, borrower_name, checkout and due times, preserving the mandatory-name assumptions.
- REFERENCES from Item to ProductModel and from Loan to Item and BorrowerAccount.
- CHECK (due_at >= checked_out_at).
- When returned_at is introduced, CHECK (returned_at IS NULL OR returned_at >= checked_out_at).

[PostgreSQL §5.5](https://www.postgresql.org/docs/18/ddl-constraints.html) documents that CHECK accepts true or null, so it cannot replace NOT NULL. A nullable foreign key can avoid a required match. Ordinary CHECK is not a supported general mechanism for maintaining cross-row/table conditions. Primary keys enforce uniqueness and non-nullness; ordinary UNIQUE treats nulls as distinct by default unless configured otherwise.

The dependency item_id→model_id and Item’s primary key constrain each snapshot to one model per item. They do not prohibit changing an item’s model_id between snapshots. The stable-membership assumption in §2 is an additional temporal policy; enforcing it would require a separate update policy or history-aware design.

To express our additional policy of at most one open loan per item, an illustrative PostgreSQL statement is:

    CREATE UNIQUE INDEX one_open_loan_per_item
        ON loan (item_id) WHERE returned_at IS NULL;

This relies on item_id being non-null and on “open” meaning exactly the absence of a recorded return. [Partial-index documentation §11.8](https://www.postgresql.org/docs/18/indexes-partial.html) specifies uniqueness over the predicate-selected subset. This does not prohibit overlapping historical intervals or prove physical possession. Such rules need separate design. The index here implements a logical invariant, so removing it is not a purely performance-neutral decision.

## 9. Views and indexes restore convenience without erasing meaning

A joined LoanDisplay view can expose model_name and borrower_name next to each loan without storing repeated current names in Loan. For example, join Loan→Item→ProductModel and Loan→BorrowerAccount using their identifiers. Use explicit selected columns and intentional names such as current_borrower_name.

[PostgreSQL CREATE VIEW](https://www.postgresql.org/docs/18/sql-createview.html) describes ordinary views as stored query definitions rather than materialized results. A join view of this kind is not automatically writable under the simple-view rules. Consequently, preserving a reading interface does not automatically preserve an old writing interface. We must define which base fact an edit changes. A CHECK OPTION on an eligible updatable view is a further condition, not a replacement for base-table integrity.

For performance, an index on Loan(borrower_id, checked_out_at) is a hypothesis for the query “show an account’s loans in time order.” Its benefit depends on sizes, selectivity, ordering, cache state, and competing writes. [PostgreSQL index introduction §11.1](https://www.postgresql.org/docs/18/indexes-intro.html) describes potential search benefits and index-maintenance overhead. There is no measured claim here that this index is faster or that normalization is faster/slower overall.

Data independence means preserving promised meaning/interfaces while changing representation where possible. It is not permission to change a view from historical names to current names while keeping the same column spelling.

## 10. What normalization cannot decide

These conclusions follow from examining the example’s assumptions:

- **Identity errors:** two account IDs for the same person can satisfy every key and FD.
- **Wrong merges:** two physical drills accidentally assigned one ID cannot be repaired by decomposition.
- **Changing meanings:** model_name intended as “name at checkout” requires event/version semantics. Updating the current model table would change the wrong fact.
- **Source conflicts:** two manuals disagreeing on a specification need source-qualified assertions and an adjudication policy, not a forced overwrite justified by an FD.
- **Missing coverage:** a valid database need not contain every physical item or unreported return.
- **Workflow/concurrency correctness:** an application-side “check then insert” is not equivalent to an enforced invariant under simultaneous writers. This module identifies the need; it does not analyze isolation algorithms.
- **Security and usefulness:** normal forms do not decide who should see borrower information or whether a search result answers a user’s question.

## 11. Correctness review and stopping point

**Theorem-backed, conditional on stated dependencies:** main decomposition losslessness, FD preservation, BCNF; candidate-key conclusions; the training example’s 3NF/BCNF distinction.

**Concrete counterexamples:** spurious loan/item pairings; loss of item→model enforcement; two trainers for one borrower/skill. These demonstrate failure without relying on a performance experiment.

**Design judgments:** treating loans as occurrences, keeping current labels separate, selecting account rather than person identity, and defining “open” by null returned_at. A real stakeholder could choose different semantics.

**Verification performed:** explicit attribute-closure calculations confirmed loan_id determines every original column; item_id determines only item/model/name; and the two training keys determine all training attributes. Proofs and counterexamples were checked in prose. No SQL engine was installed or run, and no benchmark was performed.

**Source-reading caution:** chapter 6 slide 6.36 has wording that broadly suggests adding relationship attributes to a primary key. Do not mechanically turn every descriptive attribute into key material. Apply uniqueness and minimality to the intended occurrence semantics; a due-date change should not silently become a new loan identity. The source’s neighboring mapping-cardinality discussion and chapter 7 key definition are the more precise guides.

**Next exercise, not performed:** draw the conceptual diagram, implement the four relations with test data in a disposable database, and deliberately attempt each rejected state. This module stops at the reviewed lesson and proposed schema. It does not proceed to knowledge-graph standards or frontier systems.

## Reading log

- Codd, June 1970, CACM 13(6), 377–387: full paper available; close reading of §§1.1–1.4 and 2.2–2.3. Historical motivations, not attribution of later BCNF theory.
- Chen, March 1976, TODS 1(1), 9–36: full paper available; close reading of §§2.1–2.3 and 3.1–3.2. Conceptual distinctions and mappings.
- Database System Concepts 7e: official chapter 6 and 7 slides. Author index dates: chapter 6, 13 February 2020; chapter 7, 24 January 2019. Relevant slide ranges identified above; not a claim to have read the full textbook.
- PostgreSQL 18: §§5.5, 11.1, 11.8, CREATE VIEW. Version-pinned documentation, not a claim about every SQL implementation.

No measured empirical result is asserted in this lesson. All workshop data and business rules are invented teaching examples.
