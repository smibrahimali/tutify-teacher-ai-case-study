<div align="center">

<img src="./assets/banner.svg" alt="MaryamBot — Tutify Portal" width="100%" />

<br/>

**Teacher-only AI assistant** for session logging and self-service reports — propose-then-confirm writes, server-enforced RBAC, shipped into [portal.tutify.org](https://portal.tutify.org/).

<br/>

<a href="https://portal.tutify.org/">
  <img src="https://img.shields.io/badge/Live_Portal-E4E4E7?style=flat-square&labelColor=161B22&color=E4E4E7" alt="Live Portal" />
</a>
&nbsp;
<a href="https://github.com/smibrahimali">
  <img src="https://img.shields.io/badge/Back_to_Profile-161B22?style=flat-square&labelColor=0D1117&color=161B22" alt="Back to Profile" />
</a>

</div>

<br/>

> **Confidentiality:** Tutify Portal is a proprietary production application. This repository contains no source code—only a high-level engineering breakdown, synthetic-data UI screenshots, and architecture notes.

### The Problem & Workflow Shift

Logging lessons meant bouncing across student, enrollment, time-entry, and reporting screens. MaryamBot collapses that into a guided teacher-only chat inside the portal.

| Legacy portal flow | MaryamBot guided flow |
| --- | --- |
| Open student → pick enrollment cycle → enter PKT window or duration → save / submit → check hours and earnings on separate views | Chat or quick actions → resolve student → PKT date + time or duration → review card → draft or submit → in-chat reports |

### System Architecture

```mermaid
flowchart LR
  A[Heuristics] --> B[Groq JSON + Zod]
  B --> C[Server RBAC]
  C --> D[Propose / Review Card]
  D --> E[Atomic Confirm + Cache Invalidation]
```

**RBAC & Financial Isolation** — `teacher_id` is resolved from the authenticated Supabase SSR session only. Every enrollment, session, and pay-term lookup is scoped to that teacher; internal client billing and margin fields are excluded. Persistence sits behind Postgres RLS and fail-closed flags (`ASSISTANT_MODE`, `ASSISTANT_ALLOWED_TEACHER_IDS`, `ASSISTANT_WRITES_ENABLED`).

**Hybrid Orchestration** — Quick-action chips, menus, and clarifications hit a deterministic heuristic parser first (zero tokens). Free-form prompts fall through to Groq with low-temperature, Zod-validated JSON — structured intent only, never SQL, raw IDs, or direct writes.

**Propose-Then-Confirm Guardrails** — Chat never writes sessions directly. Validated intents create a pending review card; confirmation atomically claims the action, re-checks ownership and PKT date bounds (`Asia/Karachi`, today or past 10 calendar days, no future dates), and releases the lock on failure so retries cannot duplicate records.

**Portal UX & Feature Flags** — Floating launcher and branded chat sheet across teacher routes (mobile bottom sheet / desktop popover), with student picker, bounded PKT calendar, and structured report cards. Confirmed writes invalidate TanStack Query caches so the Sessions view updates immediately. Flags fail closed; Vitest covers `MB-01`–`MB-10`.

### Interface Walkthrough *(Synthetic Seed Data)*

<table>
  <tr>
    <td width="50%" valign="top">
      <b>1. Teacher Portal Entry Point</b>
      <br/><br/>
      <img src="./screenshots/01-launcher.png" alt="Teacher portal with MaryamBot launcher" width="100%" />
      <br/><br/>
      <sub>Teacher portal home with the floating MaryamBot launcher in the bottom-right corner.</sub>
    </td>
    <td width="50%" valign="top">
      <b>2. Chat Open &amp; Quick Actions</b>
      <br/><br/>
      <img src="./screenshots/02-chat-open.png" alt="Chat open with quick actions" width="100%" />
      <br/><br/>
      <sub>Session logging prioritized up front, with Teaching hours, Earnings, and My students in the quick menu.</sub>
    </td>
  </tr>
  <tr>
    <td width="50%" valign="top">
      <b>3. Scoped Student Picker</b>
      <br/><br/>
      <img src="./screenshots/03-student-picker.png" alt="Guided session logging student picker" width="100%" />
      <br/><br/>
      <sub>Scoped roster picker resolving ambiguous names while preserving already-entered time slots.</sub>
    </td>
    <td width="50%" valign="top">
      <b>4. Propose-Then-Confirm Review Card</b>
      <br/><br/>
      <img src="./screenshots/04-review-card.png" alt="Review card before creating session" width="100%" />
      <br/><br/>
      <sub>Verification card offering <i>Log &amp; submit</i> or <i>Save as draft</i> before atomic insertion.</sub>
    </td>
  </tr>
  <tr>
    <td colspan="2" valign="top">
      <b>5. Structured Teaching Hours Report</b>
      <br/><br/>
      <img src="./screenshots/05-hours-report.png" alt="Teaching hours structured answer card" width="100%" />
      <br/><br/>
      <sub>In-chat hours and session count breakdown rendered as structured summary cards.</sub>
    </td>
  </tr>
</table>

### Stack & Engineering Rigor

<div align="center">

<img src="https://img.shields.io/badge/Next.js_App_Router-161B22?style=flat-square&logo=nextdotjs&logoColor=E4E4E7" alt="Next.js App Router" />
<img src="https://img.shields.io/badge/React-161B22?style=flat-square&logo=react&logoColor=E4E4E7" alt="React" />
<img src="https://img.shields.io/badge/TypeScript-161B22?style=flat-square&logo=typescript&logoColor=E4E4E7" alt="TypeScript" />
<img src="https://img.shields.io/badge/Tailwind_CSS-161B22?style=flat-square&logo=tailwindcss&logoColor=E4E4E7" alt="Tailwind CSS" />
<img src="https://img.shields.io/badge/TanStack_Query-161B22?style=flat-square" alt="TanStack Query" />
<img src="https://img.shields.io/badge/Supabase_&_RLS-161B22?style=flat-square&logo=supabase&logoColor=E4E4E7" alt="Supabase and RLS" />
<br/>
<img src="https://img.shields.io/badge/Groq_Structured_JSON-161B22?style=flat-square" alt="Groq Structured JSON" />
<img src="https://img.shields.io/badge/Zod_Validation-161B22?style=flat-square&logo=zod&logoColor=E4E4E7" alt="Zod Validation" />
<img src="https://img.shields.io/badge/date--fns--tz_(PKT)-161B22?style=flat-square" alt="date-fns-tz PKT" />
<img src="https://img.shields.io/badge/Vitest-161B22?style=flat-square&logo=vitest&logoColor=E4E4E7" alt="Vitest" />
<img src="https://img.shields.io/badge/Feature_Flags-161B22?style=flat-square" alt="Feature Flags" />

</div>

<br/>

**Deep Dive**

- [Architecture Decisions](./docs/architecture-decisions.md) — why heuristics-first, propose-then-confirm, server RBAC, and fail-closed flags
- [Edge Cases & QA (`MB-01`–`MB-10`)](./docs/edge-cases-and-qa.md) — regression scenarios for ambiguity, PKT bounds, atomic retry, and build safety
