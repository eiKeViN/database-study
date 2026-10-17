# Module 2: Source and evidence ledger

Read on 17 October 2026. This ledger records selected close readings, not full-book coverage. Companion lesson: [Transactions and derived views](02-transactions-and-derived-views.md).

## Repository baseline

- Repository tree inspected through GitHub: main commit `a054ce09e5a20800addb46e0c3b5790077178758`.
- [Module 1](https://github.com/eiKeViN/database-study/blob/a054ce09e5a20800addb46e0c3b5790077178758/notes/01-modeling-and-integrity.md): read the modeling assumptions, constraints, partial index, current-name view, and stated concurrency gap. Preserved account/item/occurrence identity and current-label semantics.
- [Synthesis](https://github.com/eiKeViN/database-study/blob/a054ce09e5a20800addb46e0c3b5790077178758/notes/synthesis.md): read the correctness/serving distinction, equipment-guide example, and bounded continuation. This module closes the foundational concurrency gap; it does not claim to complete the synthesis's separate EMERGE/Zanzibar reading proposals.

## Primary official documentation inspected

All links are pinned to PostgreSQL 18. These are implementation guarantees, not universal definitions for every database bearing the same isolation-level names.

| Source and selected passage | Evidence used | Important boundary |
|---|---|---|
| [§3.4 Transactions](https://www.postgresql.org/docs/18/tutorial-transactions.html), atomic grouping, BEGIN/COMMIT/ROLLBACK, implicit transactions | Database transaction boundary | No external-effect rollback or configured durability claim |
| [§13.2 Transaction Isolation](https://www.postgresql.org/docs/18/transaction-iso.html), ordinary SELECT snapshots, Repeatable Read snapshot establishment, Serializable outcome | Statement snapshot versus stable snapshot versus serial-equivalent committed outcome | Updating commands have additional rules; stable snapshot is insufficient for quota |
| [§63.5 Index Uniqueness Checks](https://www.postgresql.org/docs/18/index-unique-checks.html), uncommitted conflicting insert/delete cases | Wait and recheck conflict | Does not validate physical possession |
| [§11.8 Partial Indexes](https://www.postgresql.org/docs/18/indexes-partial.html), Example 11.3 | Uniqueness applies to predicate-selected rows | No historical-interval exclusion claim |
| [§13.3 Explicit Locking](https://www.postgresql.org/docs/18/explicit-locking.html), §§13.3.2 and 13.3.4 | Row locks, lifetime, deadlock possibility | Application protocol must cover all relevant writers |
| [§13.5 Serialization Failure Handling](https://www.postgresql.org/docs/18/mvcc-serialization-failure-handling.html), complete-transaction retry | Retry decision logic as well as SQL | Other error classes may be persistent failures |
| [CREATE VIEW](https://www.postgresql.org/docs/18/sql-createview.html), Description and Updatable Views | Query definition; join view not automatically writable | Read interface does not define historical-name semantics |
| [§39.3 Materialized Views](https://www.postgresql.org/docs/18/rules-materializedviews.html), opening definition and summary example | Persisted results and staleness | Source's performance example is not our benchmark |
| [REFRESH MATERIALIZED VIEW](https://www.postgresql.org/docs/18/sql-refreshmaterializedview.html), Description and CONCURRENTLY requirements | Full backing-query refresh, reader access, qualifying unique index | CONCURRENTLY is not incremental query evaluation |
| [§37.1 Trigger Behavior](https://www.postgresql.org/docs/18/trigger-definition.html), transaction membership and event behavior | Trigger changes participate in triggering transaction | Correct delta logic remains a design obligation |
| [§47.2 Logical Decoding Concepts](https://www.postgresql.org/docs/18/logicaldecoding-explanation.html), §§47.2.1–47.2.2 | Change stream; replay possible after crash | Client duplicate handling required; no end-to-end exactly-once claim |
| [§5.5.5 Foreign Keys](https://www.postgresql.org/docs/18/ddl-constraints.html#DDL-CONSTRAINTS-FK), deletion actions | Restrict/cascade/null actions have different effects | None selects an appropriate retention policy |

## Independently derived teaching material

- Same-item concurrent checkout: distinct loan keys do not enforce item exclusion; repeat the schedule with the inherited partial unique index.
- Cross-item quota: two decisions from count 1 produce 3; neither serial order of that program permits the second insertion.
- Account lock protocol: lock an existing stable row, then count in a separate Read Committed statement. Do not lock only an empty matching loan set.
- Incremental counts: remove old open contribution, add new open contribution; due-date corrections, returns, deletions and item reassignment have different deltas.
- Search dependencies: model rename fans out without Loan changes; stale updates can resurrect deleted documents without ordering/version protection.

These are original worked examples and proposed designs informed by the above semantics. They are not examples copied from the documentation or observed production outcomes.

## Verification status

- Read cited primary sections and existing repository baseline.
- Checked schedules and delta cases by reasoning.
- Did not install/run PostgreSQL, execute SQL, reproduce a race, measure performance, or provision a stack.
- The schedules and precision qualifications received a separate prose review; this is not an execution test.
