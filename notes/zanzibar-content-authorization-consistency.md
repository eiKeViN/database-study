# Zanzibar: which permissions may authorize which content version?

Research date: 17 October 2026. A bounded close reading of Pang et al., USENIX ATC 2019, with an original design exercise. No implementation, benchmark, product choice, or mathematics mapping is claimed.

This continues the [synthesis's bounded reading plan](https://github.com/eiKeViN/database-study/blob/main/notes/synthesis.md#7-unknowns-and-a-bounded-continuation) and sharpens the [lifecycle permission discussion](https://github.com/eiKeViN/database-study/blob/main/notes/lifecycle.md#10-permissions-and-deletion-are-part-of-semantics). The main distinction is between protecting a newly published version from obsolete permissions and enforcing a separately specified revocation deadline for every version.

## 1. Primary-source anchor

Zanzibar addresses the “new enemy” problem: earlier permission removal must protect later content. Spanner supplies externally consistent ordering; checks use one ACL snapshot. A content-change check uses the latest snapshot and returns an opaque zookie. Clients atomically store it with content and supply it on later checks; it bounds their minimum snapshot. Content storage need not share the check's transaction. Zookies are consistency tokens, not document revision IDs. Snapshot timestamps enter authorization-cache keys. [§§2.2, 2.4.4, 3.2.5, pp. 35–37, 40](https://www.usenix.org/system/files/atc19-pang.pdf).

The historical evaluation uses seven days in December 2018 and server-side live-traffic latency. Table 2 reports mean p95 Check latency of 9.46 ms for Safe versus 60.0 ms for Recent requests. Those categories distinguish zookies older versus younger than ten seconds. Availability above 99.999% concerns qualified replay probes, 90-day aggregation averaged across clusters, and three historical years; thresholds are five seconds for Safe and fifteen for Recent, with request/deadline/quota qualifications. These are neither current SLAs nor end-to-end KB measurements. [§§4.1–4.3, pp. 41–42](https://www.usenix.org/system/files/atc19-pang.pdf).

## 2. Worked exercise: a collaborative workshop guide

**Invented scenario and reasoning, not an observed deployment.** Alice manages a workshop guide; Charlie edits it; Bob formerly had access. Assume Bob has no other permission path. The number labels below describe ordering, not real clock readings or a usable token format.

1. **Initial state.** Version 7 contains ordinary setup advice. An authorization replica still has the state in which Bob can read it.
2. **Removal completes.** Alice removes Bob. Call this permission event R. She then tells Charlie that access has been removed and asks him to add confidential supplier terms. The human message matters: the second operation is deliberately after the first, even though the authorization service never reads that message.
3. **A bad read.** Charlie publishes version 8. A service combines its fresh text with the old cached answer “Bob can read this guide.” Both components can be internally well-formed, yet their combination discloses the new terms. Merely attaching the document ID to a permission result does not identify the version that result can safely cover.
4. **A correctly bound publication.** Before saving version 8, the application obtains a successful content-change authorization check for Charlie. Call its returned token Z8. It commits the pair `(version 8 content, Z8)` atomically in its own storage. A crash must not leave version 8 associated with version 7's token.
5. **Bob requests version 8.** The serving path fetches that exact content/token pair and supplies Z8 when asking whether Bob can view it. The chosen permission snapshot must be no earlier than Z8. Because removal R completed before this content-change check, that snapshot includes the removal. With no other grant, Bob is denied.
6. **A lagging replica is not an excuse.** If the required state is unavailable locally, the service must obtain an eligible evaluation or return a failure; substituting an older successful decision would break the contract. A token does not make replication instantaneous.

The crucial link is **the authorization freshness requirement travels with the content version**. A content hash identifies bytes; it does not say how fresh permissions must be. A token identifies a consistency requirement; possessing it is not permission to read.

### A related structural exercise

Instead of editing text, Alice removes Bob from a folder and then Charlie moves a previously inaccessible guide into it. A broken evaluator might combine the new parent link with the old folder membership. A coherent snapshot must not manufacture that combination. This is a different failure surface from serving fresh bytes under an old permission decision: permission relationships themselves can change across objects. The no-other-grant assumption remains essential; inherited access removal cannot cancel an unrelated direct grant by wishful interpretation. See the paper's two distinct examples in §2.2 for the underlying distinction.

## 3. What this exercise does not establish

These are boundaries of the claim, not additional promises inferred from the benchmark:

- **Immediate revocation of all old content.** Suppose Bob requests unchanged version 7 after removal. Its old token alone does not require evaluation after removal R. A product promising immediate or bounded-time revocation needs an explicitly stronger serving-freshness contract. “No old ACL on new content” and “every request observes every completed removal” are different requirements.
- **An atomic permission-and-content transaction.** Charlie could pass the content-change check and lose edit permission before the content save. Decide whether authorization at check time is sufficient or whether commit-time entitlement is required. The protocol should not be advertised as resolving every such race automatically.
- **Erasure or recall.** Denying a request does not erase an earlier export, cached bytes on someone else's device, or a message already delivered.
- **Secure generated summaries.** A summary assembled from three restricted inputs needs its own derivation and audience rules. Neither a document token nor a citation list tells us whether that summary is safe to disclose. Filtering only the final document link can still expose its restricted text through a snippet or answer.
- **Correctness of the guide.** Access control does not establish that supplier terms are accurate, well-sourced, or interpreted correctly.

Namespace-policy changes are another boundary worth keeping explicit: §3.2.2 describes consistent configuration snapshots across a request's fan-out. Do not silently assume that every policy-definition change has exactly the same freshness semantics as a relation-tuple removal.

## 4. What a generic knowledge-base designer can use

**Our design recommendations, not a universal Zanzibar architecture:**

1. State which event must be visible before which disclosure. Avoid a single vague “up to date” label covering text, permissions, indexes, and answers.
2. Identify the exact protected version throughout serving. Checking version 7 and then fetching version 8 creates a race even if each individual call succeeds.
3. Treat permission-aware caches as part of correctness. For any reuse, ask which principal, object/version, policy context, and freshness requirement the result covers. A short TTL alone is not proof of causal safety.
4. For derived answers, choose and test an audience rule. Conservative blocking and regeneration may be preferable to pretending a lossy summary has exact sentence-level dependencies.
5. Start with the simplest mechanism that meets the chosen contract. A small service may keep content and access metadata together and coordinate transactions and reads locally. A public handbook may not need confidential-read authorization at all. Global authorization infrastructure is a response to particular sharing, distribution, and load requirements.

## 5. Small acceptance-test plan, not executed results

- **Removal then edit:** deliberately delay permission replication; Bob must not receive version 8, including through previews and cached answers.
- **Content/token crash:** interrupt publication between writes; storage must never expose a mixed pair.
- **Cross-version race:** change the document between lookup and delivery; the permission decision must cover the version actually returned.
- **Inherited access:** delay folder-membership propagation while moving the guide; test both stale-snapshot and fresh-snapshot outcomes, plus a separate direct grant.
- **Revocation without edit:** leave version 7 unchanged; measure actual denial delay against the product's declared requirement rather than assuming zero.
- **Revocation during generation:** remove access after retrieval but before answer delivery; test the chosen delivery policy and whether restricted snippets remain.

Record served content revision, check freshness requirement, authorization outcome, visible bytes, and failure handling. Do not turn a permission-service latency measurement into a claim about whole-answer safety.

## 6. Unresolved questions and stopping point

The worked case closes the intended reading loop. Four design questions remain open:

1. What revocation deadline should apply to unchanged versions and in-flight responses?
2. Is edit authority required at check time, commit time, or both?
3. Which policy governs mixed-source summaries, historical views, and exports?
4. How will the chosen implementation verify freshness and fail safely when dependencies lag?

Answering these requires a concrete application's contract and a scoped experiment, not a larger reading list. No current vendor implementation or 2026 service guarantee has been evaluated.

## Source and reading record

See the [source ledger](zanzibar-sources.md) for the inspected paper sections and evidence boundaries.
