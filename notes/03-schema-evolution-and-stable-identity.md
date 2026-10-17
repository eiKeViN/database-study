# Module 3: Schema evolution and stable identity

Research cut: 17 October 2026. A worked study module, not an implemented migration or production runbook. The equipment catalog is invented. Source-backed observations are labeled; the design and counterexamples are original exercises. SQL, concurrency behavior, deployment, and recovery have **not been executed**.

This continues [Modeling and integrity](01-modeling-and-integrity.md) and makes the migration advice in [Lifecycle engineering](lifecycle.md) concrete. The new question is not whether a snapshot is normalized: **can old and new readers coexist without changing the meaning of existing assertions?**

## 1. Three promises that can fail independently

**Reasoning approach.** Start with the question, “What information must survive for a supported reader to give a justified answer?” Compare three alternatives: overwrite in place, keep two synchronized representations, or preserve old assertions and publish a new interpretation. The following examples choose among them by constructing failures. Sources test the boundaries of the reasoning; they do not choose the catalog design.

- **Identity:** this identifier continues to denote the same item, model, or category.
- **Representation:** a supported program can still read and write the stored representation.
- **Interpretation:** the answers it obtains still mean what its contract promises.

A fourth concern, operational availability, cuts across them. A compatible schema change can still block traffic while acquiring locks. Conversely, an instantaneous change can silently corrupt interpretation.

**Source observation.** GitLab's migration guide explicitly accounts for running applications that still use old column names, ORM schema caches, and dependent views. It documents phased column removal and distinguishes small-table rename procedures from larger backfills. This is deployment-specific engineering guidance, not a universal number of releases or a proof that migrations cause no downtime. [S1]

**Source observation.** OBO Principle 19 requires a new IRI when a definition substantially changes its referents; minor clarifications preserving referents are allowed. It separates exact replacement guidance (`replaced_by`) from inexact alternatives (`consider`). Domain judgment remains necessary. [S3]

These address different contracts. SQL compatibility cannot decide semantic identity; a stable ontology identifier cannot keep an old SQL statement executable.

## 2. One catalog, four different changes

Continue the community workshop from Module 1. Keep physical item IDs separate from model IDs and category IDs. `I-17` is one actual drill, `M-8` is its product model, and `C-10` is a controlled category. A category is a class of equipment, not an individual tool.

Initial conceptual records:

```text
ProductModel(model_id, display_name)
Item(item_id, model_id, inventory_label)
Category(category_id, preferred_label, definition, status)
Classification(assertion_id, item_id, category_id, evidence_ref)
```

These are explanatory relation signatures, not a complete SQL schema. In particular, history, review, and release fields are introduced below. Classifications may overlap; this module imposes no one-category-per-item rule.

### A. Rename: preserve identity, change wording

`M-8` changes its display name from “Workshop Drill 18” to “Workshop Drill 18V.” The manufacturer model and intended equipment remain the same. Retain `M-8`; record the old label and its applicable revision. An old invoice does not acquire a new original description merely because the display name changes.

The application separately wants to rename the column `display_name` to `preferred_label`. This is a representation change with the same value contract. It needs compatibility handling even though no model identity changes.

**Counterexample:** using the display name as a primary key makes a wording correction look like deletion plus creation. But preserving a numeric key while changing the category definition from “battery-powered drill” to “any portable drill” fails just as badly: stable syntax alone is not stable meaning.

### B. Changed meaning: mint a new category

`C-10`, “cordless drill,” means a drill powered by an onboard battery. A proposed replacement definition, “portable drill,” includes mains-powered handheld drills. Create `C-11` for the broader concept. Do not silently rewrite `C-10`'s definition.

A justified narrower-to-broader relationship can support a deliberately broader search. It does not establish equivalence. A new application can show `C-11` results; an old application asking for `C-10` must still exclude mains-powered drills. Keep `C-10` active if it remains useful. Creating a broader category does **not** by itself require obsoleting the narrower one.

**Counterexample:** all ten currently cataloged portable drills happen to be battery-powered. Equal answers on those ten records do not establish equivalent definitions. The eleventh, mains-powered drill exposes the error. Tests need a semantic counterexample, not only a comparison of current row counts.

### C. Merge: exact equivalence needs evidence

Suppose `C-20`, “battery-powered drill,” and `C-21`, “cordless drill,” have independently maintained but genuinely equivalent definitions under this catalog's scope. Review decides to retain `C-21` and retire `C-20` with an exact replacement mapping. Preserve the old identifier, original assertions, decision evidence, reviewer, and release of the mapping.

A current canonical view may map both IDs to `C-21`. If `I-17` was classified under both, a count of distinct tools remains one. Preserve both source assertions where they have separate evidence; do not confuse assertion count with inventory count.

**Counterexample:** near-identical names hide differing definitions: one category includes battery hammer drills and the other explicitly excludes them. A label matcher cannot authorize an exact merge. An inexact mapping or a revised hierarchy may be appropriate instead.

### D. Split: missing distinctions cannot be backfilled from nothing

An imported legacy category `C-30`, “drill bit,” mixed wood-only bits and masonry-only bits in this simplified dataset. Editors retire it for new precise classification and create `C-31` and `C-32`. This is an editorial choice for this example; a meaningful broader category could instead remain active as a parent.

- `I-41`: retained packaging says wood-only; create a supported `C-31` classification.
- `I-42`: retained packaging says masonry-only; create a supported `C-32` classification.
- `I-43`: source says only “drill bit”; preserve the legacy assertion and mark the current reclassification unresolved.

Do not classify `I-43` under both children, choose the first child, or infer its type from the old category ID. The hypothetical children are disjoint by the explicitly stated scope, but a real catalog might also need multi-material bits. That would require changing the proposed partition before migration, rather than forcing observations into it.

**Counterexample:** replacing every `C-30` ID with `C-31` satisfies a foreign key and finishes a backfill while inventing wood-only evidence. A database migration can be structurally valid and substantively false.

## 3. Expand, migrate, contract: a rename with mixed-version readers

Consider only the meaning-preserving column rename from §2A. Application A reads/writes `display_name`; B reads/writes `preferred_label`. The expanded schema E contains both. Neither application is permitted to use this deployment to change the value's meaning.

### Expand

1. Inventory readers **and writers**: web processes, workers, scheduled exports, importers, reports, and operational scripts. Include explicit column lists, positional decoding, cached schemas, views, and change-stream consumers.
2. Add the new representation without removing the old one. Decide how old inserts behave before making the new column mandatory.
3. Install a single atomic synchronization policy covering **all** approved write paths before relying on the new column. For this exercise, a database-side bridge mirrors whichever field was intentionally changed, rejects conflicting simultaneous values, and handles null transitions explicitly. The source field must be observable through version-specific writable views/entry points or explicit write-source metadata; comparing row values alone is insufficient. The exact routing, trigger code, and privileges remain an implementation task.
4. Test old A against E before enabling B. Addition is not automatically safe: `SELECT *` with positional deserialization or an insert lacking an explicit column list can break.

**Write-intent counterexample.** Before backfill, the row contains `(display_name='X', preferred_label=NULL)`. B intentionally clears the label by supplying `preferred_label=NULL`. Comparing old and new column values detects no change to that field; a naive trigger leaves `display_name='X'`, and a read fallback resurrects X. Our proposed bridge therefore requires observable write intent and mirrors the clear to both fields. Alternatively, keep B read-only until backfill has reconciled both columns, then enable its tested write path. Merely promising “null handling” does not solve the ambiguity.

An application that writes one column and later writes the other in a separate transaction is not this bridge. A crash between those transactions exposes inconsistent copies.

### Migrate

Backfill existing rows in bounded batches, with an explicit conflict policy. A worker must not overwrite a newer label using an earlier snapshot:

```text
t1  Backfill reads M-8's old value “Workshop Drill 18”.
t2  An editor commits “Workshop Drill 18V”; the bridge updates both columns.
t3  A naive worker writes its t1 value to preferred_label.
```

If the bridge accepts that t3 write, the stale worker can even revert both columns. “We use a trigger” is not a complete concurrency argument. Choose and test a current-row update or optimistic version check that causes a stale batch to retry or skip. Specify precedence before deployment.

For example, the worker reads label X with row version 7; the editor writes Y and atomically advances the row to version 8; the worker's conditional write requiring version 7 affects zero rows and retries from current state. All relevant writers must participate in version advancement. This is a proposed successful schedule, not an executed test.

During migration, B may temporarily read the new field with an old-field fallback only if the fallback has exactly the same meaning. Equality checks must be null-safe. Separately test inserts, updates, intentional clearing, conflicting two-field updates, retries, and interrupted batches. A zero mismatch count at one instant does not show that every writer is covered.

After backfill and reconciliation, enable B reads gradually. Check semantic query equivalence for the rename, not merely whether both queries return successfully. Record which versions are still allowed to write.

### Contract

Retire A and other old consumers, including queued work and rollback binaries. Establish that no permitted writer depends on the old representation. Only then remove the bridge and old column, in an order that leaves the remaining write path valid. Update dependent objects and any ORM ignore rules as required by the actual stack.

A useful compatibility matrix is explicit about stages:

| Combination | Required behavior in this plan |
| --- | --- |
| A + old schema | Baseline |
| A + E | Supported throughout the overlap |
| B + old schema | Disallowed by deployment ordering |
| B + E before backfill finishes | Reads: tested same-meaning fallback. Writes: observable-intent bridge; otherwise disabled |
| A and B + E | Supported with atomic synchronization and conflict policy |
| A + contracted schema | Unsupported; prevent deployment |
| B + contracted schema | Supported after contract tests |

A release number is not evidence that an old worker has disappeared. Use deployment inventory, write-path auditing, and retirement of old job payloads to justify the contraction gate.

**Source observation.** PostgreSQL 18 permits adding eligible constraints as `NOT VALID` and validating later. This skips the initial old-row scan but enforces the constraint for subsequent inserts/updates; validation scans existing rows. Locking still matters. This mechanism supports staged validation, not a claim of zero blocking or domain correctness. [S2]

**Counterexample:** enforcing a new required field while A still omits it can reject A's writes before backfill completes. “NOT VALID” does not mean “ignore new violations until we finish.”

## 4. Apply the protocol to meaning changes without pretending they are renames

For §2B, expansion adds `C-11` and its definition, not a second spelling of `C-10`. During overlap, versioned query contracts distinguish “battery powered” from “portable.” New items can be classified using available evidence under both appropriate categories, but a blanket bidirectional copy is wrong. An old client must never receive “portable” as if it meant “battery powered.”

For the merge, canonicalization is a release-specific derived view. It can update current search and reporting while preserving original IDs and evidence. A review decision grants semantic equivalence under the recorded scope; a foreign key merely guarantees that a referenced category row exists.

For the split, each supported reclassification is a new assertion linked to its evidence and migration decision. Unresolved records are an explicit output of the migration. Completion can mean “all records either reclassified with evidence or accounted for as unresolved,” provided the application's agreed query contract exposes that incompleteness. It cannot mean “all records made non-null by guessing.”

A mixed-version deployment can therefore be technically compatible while only supporting restricted semantic interoperability. Declare rejected writes, lossy projections, and unknown values rather than advertising universal backward compatibility.

## 5. What OBO and GO contribute, and what they do not

**Source observation.** GO's overview separates an accession from its name and definition. It describes retained obsolete terms/IDs with relationships removed and explanatory comments. It also documents alternate IDs arising from historical merges of equivalent terms. [S4]

**Important source reconciliation.** The GO Consortium's *The Gene Ontology knowledgebase in 2026* explicitly reports that GO stopped merging terms in 2022 and instead uses obsoletion with replacement. Its update sections also discuss reviewing affected annotations and citing release versions for reproducibility. The overview's alternate-ID description should not be read as the current procedure for initiating new merges. This module uses “merge” as a general catalog operation; it does not prescribe historical GO merge mechanics as current GO practice. [S5]

OBO's obsoletion policy removes logical axioms involving obsolete terms and provides replacement guidance. This makes a tombstone different from an active class. [S3] In our catalog, preserve the retired category record for interpretation and prevent new active classifications from using it through an application or database rule designed for that purpose. A plain foreign key to the tombstone does not enforce that rule.

These sources supply maintenance policies and documented practices, not measured error rates for our catalog. They do not prove an exact mapping is correct, make a split automatically decidable, or require a relational catalog to adopt an ontology stack.

## 6. Historical queries require more than a surviving ID

For this exercise, retain versioned definitions, original assertions, supersession decisions, and a publication manifest. One possible minimal extension is:

```text
CategoryRevision(category_id, revision_id, label, definition, status)
ClassificationRevision(assertion_id, revision_id, item_id, category_id,
                       effective_from, effective_to, recorded_at, evidence_ref)
MigrationDecision(decision_id, kind, input_ids, output_ids,
                  mapping_scope, evidence_ref, approved_at)
ReleaseManifest(release_id, category_revision_set,
                assertion_revision_set, mapping_revision_set, query_contract)
```

This is a conceptual design, not a validated relational decomposition. `recorded_at` alone is not a complete bitemporal implementation: reconstruction also needs immutable revision/supersession history and a rule selecting what was known at a cutoff. Overlapping effective intervals, corrections, and deletions need explicit handling.

Three queries illustrate why the manifest matters:

1. **“What did the June catalog say about I-43?”** Use the June release's assertion, definition, and label: it was recorded as `C-30`, “drill bit.” Do not silently apply today's split.
2. **“How should that June item be classified using evidence available now?”** Use later evidence applicable to the historical item. If none distinguishes it, return unresolved rather than retrofit a child category.
3. **“How many tools are in the current canonical category C-21?”** Apply the selected exact-replacement mapping and count distinct item IDs; retain provenance for the separate contributing assertions.

If a stable category's parent relationships change, historical descendant queries can change even without changing its label or ID. Reproducibility needs the relationship version and query policy too. A surviving tombstone is enough to recognize an old reference, not to reconstruct every historical query result.

## 7. Rollback has an information boundary

- **Before contraction:** reverting B to A may be feasible if the old representation remains synchronized and B has admitted no values that A cannot interpret. Verify this; do not infer it from a reversible migration file.
- **After dropping the old column:** re-adding a column restores structure, not discarded data. Recovering it needs retained values, a correct reverse transformation, or a separately tested restore/replay path.
- **After a merge:** if original category IDs and decisions were retained, the canonical view can be revised. If all references were overwritten and evidence collapsed, reversing the merge may be underdetermined.
- **After a split:** rolling back software cannot undo outside actions taken on new classifications. Nor can a reverse mapping recover which old assertion supplied an answer unless lineage was kept.

Restoring a database backup also risks discarding legitimate later writes; replay and reconciliation are separate tasks. A corrected forward release may be safer than a destructive reversal. This is an operational choice to test, not a universal preference. Retention limits may prevent full replay, which must be disclosed rather than hidden behind “immutable history.”

## 8. Acceptance packet and stopping point

A future implementation exercise should pass these explicit tests:

| Test | Expected result |
| --- | --- |
| Rename a model label | Same model ID; old release still shows its recorded label |
| Add a mains-powered portable drill | Appears in C-11 queries; excluded from C-10 queries |
| Merge two exact category aliases | One distinct tool count; both evidence assertions remain traceable |
| Attempt a name-only false merge | Review blocks exact equivalence |
| Split I-41, I-42, I-43 | Wood-only, masonry-only, and unresolved respectively |
| Race editor against stale backfill | Newer committed value is not overwritten |
| Crash or retry a batch | No silent divergence or duplicated semantic assertion |
| Run A during expanded schema | Reads/writes retain the old contract |
| Attempt A after contraction | Deployment rejected, rather than discovering failure in traffic |
| Query June with today's definitions accidentally | Test detects changed interpretation |

**Completed here:** source comparison, worked examples, a deployment protocol, failure traces, and a proposed acceptance packet. **Not completed:** trigger implementation, SQL execution, lock/load measurement, a historical ontology replay, or empirical reliability evaluation.

Three bounded questions remain:

1. Which actual client behaviors and write paths must a small executable rename test cover? Answer with one pinned database/driver combination, not a product survey.
2. Which evidence justifies the hypothetical category equivalence and split? Build a tiny labeled fixture before claiming an automatic mapping.
3. What history must be retained for these three stated historical queries? Test that minimum rather than storing everything indefinitely.

The module stops here: a coherent migration contract and counterexample set. No technology selection or wider-domain mapping follows from it.

## Sources

See [the evidence ledger](03-schema-evolution-evidence.md) for inspected sections, limitations, and source reconciliation.

- **S1:** [GitLab, Avoiding downtime in migrations](https://docs.gitlab.com/development/database/avoiding_downtime_in_migrations/).
- **S2:** [PostgreSQL 18, ALTER TABLE](https://www.postgresql.org/docs/18/sql-altertable.html).
- **S3:** [OBO Foundry, Principle 19: Stability of Term Meaning](https://obofoundry.org/principles/fp-019-term-stability.html).
- **S4:** [Gene Ontology overview](https://geneontology.org/docs/ontology-documentation/).
- **S5:** [The Gene Ontology knowledgebase in 2026, DOI](https://doi.org/10.1093/nar/gkaf1292); [public full-text PDF inspected](https://api.repository.cam.ac.uk/server/api/core/bitstreams/58da22a4-1d97-49c3-969b-b592daa53392/content).
