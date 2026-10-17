# Module 2: Transactions and derived views

17 October 2026. A close-reading continuation of [Module 1](01-modeling-and-integrity.md), using the same invented workshop lending service. PostgreSQL 18 supplies a concrete semantics baseline, not a product recommendation. All schedules and maintenance examples below are reasoned examples; no SQL engine, concurrent sessions, or benchmark was run.

## 1. Carry the meaning forward

Keep ProductModel, Item, BorrowerAccount, and Loan from Module 1. An item is a physical tool, an account is not necessarily a unique real person, and a loan is an occurrence. Names remain current labels. Add nullable returned_at to Loan; an open loan means exactly returned_at IS NULL. Required identifiers remain non-null. Preserve the primary keys, foreign keys, and timestamp checks.

The existing additional invariant is:

    CREATE UNIQUE INDEX one_open_loan_per_item
        ON loan (item_id) WHERE returned_at IS NULL;

This lesson asks three different questions:

1. Which represented states are allowed?
2. Which concurrent decisions may commit together?
3. Which committed version does a reader actually see?

A correct answer to one question does not answer the others. Nothing here establishes whether a tool was physically returned, whether an account was assigned to the right person, or whether a staff member entered the right due date.

## 2. A transaction has a boundary

The [transaction tutorial, §3.4](https://www.postgresql.org/docs/18/tutorial-transactions.html) establishes atomic grouping, commit, rollback, and implicit per-statement transactions. For our workflow, returning I1 and issuing its next loan can be one transaction: either both database changes commit or neither does. This choice prevents a crash between separately committed steps from leaving only the return recorded.

Atomicity does not mean every reader immediately adopts the new state, or that an email and a physical handover roll back with the database. The operation should report success only after successful commit. External effects need their own delivery/reconciliation policy. This module does not establish a durability configuration or disaster-recovery guarantee.

Consider a return-and-reissue operation with old loan L1 and new loan L2. Update L1.returned_at before inserting L2. If insertion fails, rolling back the entire transaction restores L1 as open. The timestamp check must still accept the supplied return time. Reversing those statements would temporarily propose two open loans and encounter the immediate unique-index check. Statement order therefore matters even when the intended final state is valid.

## 3. Close reading: isolation is a specified visibility contract

From [§13.2, especially §§13.2.1–13.2.3](https://www.postgresql.org/docs/18/transaction-iso.html):

- **Read Committed:** an ordinary SELECT uses a statement-start snapshot, plus its own transaction's changes. Successive statements can see different committed states.
- **Repeatable Read in PostgreSQL:** the snapshot is established by the first non-transaction-control statement. It excludes later concurrent commits, but includes the transaction's own changes. Phantom reads are prevented; serialization anomalies remain possible. This is stronger than the standard's minimum Repeatable Read requirements.
- **Serializable:** successfully committed participating transactions have an effect equivalent to some serial execution. A transaction can instead fail and require retry.

The important reading correction is that a stable snapshot is not proof of a valid combined decision. Also, do not extend ordinary SELECT semantics indiscriminately to updating commands: Read Committed writers can wait and re-evaluate a target row after a concurrent update.

## 4. Two borrowers race for one drill

Initially I1 has no open loan. Assume distinct loan identifiers and otherwise valid input. The unprotected failure schedule uses Read Committed or PostgreSQL Repeatable Read, not Serializable.

| Step | Transaction A | Transaction B |
|---|---|---|
| 1 | Read open loans for I1: none | |
| 2 | | Read open loans for I1: none |
| 3 | Insert LA for I1 | |
| 4 | | Try inserting LB for I1 |
| 5 | Commit | |

**Without the partial unique index**, both decisions could insert separate rows. Neither the distinct loan primary keys nor foreign keys express the intended exclusion. Putting each check and insert inside BEGIN/COMMIT does not, by itself, repair this schedule.

**With the index**, B's conflict cannot be validated solely against its earlier read. [Index uniqueness checks, §63.5](https://www.postgresql.org/docs/18/index-unique-checks.html) explicitly waits for an uncommitted conflicting insertion and rechecks its outcome. If A commits and its open loan remains, B's insertion fails. If A rolls back, that conflict disappears. The winner is not promised in advance.

This is a concrete difference between advisory availability and acceptance. The page may truthfully have displayed I1 as available when read; another borrower can win before checkout. Return an appropriate conflict response, rather than treating the old display as a reservation. A database error is not a second successful loan.

The index covers records selected by its predicate. A mistaken non-null return time can release the item in represented state. Historical interval overlap and actual possession remain outside this guarantee.

## 5. An invariant that the index cannot express

Add a clearly separate teaching policy: an account may have at most two open loans. B1 currently has one. I2 and I3 are free. Again, the failure schedule assumes Read Committed or PostgreSQL Repeatable Read.

| Step | Transaction C | Transaction D |
|---|---|---|
| 1 | Count B1's open loans: 1 | |
| 2 | | Count B1's open loans: 1 |
| 3 | Insert LC: B1 borrows I2 | |
| 4 | | Insert LD: B1 borrows I3 |
| 5 | Commit | Commit |

Each transaction individually reasons that 1 + 1 is allowed. Their writes concern different items and loan rows, so the per-item unique index offers no conflict. The resulting count is 3. In either serial ordering of the stated check-then-insert program, the second transaction would see 2 and decline. Thus this outcome is not equivalent to either serial ordering. This independently worked counterexample explains the serialization anomaly rather than identifying it with a dirty read.

Two candidate designs follow; they are alternatives to evaluate, not mandatory layers:

**A. Serialize the decision around the account.** At Read Committed, first lock the existing BorrowerAccount row for B1 using SELECT ... FOR UPDATE. After that lock has been acquired, run a separate count statement, decide, and insert if permitted. All operations capable of increasing B1's open-loan count must follow that protocol, including imports, borrower reassignment, and reopening returned loans. Keep the lock until commit. The separate post-lock count matters: do not fold it into an earlier statement snapshot.

[Row-level locking, §13.3.2](https://www.postgresql.org/docs/18/explicit-locking.html#LOCKING-ROWS) documents conflicting row locks and their transaction lifetime. Locking matching Loan rows alone is insufficient when the set is empty; an absent row is not an account-level mutex. For multi-account operations, choose a common lock order; deadlocks still need handling (§13.3.4). This protocol is cooperative, whereas the unique index directly rejects conflicting item records.

**B. Use Serializable for every relevant decision and mutation.** Keep the count and conditional insertion in one transaction. Their combined outcome must satisfy a serial ordering or a transaction must fail. PostgreSQL's Serializable Snapshot Isolation monitors dependencies; its SIRead predicate locks detect conflicts rather than blocking like ordinary row locks (§13.2.3). Retain the item constraint as an explicit invariant. The serial program itself must enforce the quota; Serializable cannot invent a missing business check. This argument assumes all transactions participating in this rule use the required isolation/protocol.

[Serialization-failure handling, §13.5](https://www.postgresql.org/docs/18/mvcc-serialization-failure-handling.html) requires retrying the complete transaction, including the logic that chooses its SQL and values. Reusing the failed attempt's old count defeats the purpose. Classify a genuine item conflict separately from a retryable serialization failure. Retries need a policy for repeated contention and must not repeat already-performed external effects blindly.

## 6. An ordinary view is a reading interface

A useful read interface can be defined as follows (illustrative SQL, not executed):

    CREATE VIEW loan_display AS
    SELECT l.loan_id, l.item_id, l.borrower_id,
           m.model_name AS current_model_name,
           b.borrower_name AS current_borrower_name,
           l.checked_out_at, l.due_at, l.returned_at
    FROM loan AS l
    JOIN item AS i ON i.item_id = l.item_id
    JOIN productmodel AS m ON m.model_id = i.model_id
    JOIN borroweraccount AS b ON b.borrower_id = l.borrower_id;

Unquoted SQL identifiers above are lower-case versions of Module 1's conceptual names; an eventual implementation may choose different physical spellings.

[CREATE VIEW, Description and Updatable Views](https://www.postgresql.org/docs/18/sql-createview.html) distinguishes a stored query from stored results. This multi-table join is not automatically updatable. An edit to current_borrower_name means changing an account label, not one loan's historical receipt. Explicit write operations should make that distinction clear.

Reason through two reads: a clerk reads L1's display, another transaction renames B1 and commits, then the clerk reads again. A later Read Committed statement can display the new name; the same Repeatable Read snapshot can retain the old one. Neither behavior means the ordinary view has a refresh job. Its visibility comes from the reading transaction.

Likewise, an ordinary view is not a promise that the answer remains current after it reaches a browser. A rendering can age while the user thinks. Borrowing still requires an authoritative write decision.

## 7. A materialized view introduces a second maintained state

Suppose a dashboard stores open-loan counts by model. [Materialized views, §39.3](https://www.postgresql.org/docs/18/rules-materializedviews.html) stores query results and reads those stored rows; the defining query is used to populate them. Consequently, a base-table commit need not change the displayed count until maintenance occurs.

In our example, a refresh records M1 with five open loans. A later return commits, leaving four authoritative open loans. The dashboard can still show five. This is coherent historical output if its freshness contract permits it; it is unsuitable as the sole checkout authority.

[REFRESH MATERIALIZED VIEW](https://www.postgresql.org/docs/18/sql-refreshmaterializedview.html) reruns the backing query and replaces the contents. CONCURRENTLY permits concurrent SELECTs, requires an already-populated view and a suitable all-row, column-only UNIQUE index, and still allows only one refresh at a time per view. It does not mean continuously maintained or incremental query evaluation. WITH NO DATA makes the view unscannable, not a conveniently empty current result.

Record refresh status and a meaningful source-version/snapshot boundary if the application requires one. A job completion timestamp alone does not prove all commits through that instant are included. Separate materialized views refreshed independently can represent different source states; joining them does not create a common release automatically. These are design consequences of choosing independently maintained copies.

## 8. Work through incremental maintenance instead of saying “sync”

Define a hypothetical maintained table ModelOpenCount(model_id, open_count), with a row for each model. This is a design exercise, not native REFRESH syntax. Assume initially correct counts, stable Item→ProductModel membership, reliable access to old and new loan values, and serialized or atomic counter changes.

For each changed loan, remove its **old** contribution if it was open, then add its **new** contribution if it is open. The affected model is obtained from the corresponding item. Independently derived cases:

| Change | Required contribution change |
|---|---|
| Insert an open loan for M1 | Add 1 to M1 |
| Record its return | Subtract 1 from M1 |
| Correct only its due time | No count change |
| Delete an already-returned loan | No count change |
| Delete an open loan | Subtract 1 from its model |
| Correct open loan's item from M1 to M2 | Subtract 1 from M1; add 1 to M2 |
| Reopen a returned loan | Add 1, subject to the item and quota rules |

If old and new contributions target the same model, their net change may be zero. Avoid turning one update into an extra loan. An aggregate using MIN(due_at) needs more information when the earliest loan disappears: subtracting its old minimum is meaningless. Either retain enough supporting state or recompute the affected group. Incremental does not mean every query admits a one-counter update.

**Synchronous option.** Maintain this summary in the same database transaction as Loan. [Trigger behavior, §37.1](https://www.postgresql.org/docs/18/trigger-definition.html) confirms a trigger executes within its triggering transaction, so database changes roll back together on error. A trigger is a mechanism, not a proof: cover INSERT, UPDATE and DELETE, correct old/new membership, bulk operations, and concurrent updates. For this proposed protocol, prohibit TRUNCATE and other maintenance paths that bypass those updates unless a separate summary-repair mechanism covers them. A naive read-count/write-count procedure can itself lose updates. A group-row update such as count = count + delta needs appropriate initialization and concurrency handling.

**Asynchronous option.** Apply committed changes later. [Logical decoding concepts, §§47.2.1–47.2.2](https://www.postgresql.org/docs/18/logicaldecoding-explanation.html) documents a change stream and possible replay after a crash. It explicitly places duplicate-handling responsibility on clients. Transport progress alone does not prove an application projection is correct.

Our design therefore needs an atomic relationship between applying a change and recording consumption, or another sound deduplication protocol. Applied twice, a +1 delta gives the wrong count. Marked consumed before application, a crash loses it. Applied before an unrecorded checkpoint, a crash can repeat it. A periodic comparison against a fresh authoritative recomputation can detect drift; repair still needs a controlled snapshot and replay boundary.

## 9. Search adds dependency and deletion questions

Imagine a search document for each loan, containing its item, model label, and account label. This is hypothetical; no search engine is selected.

- Renaming M1 affects every document derived through its items, although no Loan row changes. Listening only for Loan events misses this dependency.
- Returning L1 changes its open/returned status. Deleting L1 requires removal, not merely ignoring future updates.
- If an old document update arrives after a delete, it can resurrect L1. A version-aware tombstone or equivalent ordering policy must reject stale updates. Tombstone retention must cover the replay/reordering assumptions; keeping it for an arbitrary short time is not a proof.
- If a loan is mistakenly reassigned, both the old and new account's derived lists/counts need repair.
- A delayed copy containing an account name is a disclosure issue if that copy should no longer be served. Decide whether serving must be blocked while cleanup completes; a database deletion alone does not certify all copies removed.

For an external projection, define what “updated” acknowledges: source commit, change capture, application, query visibility, or client cache refresh. Do not collapse those milestones into one success flag. An acceptable stale dashboard and an authoritative checkout operation can have deliberately different contracts.

## 10. Deletion is a domain decision before it is SQL

[Foreign-key actions, §5.5.5](https://www.postgresql.org/docs/18/ddl-constraints.html#DDL-CONSTRAINTS-FK) distinguishes NO ACTION, RESTRICT, CASCADE, and SET NULL/DEFAULT. Cascading an Item deletion into Loan would erase represented loan occurrences. Setting a required borrower reference to null conflicts with its NOT NULL requirement. Neither is automatically the right meaning of “remove account” or “retire drill.”

For this teaching model, propose retiring an item through an explicit status while retaining permitted loan history, rather than deleting the item as a synonym for unavailable. That is a new policy requiring new schema and checkout checks, not a capability already proved here. Account erasure likewise requires a deliberate decision about permitted historical retention and all derived copies. No legal compliance claim is made.

## 11. What has been established, and what remains to test

**Established by reasoning:** a per-item constraint does not imply a per-account quota; the two successful quota decisions have no equivalent serial ordering; an open-loan delta must consider both old and new membership; a model rename has dependencies beyond changed Loan rows.

**Documented, version-specific:** uniqueness conflict handling, the specified isolation levels, row-lock behavior, ordinary/materialized-view behavior, transactional triggers, and decoding replay.

**Proposed, not implemented:** lock protocol, summary maintenance, search dependency tracking, freshness/version markers, deduplication, and retirement policy. No performance ranking follows from this lesson.

A bounded next exercise is a disposable two-session test harness, not further tool selection:

1. Run the I1 race with and without the unique index; inspect both outcomes and committed rows.
2. Run the quota schedule with different items under Read Committed, Repeatable Read, and Serializable; log every statement, error, retry, and final count.
3. Test the account-lock protocol with an intentionally bypassing writer to expose its cooperative boundary.
4. Read an ordinary view across a rename at both snapshot scopes, then contrast a deliberately unrefreshed materialized view.
5. Replay insert, return, correction, delete, duplicate delivery, and delayed pre-delete update; compare the maintained result to recomputation at the same declared source boundary.

Stop when the observations explain the guarantees and their limits. These tests are specified here but have not been executed.

## Source and reading record

See the [source and evidence ledger](02-source-evidence-ledger.md) for inspected PostgreSQL sections, evidence boundaries, and verification status.
