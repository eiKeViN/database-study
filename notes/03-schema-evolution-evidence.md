# Evidence ledger: schema evolution and stable identity

Checked 17 October 2026. Companion to [Module 3](03-schema-evolution-and-stable-identity.md). This is a targeted reading record, not a literature survey. No source is claimed to validate the complete hypothetical design.

## What was already covered

Read the existing public-study snapshot's README, synthesis, lifecycle note, and Module 1's modeling assumptions. The earlier material already distinguishes names from identity, representation from meaning, temporal questions, and expand/migrate/contract at a high level. Module 3 adds a single worked migration, explicit mixed-version contract, counterexample schedules, four distinct semantic operations, and recovery limits. It does not repeat the broad architecture survey or select a product stack.

## Primary sources and supported claims

### S1. GitLab: Avoiding downtime in migrations

- URL: https://docs.gitlab.com/development/database/avoiding_downtime_in_migrations/
- Type/version: living first-party operational documentation, consulted on the research date; not an immutable release snapshot.
- Inspected: “Dropping columns,” “Renaming columns,” and relevant dependency/trigger discussion.
- Supports: old processes and schema caches can depend on removed names; phased removal and dependent-object checks matter; the documented small-table rename procedure is not the large-backfill procedure.
- Does not establish: universal release counts, a complete bridge for this example, or measured downtime/error probability.
- Use: motivates inspecting every client and deployment phase. The hypothetical synchronization policy, race schedule, and compatibility matrix are this module's reasoning, not copied GitLab implementation instructions.

### S2. PostgreSQL 18: ALTER TABLE

- URL: https://www.postgresql.org/docs/18/sql-altertable.html
- Type/version: official, major-version-pinned reference; minor-version/current documentation updates remain possible.
- Inspected: ADD table_constraint [NOT VALID], VALIDATE CONSTRAINT, and associated locking notes.
- Supports: staged validation skips the initial existing-row check while enforcing eligible constraints on subsequent writes; later validation checks retained rows. Operations still acquire locks.
- Does not establish: zero lock wait, acceptable production latency, or semantic validity of transformed data.
- Use: distinguishes validation scheduling from permission to insert invalid new values. No PostgreSQL statement or trigger was executed here.

### S3. OBO Foundry Principle 19: Stability of Term Meaning

- URL: https://obofoundry.org/principles/fp-019-term-stability.html
- Type/version: living normative community policy, consulted on the research date.
- Inspected: summary; recommendations; obsoletion implementation; counterexample.
- Supports: substantial referent changes require a new IRI; same-referent clarification is permitted; obsoletion removes logical use; exact replacement and inexact consideration are distinct guidance.
- Does not establish: an algorithm deciding equivalence, safe automatic reannotation of every external record, or a universal relational implementation.
- Use: tests whether a proposed semantic mutation is being mislabeled a rename. The equipment cases are invented and independently reasoned.

### S4. Gene Ontology overview

- URL: https://geneontology.org/docs/ontology-documentation/
- Type/version: living first-party documentation.
- Inspected: “GO term elements,” especially mandatory elements, alternate IDs, synonyms, and obsolete tag; “GO is a dynamic ontology.”
- Supports: accession, name, and definition are distinct; obsolete IDs persist with explanation while relationships are removed; alternate IDs document historical identical-meaning merges.
- Important limit: its description of alternate IDs does not itself establish that new merges are current practice. S5 explicitly narrows that interpretation.
- Does not establish: complete historical replay from a tombstone alone or correct automatic resolution of a split.

### S5. Gene Ontology Consortium: The Gene Ontology knowledgebase in 2026

- DOI: https://doi.org/10.1093/nar/gkaf1292
- Full-text PDF inspected: https://api.repository.cam.ac.uk/server/api/core/bitstreams/58da22a4-1d97-49c3-969b-b592daa53392/content
- Type/version: consortium-authored research update, titled 2026; inspected PDF is headed Nucleic Acids Research 2025, gkaf1292 and reports acceptance on 24 October 2025. Do not infer its publication year solely from its title.
- Inspected: opening update sections on printed page 2, plus annotation-update passage.
- Supports: release citation for reproducibility; the report that GO stopped merging terms in 2022 and uses obsoletion/replacement; review of affected annotations.
- Reconciliation: distinguishes historical alternate-ID representation (S4) from reported later procedure. Neither account is silently discarded.
- Limitation: existing synthesis already flags the paper's inconsistent comparison-date prose/link. This module deliberately uses no growth or obsoletion totals and makes no inferred quantitative comparison window.
- Access note: the institutional public PDF supplied the inspected full text.

## Original reasoning, not source findings

1. **Compatibility is a relation among versions and states.** A scalar “migration succeeded” cannot distinguish A+E from A+contracted-schema. The compatibility matrix follows from the stipulated field dependencies.
2. **Synchronization and freshness are separate.** An atomic bridge can faithfully copy a stale worker's write; therefore atomicity alone cannot prevent a backfill from reverting a newer edit. The three-event schedule supplies a constructive counterexample.
3. **Dataset agreement does not establish definition equivalence.** The all-battery sample is consistent with two different predicates. Adding a mains-powered portable drill separates them without any change to SQL validity.
4. **Split migration has an information limit.** An old broad assertion supports neither specific child by itself. Choosing one or both adds unsupported content; explicit unresolved status preserves the known information.
5. **Preserved identity does not preserve historical query semantics.** Changed hierarchy edges affect descendant queries even if IDs remain stable. A historical contract must select relation/mapping versions as well as records.
6. **Reversal depends on retained distinctions.** A many-to-one rewrite cannot recover original sources after those distinctions are discarded. Keeping decision and assertion history makes reversal more plausible, not guaranteed.

These are deductions under the module's assumptions. They are not claims that the literature lacks these ideas, nor empirical findings that this proposed design works in production.

## Verification status

- Files written and checked for basic integrity; no publication performed.
- Conceptual and source review completed on 17 October 2026. The example explicitly requires observable write intent or a pre-backfill write gate and includes a reasoned optimistic-version schedule. No execution was performed.
- No SQL execution, migration load test, benchmark, ontology snapshot replay, or measured reliability result.
