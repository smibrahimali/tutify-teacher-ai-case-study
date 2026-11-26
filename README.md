<div align="center">

<img src="./assets/banner.svg" alt="MaryamBot — Tutify Portal Engineering Case Study" width="100%" />

<br/><br/>

<sub>System design, architecture, and UI walkthrough of the teacher-facing AI assistant shipped into <a href="https://portal.tutify.org/">portal.tutify.org</a> during my Software Engineering internship.</sub>

<br/><br/>

<a href="https://portal.tutify.org/">
  <img src="https://img.shields.io/badge/LIVE_PORTAL_↗-E4E4E7?style=for-the-badge" alt="Live Portal" />
</a>&nbsp;
<a href="https://github.com/smibrahimali">
  <img src="https://img.shields.io/badge/BACK_TO_PROFILE_↗-161B22?style=for-the-badge" alt="Back to Profile" />
</a>

</div>

<br/>

> **Confidentiality Notice:** Tutify Portal is a proprietary application. Per company guidelines, this repo contains no source code—only a high-level architectural breakdown and local synthetic-data screenshots.

<br/>

### ✦ The Problem & Workflow Shift

Teachers previously had to hop across multiple screens (`My Students → Profile → Enrollment → Add Session`) to log a lesson, while checking hours and earnings on separate pages. **MaryamBot** brings those workflows into a single, role-scoped chat sheet inside the teacher portal.

| Workflow | Legacy Multi-Screen Portal | MaryamBot Guided Flow |
| :--- | :--- | :--- |
| **Session Logging** | 4+ screen transitions per lesson | Guided chat flow (`Student → PKT Date → Time → Review Card`) |
| **Ambiguous Names** | Manual search and re-entry | Inline disambiguation picker preserving entered time slots |
| **Write Safety** | Direct form submission | Two-step **Propose → Review Card → Atomic Confirm** |
| **Hours & Earnings** | Separate dashboard tabs | Instant read-only summary cards inside the same thread |

**Strict Scope Boundaries:** Teacher-only access. Explicitly declines guardian, student, or admin workflows, billing mutations, future-dated logs, quizzes, PTMs, and off-topic chat.

<br/>

### ✦ System Architecture

```mermaid
flowchart LR
    A[Teacher Input] --> B{Layer 1: Heuristics}
    B -- Chip / Flow --> D[Server RBAC & Scope Resolver]
    B -- Free-Form --> C[Layer 2: Groq JSON + Zod]
    C --> D
    D -- Read Query --> E[Hours / Roster / Cycle Card]
    D -- Log Intent --> F[Pending Action + Review Card]
    F -- Confirm Click --> G[Atomic Claim + PKT & Scope Check]
    G -- Valid --> H[Insert Session + Invalidate Cache]
    G -- Error --> I[Release Claim for Safe Retry]