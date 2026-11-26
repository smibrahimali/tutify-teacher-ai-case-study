<div align="center">

<img src="./assets/banner.svg" alt="MaryamBot - Tutify Portal Engineering Case Study" width="100%" />

<br/><br/>

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

> **Confidentiality Notice:** Tutify Portal is a proprietary production application. Per company guidelines, this repository contains no source code—only a high-level engineering breakdown of the problem, architecture, verification practices, and local synthetic-data screenshots.

<br/>

### ✦ The Problem & Workflow Shift

Logging completed lessons on the Tutify teacher portal previously required hopping across multiple screens—opening a student, selecting an enrollment cycle, entering duration or Pakistan Standard Time (`Asia/Karachi`) windows, and saving or submitting for approval—while checking hours and earnings on separate views.

**MaryamBot** brings those workflows into a single, role-scoped chat sheet inside `portal.tutify.org`:

| Dimension | Legacy Multi-Screen Portal | MaryamBot Guided Flow |
| :--- | :--- | :--- |
| **Session Logging** | 4+ screen transitions per lesson | Guided chat flow (`Student → PKT Date → Time → Review Card`) |
| **Ambiguous Names** | Manual search and re-entry | Inline disambiguation picker preserving captured time slots |
| **Write Safety** | Direct form submission | Two-step **Propose → Review Card → Atomic Confirm** |
| **Hours & Earnings** | Separate dashboard pages | Instant read-only summary cards inside the same thread |

**Strict Domain Boundaries:** Teacher-only access. Explicitly declines guardian, student, or admin workflows, billing mutations, future-dated logs, quizzes, PTMs, and off-topic chat.

<br/>

### ✦ System Architecture & How I Solved It

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