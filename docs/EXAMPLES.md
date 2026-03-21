# Practical Examples

Real-world scenarios showing exactly what happens when you use the skill. Each example shows the prompt, the decomposition, the wave plan, and the result.

---

## Example 1: "Add a Payment System"

### What You Say

```
Add Stripe payments to our app. Need a checkout flow, webhook handler,
subscription management, and invoice history page.
```

### What Claude Does

**Step 1 — Decompose** (identifies 11 subtasks):

```
 1. Research existing payment patterns in codebase
 2. Research Stripe API for checkout + subscriptions
 3. Define types (Plan, Subscription, Invoice, WebhookEvent)
 4. Create billing tables migration (plans, subscriptions, invoices)
 5. Implement POST /api/checkout/session endpoint
 6. Implement Stripe webhook handler (/api/webhooks/stripe)
 7. Implement GET /api/subscriptions endpoints
 8. Implement GET /api/invoices endpoint
 9. Build CheckoutPage component
10. Build InvoiceHistory component
11. Write tests for webhook handler (critical path)
```

**Step 2 — Map dependencies** (finds the parallelism):

```
                    ┌─────────┐  ┌─────────┐  ┌─────────┐
 INDEPENDENT:       │ 1.Code  │  │ 2.Stripe│  │ 3.Types │
                    │ patterns│  │   docs  │  │         │
                    └────┬────┘  └────┬────┘  └────┬────┘
                         │            │            │
                         ▼            ▼            ▼
                    ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌──────────┐
 NEEDS TYPES:       │ 4.DB    │  │ 5.Check-│  │ 6.Web-  │  │ 7.Subs   │
                    │migration│  │   out   │  │  hook   │  │  API     │
                    └─────────┘  └─────────┘  └─────────┘  └──────────┘
                                                   │
                    ┌─────────┐  ┌─────────┐  ┌────┴────┐
 NEEDS CODE:        │ 8.Invoic│  │ 9.Check-│  │10.Invoic│
                    │  es API │  │ out Page│  │  e Page │
                    └─────────┘  └─────────┘  └─────────┘
                                                   │
                                              ┌────┴────┐
 NEEDS HANDLER:                               │11.Tests │
                                              └─────────┘
```

**Step 3 — Plan waves**:

```
Wave 1 (3 agents, parallel):
  [1] Search codebase for payment patterns
  [2] Fetch Stripe API docs for checkout + subscriptions
  [3] Define TypeScript types

Wave 2 (4 agents, parallel):
  [4] Create billing tables migration
  [5] Build checkout session endpoint
  [6] Build webhook handler
  [7] Build subscription endpoints

Wave 3 (3 agents, parallel):
  [8] Build invoices endpoint
  [9] Build CheckoutPage component
  [10] Build InvoiceHistory component

Wave 4 (2 agents, parallel):
  [11] Webhook handler tests
  [12] Code review + security review

11 tasks in 4 waves. Sequential would be 11 waves.
Speedup: ~2.75x
```

**Step 4 — Execute**: Claude launches 3 agents in Wave 1 (all in a single message), waits for them to finish, then launches 4 agents for Wave 2, and so on.

**Step 5 — Verify**: Runs build, runs tests, confirms 80%+ coverage, dispatches code review and security review agents.

---

## Example 2: "Why Is This Page Slow?"

### What You Say

```
The /dashboard page takes 8 seconds to load. Users are complaining.
Find out why and fix it.
```

### What Claude Does

**The research blitz** — Instead of checking one thing at a time, Claude launches 5 investigation agents simultaneously:

```
Wave 1 (5 agents, parallel):
  ┌─────────────────────────────────────────────────────────────┐
  │  Agent 1: Profile the /dashboard API endpoint               │
  │  Agent 2: Check database queries for N+1 or missing indexes │
  │  Agent 3: Analyze React component render performance        │
  │  Agent 4: Check network waterfall (API calls from frontend) │
  │  Agent 5: Review recent commits to dashboard-related files  │
  └─────────────────────────────────────────────────────────────┘
```

**Wave 2**: All 5 agents report back. Claude synthesizes findings:

```
Root causes found:
  - Agent 2: N+1 query in getUserDashboard() — 47 DB queries for 47 widgets
  - Agent 4: Frontend makes 12 sequential API calls (could be 3 parallel)
  - Agent 3: DashboardGrid re-renders on every widget load (missing memo)
```

**Wave 3**: Fix all three issues in parallel (different files, no conflicts):

```
Wave 3 (3 agents, parallel):
  [1] Add eager loading to getUserDashboard query
  [2] Batch frontend API calls into 3 parallel requests
  [3] Add React.memo to DashboardGrid + child widgets
```

**Wave 4**: Regression tests + performance verification.

**Result**: 8 seconds down to 1.2 seconds. Found 3 issues simultaneously that sequential investigation would have found one at a time.

---

## Example 3: "Set Up a New Project From Scratch"

### What You Say

```
Create a new TypeScript API project with Express, PostgreSQL,
Redis caching, JWT auth, rate limiting, and Docker setup.
```

### What Claude Does

```
Wave 1 (4 agents, parallel) — Foundation:
  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐
  │ Research best   │  │ Define types   │  │ Init project   │  │ Research Redis │
  │ Express patterns│  │ (User, Token,  │  │ (package.json, │  │ caching        │
  │ for 2024       │  │  Config, etc.) │  │  tsconfig,     │  │ patterns       │
  │                │  │                │  │  .env.example)  │  │                │
  └────────────────┘  └────────────────┘  └────────────────┘  └────────────────┘

Wave 2 (5 agents, parallel) — Independent modules:
  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐
  │ PostgreSQL │ │ Redis      │ │ JWT auth   │ │ Rate       │ │ Docker +   │
  │ connection │ │ client +   │ │ middleware │ │ limiter    │ │ Compose    │
  │ + schema   │ │ cache layer│ │ + routes   │ │ middleware │ │ setup      │
  └────────────┘ └────────────┘ └────────────┘ └────────────┘ └────────────┘

Wave 3 (3 agents, parallel) — Integration:
  ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
  │ Wire middleware   │ │ Write tests      │ │ Health check     │
  │ chain (auth →     │ │ (unit + integra- │ │ endpoint +       │
  │ rate limit → etc.)│ │  tion for each)  │ │ graceful shutdown│
  └──────────────────┘ └──────────────────┘ └──────────────────┘

Wave 4 (2 agents, parallel) — Verification:
  ┌──────────────────┐ ┌──────────────────┐
  │ Code review      │ │ Security review  │
  │ (all files)      │ │ (auth, env vars, │
  │                  │ │  Docker security)│
  └──────────────────┘ └──────────────────┘
```

**Result**: Full project scaffolded in 4 waves instead of 14 sequential steps. 47 files created, all tests passing, Docker ready.

---

## Example 4: "Refactor This Monster File"

### What You Say

```
Split auth.ts (900 lines) into separate modules: password.js,
token-management.js, session-handling.js, oauth-providers.js,
permissions.js, and middleware.js
```

### Why This One Is Special

Multiple agents extracting code from the same file will conflict — agent A removes lines 100-200 while agent B is reading line 150. The skill solves this with **worktree isolation**:

```
Wave 1 (3 agents, parallel):
  [1] Analyze auth.ts — map functions to modules
  [2] Identify shared utilities and cross-references
  [3] Run existing tests for green baseline

Wave 2 (1 agent):
  [4] Extract shared utilities → auth/constants.js

Wave 3 (4 agents, parallel, WORKTREES):
  ┌─────────────────────────────────────────────────────────────┐
  │  Each agent gets its own COPY of the entire codebase:       │
  │                                                             │
  │  Agent 5: Extract password functions → auth/password.js     │
  │  Agent 6: Extract token functions → auth/token-management.js│
  │  Agent 7: Extract session functions → auth/session-handling.js│
  │  Agent 8: Extract OAuth functions → auth/oauth-providers.js │
  │                                                             │
  │  No conflicts possible — they're editing different copies.  │
  └─────────────────────────────────────────────────────────────┘

Wave 4: Merge all worktrees back, one at a time, running build between each merge.

Wave 5: Run full test suite — all original tests must still pass.

Wave 6: Code review.
```

**Benchmark result**: 172.9 seconds vs 541.5 seconds without the skill. **3.1x faster**, 60% fewer tokens.

---

## Example 5: "This Is Too Small, Don't Bother"

### What You Say

```
Fix the typo on line 42 of README.md
```

### What Claude Does

```
Task size: TRIVIAL (1 step)
→ Skip decomposition
→ Edit the file directly
→ Done in ~5 seconds
```

The skill adds ~12 seconds of overhead for the sizing check, but does **not** launch agents, create wave plans, or do anything unnecessary. This is by design — the benchmarks confirm that trivial tasks pass at 100% both with and without the skill.

---

## Example 6: "Mostly Serial, But Not Entirely"

### What You Say

```
Set up a CI pipeline: lint → test → build → deploy to staging.
Each step depends on the previous one passing.
```

### What Claude Does

The skill is honest about the serial constraint, but still finds parallelism in the research phase:

```
Wave 1 (3 agents, parallel) — Research:
  [1] Research CI patterns for our stack
  [2] Research deployment targets + staging config
  [3] Identify required environment variables + secrets

Wave 2 → 5 (serial) — Pipeline stages:
  [4] Configure lint step (needs: 1)
  [5] Configure test step (needs: 4)
  [6] Configure build step (needs: 5)
  [7] Configure deploy step (needs: 6, 2, 3)

Honest assessment:
  "The lint→test→build→deploy chain is inherently serial.
   I parallelized the initial research (Wave 1) to gather
   all context simultaneously, then executed the pipeline
   stages in order."
```

---

## Quick Reference: Task Type → Parallelism Pattern

| Task Type | Where Parallelism Lives | Typical Speedup |
|-----------|------------------------|-----------------|
| **Full-stack feature** | Independent modules (DB, API, UI, tests) | 2-3x |
| **Bug investigation** | Research blitz (logs, code, git, docs) | 1.5-2x |
| **Greenfield project** | Module scaffolding after shared types | 2-3x |
| **File refactoring** | Worktree-isolated extractions | **3x+** |
| **Research / analysis** | Parallel queries per topic | 2-3x |
| **Database migration** | Independent table migrations | 1.5-2x |
| **Trivial fix** | None (correctly skipped) | 1x |
| **Serial pipeline** | Research phase only | 1.2-1.5x |
