# Database and Knowledge-System Design Study

A growing set of source-grounded learning notes on database design and mature knowledge systems: their principles, concrete engineering choices, demonstrated effects, and unresolved problems.

The first research cut is **16 October 2026**. This is an AI-assisted study collection, not an authoritative textbook, exhaustive literature review, or implemented system. The notes distinguish source findings, design interpretations, hypothetical examples, and questions still requiring evidence. Selected notes include brief Chinese summaries; the main text is English.

## Start here

- For the overall picture: [First research map](notes/synthesis.md).
- For the learning materials: [Source index](SOURCES.md).
- For a concrete first lesson: [Modeling and integrity](notes/01-modeling-and-integrity.md).

## Reading route

1. **Modeling before technology selection.** Use *Database System Concepts*, seventh edition, as the main textbook, with Codd and Chen for the original motivations. Work through the equipment-lending example in [Module 1](notes/01-modeling-and-integrity.md): entities, occurrences, keys, functional dependencies, lossless decomposition, dependency preservation, and SQL integrity boundaries.
2. **Implementation costs and systems tradeoffs.** Use the completed CMU 15-445/645 Fall 2025 course as an implementation companion. Read *Designing Data-Intensive Applications*, second edition, for authoritative versus derived data, evolution, and end-to-end correctness. See the selected route in [Foundations](notes/foundations.md) and the learning guide in [Lifecycle engineering](notes/lifecycle.md).
3. **Meaning, evidence, and interoperability.** Study knowledge-graph foundations and a bounded RDF/RDFS, OWL, SHACL, SKOS, and PROV-O packet. Distinguish inference from validation, identity from labels, and provenance from truth. Start with [Foundations](notes/foundations.md).
4. **Real systems and their constraints.** Compare Wikidata/Wikibase, Gene Ontology/OBO, and Uber Databook. Add GitLab Handbook as a document-oriented counterpoint. The [case studies](notes/production-systems.md) explain what each source establishes and where empirical evidence remains weak.
5. **Change, evaluation, and the frontier.** Study [Lifecycle engineering](notes/lifecycle.md), [Evaluation and research gaps](notes/evaluation-gaps.md), and [Frontier readings](notes/frontier-readings.md). DBSP concerns incremental maintenance; LOTUS concerns semantic operators and optimization; SemBench concerns workload-level evaluation. Their guarantees are different.
6. **Reconcile the lessons.** Return to the [synthesis](notes/synthesis.md), its ordinary-domain examples, and its bounded next-reading questions. Do not infer a universal architecture from a successful case.

This round deliberately studies general database and knowledge-system design before transferring ideas to a specialized domain. It selects no product stack.

## Contents

| Note | Main question |
| --- | --- |
| [Foundations](notes/foundations.md) | Which conceptual distinctions and learning resources should anchor the study? |
| [Modeling and integrity](notes/01-modeling-and-integrity.md) | What do keys, normalization, and constraints actually guarantee? |
| [Production systems](notes/production-systems.md) | How do mature systems organize meaning, authority, evidence, and serving? |
| [Lifecycle engineering](notes/lifecycle.md) | What must happen when sources, claims, schemas, or permissions change? |
| [Evaluation and research gaps](notes/evaluation-gaps.md) | What demonstrates quality, and what do current evaluations miss? |
| [Frontier readings](notes/frontier-readings.md) | Which recent systems papers are worth reading after the fundamentals? |
| [Synthesis](notes/synthesis.md) | Which lessons survive comparison, and which remain conditional? |

## Evidence and reading status

- The collection includes source screening, selected close readings, and original synthesis. Listing a book does **not** mean it has been read in full. Individual notes document inspected sections and access limitations.
- Module 1's dependency calculations and counterexamples were reviewed in prose. Its SQL was **not executed**, and no performance benchmark or user study was run.
- Historical production measurements remain historical; they are not current service guarantees. Preprints and draft standards are labeled separately from published papers and stable recommendations.
- Proposed experiments and next exercises are **unperformed** unless a later note records their execution.
- Publicly available source material is linked and attributed. The repository contains study notes, not redistributed textbooks or a claim to their rights.

## Keeping the study useful

For each new reading, record the problem, assumptions, design rationale, evidence, failure modes, and what remains unclear. Keep source claims separate from interpretation. Update the relevant note and source index when correcting a claim; use Git history for the record rather than duplicating dated copies of every file.

Two possible next close readings are EMERGE v2 and Zanzibar, scoped in the synthesis. They are suggestions, not completed modules.
