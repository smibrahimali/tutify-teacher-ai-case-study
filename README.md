<div align="center">

<img src="./assets/banner.svg" alt="MaryamBot — Tutify Portal" width="100%" />

<br/>

**Teacher-only AI assistant** — propose-then-confirm session logging with server-enforced RBAC, shipped into [portal.tutify.org](https://portal.tutify.org/).

<br/>

<a href="https://portal.tutify.org/">
  <img src="https://img.shields.io/badge/LIVE_PORTAL-E4E4E7?style=for-the-badge&labelColor=161B22&color=E4E4E7" alt="Live Portal" />
</a>
&nbsp;
<a href="https://github.com/smibrahimali">
  <img src="https://img.shields.io/badge/BACK_TO_PROFILE-161B22?style=for-the-badge&labelColor=0D1117&color=161B22" alt="Back to Profile" />
</a>

</div>

<br/>

> **Confidentiality:** Tutify Portal is a proprietary production application. This repository contains no source code—only a high-level engineering breakdown, synthetic-data UI screenshots, and architecture notes.

### The Problem & Workflow Shift

| Workflow | Legacy Portal | MaryamBot |
| --- | --- | --- |
| **Session Logging** | Multi-screen student → cycle → time → save | Guided chat: student → PKT → review |
| **Ambiguous Names** | Manual search across roster screens | In-chat picker; time slots preserved |
| **Write Safety** | Immediate save with little confirm UX | Propose → review card → atomic confirm |
| **Hours & Earnings** | Separate reporting views | In-chat structured report cards |

### System Architecture

```mermaid
flowchart LR
  I[Teacher Input] --> H[Heuristics]
  H -->|Guided / Chip| R[Server RBAC]
  H -->|Free-Form| G[Groq JSON + Zod]
  G --> R
  R --> P[Propose / Review Card]
  P --> C[Atomic Confirm + Cache Invalidation]
```

<table>
  <tr>
    <td width="50%" valign="top">
      <b>RBAC &amp; Financial Isolation</b><br/><br/>
      Session-derived teacher identity only.<br/>
      Scoped reads; billing margins never exposed.
    </td>
    <td width="50%" valign="top">
      <b>Hybrid Orchestration</b><br/><br/>
      Heuristics first — zero tokens for chips.<br/>
      Free-form → Groq JSON, Zod-validated intent.
    </td>
  </tr>
  <tr>
    <td width="50%" valign="top">
      <b>Propose-Then-Confirm</b><br/><br/>
      Chat never writes sessions directly.<br/>
      Atomic claim on confirm; safe retries.
    </td>
    <td width="50%" valign="top">
      <b>Portal UX &amp; Feature Flags</b><br/><br/>
      Floating launcher + chat across teacher routes.<br/>
      Fail-closed flags; Vitest <code>MB-01</code>–<code>MB-10</code>.
    </td>
  </tr>
</table>

### Interface Walkthrough *(Synthetic Seed Data)*

<table>
  <tr>
    <td width="50%" valign="top">
      <b>1. Teacher Portal Entry Point</b>
      <br/><br/>
      <img src="./screenshots/01-launcher.png" alt="Teacher portal with MaryamBot launcher" width="100%" />
      <br/><br/>
      <sub>Floating launcher on the teacher portal home.</sub>
    </td>
    <td width="50%" valign="top">
      <b>2. Chat Open &amp; Quick Actions</b>
      <br/><br/>
      <img src="./screenshots/02-chat-open.png" alt="Chat open with quick actions" width="100%" />
      <br/><br/>
      <sub>Chat open with session logging quick actions.</sub>
    </td>
  </tr>
  <tr>
    <td width="50%" valign="top">
      <b>3. Scoped Student Picker</b>
      <br/><br/>
      <img src="./screenshots/03-student-picker.png" alt="Guided session logging student picker" width="100%" />
      <br/><br/>
      <sub>Roster picker for ambiguous names, slots kept.</sub>
    </td>
    <td width="50%" valign="top">
      <b>4. Propose-Then-Confirm Review Card</b>
      <br/><br/>
      <img src="./screenshots/04-review-card.png" alt="Review card before creating session" width="100%" />
      <br/><br/>
      <sub>Review card before draft or submit confirm.</sub>
    </td>
  </tr>
  <tr>
    <td colspan="2" valign="top">
      <b>5. Structured Teaching Hours Report</b>
      <br/><br/>
      <img src="./screenshots/05-hours-report.png" alt="Teaching hours structured answer card" width="100%" />
      <br/><br/>
      <sub>In-chat hours and session count summary cards.</sub>
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
