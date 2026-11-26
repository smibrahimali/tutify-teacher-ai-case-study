# Architecture Decisions

High-level ADRs for MaryamBot. No proprietary schemas, source, or credentials — design intent only.

---

## ADR-01 — Heuristics-First + Constrained Groq JSON

**Decision:** Resolve quick actions, menus, and multi-turn clarifications with a deterministic heuristic parser before calling Groq. Free-form prompts return low-temperature, Zod-validated structured intent JSON only.

**Why:** Known teacher intents (log session, hours, earnings, roster) should be instant and zero-token. The LLM must never emit SQL, raw database IDs, or direct write payloads — only intent the server can authorize and execute.

**Consequence:** Predictable latency for common paths; graceful fallback when Groq is offline or unconfigured; validation failures never reach the write path.

---

## ADR-02 — Propose-Then-Confirm with Atomic Claims

**Decision:** Chat never inserts sessions directly. Validated log intents create a pending review card; confirmation atomically claims that pending action, re-verifies ownership and date bounds, then writes. On failure, the claim is released.

**Why:** Conversational UIs invite accidental submits and double-taps. An explicit review step plus claim/release prevents duplicate sessions and false confirmations under retry.

**Consequence:** Teachers always see student, PKT date, and time/duration before commit; retries are safe; mid-flow topic switches can soft-gate with Continue / Start over.

**Operational detail:** Civil dates are locked to `Asia/Karachi` — today or the past 10 calendar days; future dates are rejected. Ambiguous surnames open a disambiguation picker without dropping captured time slots.

---

## ADR-03 — Server-Side RBAC & Financial Field Exclusion

**Decision:** Resolve `teacher_id` exclusively from the authenticated Supabase SSR session. Scope every enrollment, session, and pay-term read to that teacher. Exclude internal client billing and company margin fields from assistant responses.

**Why:** Client payloads and LLM output are untrusted. MaryamBot is teacher-only — no student, guardian, or admin surface — and must not leak financial internals while still answering hours and estimated earnings questions.

**Consequence:** Cross-teacher privacy by construction; reports stay within allowed fields; RLS plus service-role write boundaries reinforce the same isolation at persistence.

**Operational detail:** Confirmed session writes invalidate TanStack Query caches so the portal Sessions view updates immediately after atomic confirm.

---

## ADR-04 — Fail-Closed Feature Flags

**Decision:** Gate rollout with `ASSISTANT_MODE`, `ASSISTANT_ALLOWED_TEACHER_IDS`, and `ASSISTANT_WRITES_ENABLED`. Missing, empty, or disabled flags mean the assistant is unavailable or read-only — never accidentally write-capable in production.

**Why:** Staged teacher allowlists and write kill-switches reduce blast radius during internship delivery and post-ship hardening.

**Consequence:** Safe progressive rollout; builds and deploys remain safe without API keys or with flags off; ops can disable writes without a code change.

**Operational detail:** UX ships as a floating launcher and branded chat sheet (mobile bottom sheet / desktop popover) with student picker, bounded PKT calendar, and structured report cards. Regression coverage: Vitest `MB-01`–`MB-10`.
