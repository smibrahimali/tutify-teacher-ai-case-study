<div align="center">

## TUTIFY PORTAL — MARYAMBOT (ENGINEERING CASE STUDY)

**Teacher-Only AI Assistant** &nbsp;·&nbsp; **Propose-Then-Confirm Architecture** &nbsp;·&nbsp; **RBAC & Data Isolation**

<sub>System design, problem-solving breakdown, and UI walkthrough of the teacher-facing AI assistant shipped into <a href="https://portal.tutify.org/">portal.tutify.org</a> during my Software Engineering internship.</sub>

<br/><br/>

<a href="https://portal.tutify.org/">
  <img src="https://img.shields.io/badge/LIVE_PORTAL_↗-E4E4E7?style=for-the-badge" alt="Live Portal" />
</a>&nbsp;
<a href="https://github.com/smibrahimali">
  <img src="https://img.shields.io/badge/BACK_TO_PROFILE_↗-161B22?style=for-the-badge" alt="Back to Profile" />
</a>

</div>

<br/>

> **Confidentiality Notice:** Tutify Portal is a proprietary production application. In accordance with company guidelines, this repository contains no source code—only a high-level engineering breakdown of the problem, architecture, verification practices, and synthetic-data UI screenshots.

<br/>

### ✦ The Problem

Logging completed lessons on the Tutify teacher portal required navigating across multiple screens—opening a student, selecting an enrollment cycle, entering duration or Pakistan Standard Time (`Asia/Karachi`) windows, saving or submitting for guardian approval, and checking hours or earnings on separate views.

The goal was to build **MaryamBot**, an embedded, teacher-only conversational assistant inside the portal (`portal.tutify.org`), to:
- Streamline **session logging** into a guided flow (`student → PKT date → time or duration → review card → draft or submit`).
- Provide instant **self-service reporting** for teaching hours, cycle-wise estimated earnings per student, active/past student rosters, and session counts.
- Enforce strict **domain and security boundaries**—no student/guardian/admin access, no billing mutations, no future-dated logs, and zero exposure of internal financial margins.

<br/>

### ✦ System Architecture & How I Solved It

#### 1. Server-Enforced RBAC & Financial Isolation
- **Server-side identity resolution:** `teacher_id` is derived strictly from the authenticated Supabase SSR session—never trusted from client payloads or LLM output.
- **Scoped lookups & field exclusion:** Every enrollment, session, and pay-term query is bound to the signed-in teacher's active/past enrollments, explicitly excluding internal client billing and company margin fields.
- **Database RLS & staged rollout:** Assistant persistence tables enforce Postgres Row-Level Security (service-role write access only) behind fail-closed feature flags (`ASSISTANT_MODE`, `ASSISTANT_ALLOWED_TEACHER_IDS`, and `ASSISTANT_WRITES_ENABLED`).

#### 2. Two-Layer Intent Orchestration (Heuristics First, Structured LLM Second)
- **Deterministic fast path:** Quick-action chips, menu triggers, and multi-turn clarifications are handled first by a deterministic heuristic parser for instant, zero-token responses.
- **Constrained LLM fallback:** Free-form or ambiguous prompts route to Groq with a low-temperature, Zod-validated JSON schema—returning **structured intent only** (never raw SQL, database IDs, or direct writes), backed by rate-limit retries and graceful fallback messaging when offline or unconfigured.

#### 3. Propose-Then-Confirm Write Guardrails
- **Atomic two-step execution:** Chat requests never write sessions directly; validated log intents create a pending review card, and confirmation atomically claims the pending action, re-verifies ownership and date bounds, and releases the lock on failure so retries never produce duplicate records or false confirmations.
- **Time-window & ambiguity rules:** Civil dates are locked to `Asia/Karachi` (today or the past **10 calendar days**, rejecting future dates), ambiguous student surnames trigger an explicit disambiguation picker without dropping captured time slots, and mid-flow topic shifts trigger a soft `Continue / Start over` gate.

#### 4. Portal UX Integration & Verification
- **Responsive embedded shell:** Built as a floating launcher and branded chat sheet across teacher routes (mobile bottom sheet / desktop popover) with an interactive student picker, bounded PKT calendar, and structured report cards.
- **Cache synchronization & test coverage:** Confirmed session writes invalidate TanStack Query caches so the portal's main Sessions view updates immediately; verified with Vitest regression suites (`MB-01`–`MB-10`), synthetic two-teacher privacy seeds, and a no-API-key build check.

<br/>

### ✦ Interface Walkthrough *(Synthetic Seed Data)*

<table>
  <tr>
    <td width="50%" valign="top">
      <b>◈ &nbsp;1. Teacher Portal Entry Point</b>
      <br/><br/>
      <img src="./screenshots/01-launcher.png" alt="Teacher portal with MaryamBot launcher" width="100%" />
      <br/><br/>
      <sub>Teacher portal home view with the floating MaryamBot launcher in the bottom-right corner.</sub>
    </td>
    <td width="50%" valign="top">
      <b>❖ &nbsp;2. Chat Open &amp; Quick Actions</b>
      <br/><br/>
      <img src="./screenshots/02-chat-open.png" alt="Chat open with quick actions" width="100%" />
      <br/><br/>
      <sub>Session logging prioritized up front, with Teaching hours, Earnings, and My students in the quick menu.</sub>
    </td>
  </tr>
  <tr>
    <td width="50%" valign="top">
      <b>⊹ &nbsp;3. Scoped Student Picker</b>
      <br/><br/>
      <img src="./screenshots/03-student-picker.png" alt="Guided session logging student picker" width="100%" />
      <br/><br/>
      <sub>Scoped roster picker resolving ambiguous names while preserving already-entered time slots.</sub>
    </td>
    <td width="50%" valign="top">
      <b>◎ &nbsp;4. Propose-Then-Confirm Review Card</b>
      <br/><br/>
      <img src="./screenshots/04-review-card.png" alt="Review card before creating session" width="100%" />
      <br/><br/>
      <sub>Explicit verification card offering <i>Log &amp; submit</i> or <i>Save as draft</i> before atomic database insertion.</sub>
    </td>
  </tr>
  <tr>
    <td colspan="2" valign="top">
      <b>✦ &nbsp;5. Structured Teaching Hours Report</b>
      <br/><br/>
      <img src="./screenshots/05-hours-report.png" alt="Teaching hours structured answer card" width="100%" />
      <br/><br/>
      <sub>In-chat hours and session count breakdown rendered as structured summary cards.</sub>
    </td>
  </tr>
</table>

<br/>

### ✦ Stack & Engineering Practices

<div align="center">

<img src="https://img.shields.io/badge/Next.js_App_Router-161B22?style=flat-square&logo=nextdotjs&logoColor=E4E4E7" />
<img src="https://img.shields.io/badge/React-161B22?style=flat-square&logo=react&logoColor=E4E4E7" />
<img src="https://img.shields.io/badge/TypeScript-161B22?style=flat-square&logo=typescript&logoColor=E4E4E7" />
<img src="https://img.shields.io/badge/Tailwind_CSS-161B22?style=flat-square&logo=tailwindcss&logoColor=E4E4E7" />
<img src="https://img.shields.io/badge/TanStack_Query-161B22?style=flat-square" />
<img src="https://img.shields.io/badge/Supabase_&_RLS-161B22?style=flat-square&logo=supabase&logoColor=E4E4E7" />
<br/>
<img src="https://img.shields.io/badge/Groq_Structured_JSON-161B22?style=flat-square" />
<img src="https://img.shields.io/badge/Zod_Validation-161B22?style=flat-square&logo=zod&logoColor=E4E4E7" />
<img src="https://img.shields.io/badge/date--fns--tz_(PKT)-161B22?style=flat-square" />
<img src="https://img.shields.io/badge/Vitest-161B22?style=flat-square&logo=vitest&logoColor=E4E4E7" />
<img src="https://img.shields.io/badge/Feature_Flags-161B22?style=flat-square" />

</div>