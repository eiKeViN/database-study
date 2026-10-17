# Sources for the Zanzibar consistency case

Consulted 17 October 2026. Companion case: [Content and authorization consistency](zanzibar-content-authorization-consistency.md).

## Primary paper

Ruoming Pang et al. **Zanzibar: Google's Consistent, Global Authorization System.** USENIX ATC 2019, pp. 33–46.

- Official full text: https://www.usenix.org/system/files/atc19-pang.pdf
- Official publication page: https://www.usenix.org/conference/atc19/presentation/pang
- Focused reading: §2.2, §§2.4.1–2.4.4, §§3.1–3.2.5, §§4.1–4.3. Supporting context: §§4.4–4.5 and §5.
- Locate the two permission-order/content-order examples and client protocol on printed pp. 35–36; the Check API on p. 37; configuration consistency on p. 39; cache timestamp handling on p. 40; request categories and measurement definitions on pp. 41–42.
- Table 2's entries are means and standard deviations of percentile latency over its sampled period. Its p95 figures are not a maximum, an all-request p95, or a client-observed end-to-end latency guarantee. The case uses the Safe/Recent distinction deliberately.
- The paper is an author-reported historical production study. No independent reproduction or present-day deployment audit was performed here.

## Existing repository context

- Synthesis: https://github.com/eiKeViN/database-study/blob/main/notes/synthesis.md
- Lifecycle: https://github.com/eiKeViN/database-study/blob/main/notes/lifecycle.md

The case builds on the synthesis and lifecycle notes as published in [the initial study commit](https://github.com/eiKeViN/database-study/tree/a054ce09e5a20800addb46e0c3b5790077178758).

## Evidence separation

Paper mechanism and historical metric claims are cited directly. The workshop example, failure cases, acceptance tests, and KB recommendations are original explanatory/design work, not paper experiments or a completed implementation. No new empirical results are claimed.
