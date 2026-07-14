# AI Code Coach — Case Study

> **An AI coding tutor that reads a student's code and coaches them — without ever writing it for them.**
> 🚧 In active development. This repository is a written case study of the system's design.

<p align="center">
  <img src="assets/acc-agent-banner.png" alt="AI Code Coach — the tutor that never writes your code for you" width="100%">
</p>

> [!NOTE]
> **This repo is a static case study — Markdown and diagrams only.**
> There is no application server here, no API keys, and no product screenshots.
> The coach runs as an agent inside [Claude Code](https://claude.com/claude-code);
> a live walkthrough is available on request.

---

## The problem

Most ways to learn to code fail in the same way: they hand you the answer. A YouTube tutorial types it for you. A bootcamp paces it for someone else. And a general-purpose AI assistant will happily write the whole function — which feels productive and teaches you nothing. People watch hundreds of hours and still go blank the moment they have to write code themselves.

I teach people to code, and I kept watching the same thing happen. **AI Code Coach is the tutor I wished my students had:** it reads what they wrote, asks the right question, and makes them write the next line themselves.

---

## What it is

An **adaptive tutoring agent that runs inside a real terminal** (Claude Code). Because it lives where the code lives, it can do what a screen-recorded course or a chat window can't: create the lesson files, run the student's actual code, and read their real `git` and test output — then grade against what the program *did*, not what the student claims it does.

There is no separate web app, database, or billing to run. The entire coach is a **rulebook plus a file-based memory of the learner** — a ~480-line operating contract (`CLAUDE.md`) the agent loads every session, a live progress tracker it reads and writes, graded review files co-located with each exercise, and a persistent cross-session memory of who the learner is and how they learn best. That's the product: the *behavior*, encoded as enforceable rules.

<p align="center">
  <img src="assets/acc-teaching-loop.png" alt="The four-beat teaching loop: Tell, Show, You Do, I Check" width="100%">
</p>

---

## How the prompt scaffolds the whole project

This is the part that surprises people: **there is no application to deploy.** The entire coach is a single prompt file — a ~480-line, ~5,600-word `CLAUDE.md` operating contract — that the agent loads at the top of every session. That one file *is* the product. It doesn't just describe behavior; it dictates exactly what the agent must do, refuse, write, and where.

### The rulebook: ~30 sections, each an enforceable contract

`CLAUDE.md` reads less like documentation and more like a constitution. It's organized into roughly **30 titled sections**, and each one is a rule the agent has to obey on every turn — not a suggestion it can drift from. A few of them:

- **Session start** — read `progress.md` first, resume from the exact "Next up," never ask "where were we?"
- **Learn-mode gate** — teaching is *locked* until the student says `start course`; open-mode Q&A can't advance the course.
- **Four-beat loop** — every lesson runs Tell → Show → You Do → I Check, with real-output verification.
- **No-answers rule** — never write the student's code; help only with a different-scenario example.
- **Verify-before-handoff** — a hard gate: compile and run every snippet before the student is allowed to see it.
- **Grading rubric + advancement gate** — a 0–100 score, and no advancing below 60 or with an unresolved blocker.
- **Learner guardrails (×5)** — silent detectors for frustration, copy-paste, perfectionism, and vocab gaps.
- **File map** — the canonical spec for what gets written and *where*.

<p align="center">
  <img src="assets/acc-rulebook.png" alt="Anatomy of the CLAUDE.md rulebook — 30 sections, each an enforceable rule the agent must follow" width="100%">
</p>

Because the behavior lives in the prompt, **changing how the coach teaches is a documented, version-controlled edit — not a code deploy.** The git history of `CLAUDE.md` is a literal changelog of pedagogy: every rule traces back to a real moment the teaching needed to be tightened.

### What that single prompt spawns

The rulebook's *File map* section turns the prompt into a filesystem. From those instructions alone, the agent scaffolds a whole course as a disciplined tree — and holds itself to the exact folder standards it grades the student on: one responsibility per file, kebab-case names, module folders, and a graded `review.md` co-located with every piece of work.

So far, that one prompt has grown into **40 lesson folders, 54 graded reviews, and 8 shared doc files, backed by 59 persistent memory entries** — all following a structure the prompt specified up front.

<p align="center">
  <img src="assets/acc-scaffold-tree.png" alt="The file tree the prompt scaffolds: rulebook, hot and cold docs, per-module lesson folders, co-located reviews" width="100%">
</p>

### Booting a session from files alone

The agent is stateless between sessions, so the prompt has to *rebuild the entire teaching context from files* every time — and it does, in a fixed order the rulebook mandates: load the rulebook and live state, restore who the learner is from persistent memory, gate the mode, resume at the exact "Next up," teach, then write the new state back. Because the last thing every loop does is persist state, a session can be cleared at any breakpoint and the next one boots clean and **lossless**.

<p align="center">
  <img src="assets/acc-session-bootstrap.png" alt="Session bootstrap sequence: load rulebook, restore learner, gate mode, resume, teach and save" width="100%">
</p>

This is the whole architecture in one sentence: **a prompt that scaffolds a course, grades against real output, and rebuilds its own context from disciplined files** — no server, no database, just rules and state.

---

## How the AI actually teaches

The coaching behavior is the product, and it's enforced by an operating contract the agent must follow every session — not left to the model's mood.

**1. The no-code rule (a hard constraint).** The agent will not write, complete, or fill in a single line of the student's code. Mid-exercise help explains the *concept* with a worked example in a **completely different domain**, with different variable names, so the exercise answer can't leak. The real solution appears **only** in the post-submission review — and there's an explicit trap for the obvious workaround: try to fish the answer out by leaving "learn mode" and the agent recognizes it and redirects.

**2. The four-beat loop, every lesson.** **TELL** (teach from first principles, define jargon before use) → **SHOW** (a readable lesson plus a runnable, commented example in a *different* scenario than the exercise) → **YOU DO** (the student writes every line in their own file, with an exact run command and expected output) → **I CHECK** (the agent runs the code and grades the *real output* against the requirement). The governing rule: **"it ran" is never the same as "it's correct."**

**3. It grades on evidence, not vibes.** Every submission is scored on a 0–100 rubric (Correctness · Tests · Standards · Readability · Concept grasp) written to a `review.md` file co-located with the work. Skills move through `not started → learning → practiced → mastered`, and a skill is only *mastered* after **two clean, un-hinted reps on separate, spaced occasions.**

**4. Advancement gates that won't let you skip.** The student advances past a lesson only when they score **≥ 60 and have no blocking skill still below `practiced`.** Below that, the agent re-teaches the failing micro-skill in a fresh scenario; on proven strength it *accelerates* — compressing drills or offering a test-out. The pace adapts in **both** directions.

**5. It stays on the lesson.** A learn-mode gate keeps the agent scoped to the current lesson and already-taught material; off-topic asks are parked on the roadmap and redirected, instead of turning into a free-for-all chatbot.

**6. Silent learner guardrails.** Always-on detectors watch for frustration (stop advancing, shrink to a winnable step), copy-paste / AI-submitted work (met with *"walk me through line N — what does it do and why?"*), perfectionism (every task is time-boxed), and vocabulary gaps (periodic "define X vs Y" checks) — each one changes *how* the agent teaches on the fly.

<p align="center">
  <img src="assets/acc-mastery-gate.png" alt="Mastery ladder and advancement gate: skills move from not started to mastered only on spaced, un-hinted evidence" width="100%">
</p>
<p align="center"><em>Every skill is scored, and no one advances past a lesson without the evidence to back it.</em></p>

---

## Architecture

The "architecture" here is deliberately not a server diagram — it's a **context and state design.** The agent is stateless between sessions; continuity comes entirely from a small set of disciplined files it reads and writes.

<p align="center">
  <img src="assets/acc-architecture.png" alt="State design: the agent reads CLAUDE.md, progress.md, and persistent memory, and writes lessons, runs code, and graded reviews" width="100%">
</p>

```mermaid
flowchart TD
  S[Student in the terminal] --> AGENT[Claude Code agent]
  RULES[CLAUDE.md — operating contract] --> AGENT
  PROG[progress.md — live state, mastery, spacing] --> AGENT
  MEM[Persistent memory — who the learner is, how they learn] --> AGENT
  AGENT --> FILES[Authors lesson.md + example + exercise]
  AGENT --> RUN[Runs the student's real code and tests]
  RUN --> REVIEW[Writes a graded review.md beside the work]
  REVIEW --> PROG
  AGENT --> GIT[Reads git status and log — never commits for the student]
```

Every lesson runs the same disciplined loop, and state is persisted so a session can be cleared and resumed losslessly:

```mermaid
flowchart TD
  A[Session starts] --> B[Load CLAUDE.md rules and progress.md state]
  B --> C[Resume at the exact Next up point]
  C --> D[TELL: teach the concept from first principles]
  D --> E[SHOW: lesson.md plus runnable example, different scenario]
  E --> F[YOU DO: student writes every line in their own file]
  F --> G[I CHECK: run the real code, verify output vs requirement]
  G --> H[Write graded review.md, update progress.md and spacing]
  H --> I{Score >= 60 and no blockers?}
  I -- yes --> J[Advance, or accelerate on proven mastery]
  I -- no --> D
```

That ordering — **load state → teach → student builds → verify real output → grade → gate** — is a project-wide rule, on every lesson, no exceptions.

---

## Notable design decisions

- **The rulebook is the source of truth.** All coaching behavior — the no-code rule, the grading rubric, the enforcement ramp, the guardrails — lives in one versioned `CLAUDE.md` contract the agent reloads every session. Changing how the coach behaves is a documented edit, not a code deploy.
- **State is files, not a database.** A single hot `progress.md` holds current state, skill mastery, focus areas, and a spaced-repetition schedule; graded reviews are co-located with the work they grade. Anyone can read the whole story of one unit of work by opening one folder.
- **Persistent, cross-session learner memory.** A durable memory store (~60 entries and growing) records who the learner is and how they learn — preferences, recurring error patterns, format asks, frustration triggers — so coaching compounds across sessions instead of resetting each time.
- **Context is a budget.** Only the two lean hot files load every session; deep pedagogy, history, and per-topic playbooks live in cold files read on demand, and the agent flags natural `/clear` points once state is saved — keeping the working context sharp and cheap.
- **Verify before handing anything over.** A hard gate forbids showing the student code the agent hasn't compiled and run: every snippet is type-checked and executed, shared identifiers are cross-checked across spec / test / starter files, and the verification receipt is shown in-message. Eyeballing adapted example code is banned.
- **Fresh docs, pulled live — never taught from memory.** Before any lesson touches a library, its current API is pulled from the official documentation through **Context7 (MCP)** — a live docs connection — and pinned to the non-deprecated version, with the source and version noted at the bottom of the lesson. Package *names* are verified too, not just APIs (e.g. `@dnd-kit/*` not `react-beautiful-dnd`, `motion` not `framer-motion`, TanStack Query v5's object-arg API). If a Context7 lookup fails, the agent falls back to fetching the official docs directly. This is why the course never drifts onto a stale tutorial's version of an API — and it's paired with coaching the student to decode the real hover signature themselves, so their reliance on AI *drops*.
- **Taught to real industry demand, checked against live listings.** A `market check` command pulls current signal from four trusted sources — the Stack Overflow Developer Survey, Hacker News "Who is Hiring," roadmap.sh, and LinkedIn job postings — writes a dated `market-demand.md` snapshot, and outputs a gap analysis of the learner's skills versus what AI-app roles are actually hiring for (on-track / watch / out-of-scope). The finding feeds back into the curriculum: **module depth is weighted toward high-demand, within-stack skills**, so time is spent where the job market spends it, not on what's merely fashionable.
- **Git stays in the student's hands.** The agent teaches and verifies version control but never runs `git` for the student — every submission is their own commit on their own branch.

---

## The pedagogy, encoded

| Mechanism | What it enforces |
|---|---|
| **No-answers rule** | Different-scenario help only; the solution appears solely in the review |
| **Four-beat loop** | TELL → SHOW → YOU DO → I CHECK, with real-output verification |
| **0–100 rubric** | Correctness 35 · Tests 15 · Standards 20 · Readability 15 · Concept 15 |
| **Mastery ladder** | `not started → learning → practiced → mastered` on objective, spaced evidence |
| **Advancement gate** | Advance only at ≥ 60 with no blockers; remediate or accelerate |
| **Enforcement ramp** | A small graded core early; each standard switched on (and announced) at its home module |
| **Spaced repetition** | A spacing schedule resurfaces due topics via `drill me` / `leet me` / `test your skills` |
| **Module-exit gate** | No module completes without exercise + drill + syntax drill + one cohesive program, each graded |

---

## The curriculum it drives

A fixed 17-rung dependency ladder (M0–M16) from foundations to interview-ready in the AI niche, each rung a real, résumé-worthy capability milestone:

<p align="center">
  <img src="assets/acc-curriculum-ladder.png" alt="The 17-rung curriculum: foundations, frontend, backend and data, production, and the AI niche" width="100%">
</p>

**Foundations & language** — beginner on-ramp · JavaScript · TypeScript
**Frontend** — web platform / HTML-CSS-a11y · React · React application architecture (Zustand, TanStack Query & Table, RHF + Zod, dnd-kit, Motion)
**Backend & data** — type-safe Express · Next.js App Router full-stack · Postgres + Drizzle on Neon · authentication & authorization
**Production** — real-world plumbing (uploads, email, webhooks, rate limiting) · testing & quality (Vitest, Testing Library, Playwright) · security & performance hardening · deployment, CI/CD & prod ops
**The AI niche** — AI foundations (Vercel AI SDK) · RAG & embeddings (pgvector) · agents, tools & evals — ending in a **three-app deployed AI portfolio** and a DSA / interview-readiness sprint.

Spiral threads (git, self-written tests, debugging, accessibility, spaced repetition, security mindset, requirements gathering, data-analysis mindset, and more) are woven through **every** module rather than taught once. The first live page ships in week one.

---

## Status & contact

🚧 **In active development.** I'm happy to give a live walkthrough of the agent teaching a real lesson end to end.

📫 **Joe Letner** — jrletner@gmail.com · [LinkedIn](https://www.linkedin.com/in/joe-letner-4a37ba99/)
