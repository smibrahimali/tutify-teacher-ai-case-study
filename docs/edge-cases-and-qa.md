# Edge Cases & QA (`MB-01`–`MB-10`)

Regression scenarios verified against MaryamBot with synthetic fixture data. High-level expected behavior only — no proprietary implementation details.

| ID | Scenario | Expected behavior |
| --- | --- | --- |
| **MB-01** | Ambiguous student surname matches multiple roster entries | Show an explicit disambiguation picker; do not guess |
| **MB-02** | Time slots captured before the student is resolved | Preserve entered slots through picker → review card |
| **MB-03** | Session date within PKT 10-day lookback (`Asia/Karachi`) | Accept civil date in range (today or past 10 calendar days) |
| **MB-04** | Future-dated session request | Reject with a clear message; no pending write created |
| **MB-05** | Bare AM/PM without a concrete hour | Prompt for clarification; never silently default into a write |
| **MB-06** | Mid-flow topic switch (e.g. hours while logging) | Soft gate: Continue current flow / Start over |
| **MB-07** | Confirm retry after transient failure | Release atomic claim; retry does not create a duplicate session |
| **MB-08** | Double-confirm / claim race | Single write wins; second confirm fails safely |
| **MB-09** | Synthetic two-teacher privacy seed | Teacher A never sees Teacher B roster, sessions, or earnings |
| **MB-10** | Production build with no Groq API key | App builds; assistant degrades gracefully (heuristics / offline messaging) |

Related: [Architecture Decisions](./architecture-decisions.md)
