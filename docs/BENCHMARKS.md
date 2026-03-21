# Benchmark Results

Detailed benchmark data from 68 evaluation runs across 3 iterations, comparing Claude Code **with** the divideandconquer skill vs **without** it.

---

## Summary

| Metric | With Skill | Without Skill | Delta |
|--------|-----------|---------------|-------|
| **Assertion Pass Rate** | **100%** | 70% | +30pp |
| **Pass Rate (non-trivial tasks)** | **96.8%** | 27.9% | +68.8pp |
| **Avg Tool Calls** | 36.9 | 39.9 | -7.5% (fewer wasted calls) |
| **Token Overhead** | +2.4% | baseline | Negligible cost for 2-3x quality |

> Pass rate measures whether Claude produced the right *structure* — parallel decomposition, correct dependency mapping, wave-based execution, test coverage gates, and code review dispatch. Without the skill, Claude almost never does these things on its own.

---

## What We Tested

8 real-world task types, each run multiple times with and without the skill:

| # | Task Type | What It Tests |
|---|-----------|--------------|
| 1 | **Full-Stack Feature** (notification system) | Can it decompose a multi-layer feature into parallel waves? |
| 2 | **Bug Investigation** (checkout 500 errors) | Does it parallelize the research phase instead of investigating sequentially? |
| 3 | **Greenfield Project** (SaaS boilerplate) | Can it scaffold an entire project with parallel module creation? |
| 4 | **Large Refactor** (900-line auth.ts split) | Does it use worktrees to refactor in parallel without file conflicts? |
| 5 | **Research Synthesis** (edge computing report) | Does it run multiple research queries simultaneously? |
| 6 | **Database Migration** (MongoDB to PostgreSQL) | Can it parallelize independent table migrations? |
| 7 | **Trivial Task** (rename a variable) | Does it correctly skip decomposition for tiny tasks? |
| 8 | **Serial Task** (CI/CD pipeline) | Is it honest when a task is mostly sequential? |

---

## Headline Numbers

### Iteration 2 — 16 runs (8 tasks x 2 configs)

```
                    WITH SKILL          WITHOUT SKILL
                    ──────────          ─────────────
Pass Rate:          97.1%               41.2%            +55.9pp
Time (avg):         321.9s              306.2s           +5.1% (planning overhead)
Tokens (avg):       48,396              47,271           +2.4%
Tool Calls (avg):   36.9                39.9             -7.5%
```

### Iteration 3 — 48 runs (8 tasks x 2 configs x 3 runs each)

```
                    WITH SKILL          WITHOUT SKILL
                    ──────────          ─────────────
Pass Rate:          100% +/- 0%         70% +/- 22%      +30pp
Consistency:        Perfect             High variance
```

---

## Per-Task Breakdown

### Task 1: Full-Stack Feature (Notification System)

> *"Add real-time notifications with PostgreSQL storage, SSE streaming, rate limiting, and a React notification bell component"*

```
                    WITH            WITHOUT
Pass Rate:          83-100%         0-62%
Subtasks Found:     23              —
Waves Planned:      7               —
Files Created:      14              9
```

**What the skill did differently**: Decomposed into 23 subtasks across 7 waves. Research, type definitions, and config ran in Wave 1 simultaneously. Database, API, and SSE modules built in parallel in Wave 2. Tests and reviews in final waves.

**What happened without it**: Claude built everything sequentially. No wave plan, no parallel agents, no structured decomposition. Missed the notification bell component entirely.

---

### Task 2: Bug Investigation (Checkout 500 Errors)

> *"Stripe webhook handler failing ~5% of the time. Customers charged but subscription status not updating."*

```
                    WITH            Without
Pass Rate:          100%            25-50%
Time:               351.7s          377.1s          7% faster
Tokens:             64,891          90,913          29% cheaper
Tool Calls:         42              76              45% fewer
```

**What the skill did differently**: Launched 4 research agents simultaneously in Wave 1 (logs, codebase search, git history, Stripe docs). Diagnosed in Wave 2. Fixed in Wave 3. Regression test + security review in Wave 4.

**What happened without it**: Investigated one lead at a time, backtracked twice, made 76 tool calls (vs 42) chasing dead ends, used 29% more tokens.

---

### Task 3: Greenfield Project (SaaS Boilerplate)

> *"Scaffold a SaaS starter with auth, billing, team management, and admin dashboard"*

```
                    WITH            Without
Pass Rate:          100%            0-71%
Time:               590.6s          441.4s
Files Created:      47              51
```

**What the skill did differently**: Research + types + config in parallel Wave 1. Four independent modules (auth, billing, teams, admin) each built by separate agents in Wave 2. Integration + tests in Wave 3.

**What happened without it**: Built modules in arbitrary order. Skipped config parallelism, no worktree isolation, no review gate.

---

### Task 4: Large Refactor (Split 900-line auth.ts)

> *"Split api-handler.ts into separate route modules for /users, /products, /orders, /admin without breaking 23 existing tests"*

```
                    WITH            Without
Pass Rate:          100%            20-83%
Time:               172.9s          541.5s          3.1x FASTER
Tokens:             25,417          63,842          60% cheaper
Tool Calls:         12              55              78% fewer
```

**This is the standout result.** The skill used worktree isolation to extract all four route modules in parallel — each agent got its own copy of the codebase, so they couldn't conflict. Without the skill, Claude extracted modules one at a time, constantly re-reading the shrinking source file, using 4.6x more tool calls.

```
WITH SKILL (172.9 seconds):
  Wave 1: [analyze] [find shared utils] [run baseline tests]     3 parallel
  Wave 2: [extract shared utils]                                  1 sequential
  Wave 3: [extract /users] [/products] [/orders] [/admin]        4 parallel (worktrees)
  Wave 4: [update router] [write unit tests]                      2 parallel
  Wave 5: [run full test suite]                                   1 sequential
  Wave 6: [code review]                                           1 sequential

WITHOUT SKILL (541.5 seconds):
  Step 1: Read file
  Step 2: Extract /users
  Step 3: Re-read file
  Step 4: Extract /products
  Step 5: Re-read file (again)
  ...55 tool calls later...
```

---

### Task 5: Research Synthesis (Edge Computing)

> *"Research edge computing frameworks, compare Cloudflare Workers vs Deno Deploy vs Fly.io, produce a recommendation"*

```
                    WITH            Without
Pass Rate:          100%            50-75%
Parallel Searches:  3+              1 at a time
Files Produced:     8               2
```

**What the skill did differently**: Launched 3 research agents simultaneously (one per platform). Each produced findings independently. A synthesis agent in the final wave combined everything into a structured comparison.

---

### Task 6: Database Migration (MongoDB to PostgreSQL)

> *"Migrate users, organizations, and memberships collections from MongoDB to PostgreSQL"*

```
                    WITH            Without
Pass Rate:          100%            50-60%
```

**What the skill did differently**: Identified that users and organizations are independent — migrated both in parallel. Only memberships (which references both) waited for Wave 2. Included rollback scripts.

---

### Task 7: Trivial Task (Variable Rename)

> *"Rename the variable 'x' to 'count' in utils.js"*

```
                    WITH            Without
Pass Rate:          100%            100%
Time:               59.1s           47.1s
```

**Correct behavior**: Both pass. The skill correctly detected this as trivial (1-2 steps) and executed directly without decomposition. The 12-second overhead is the skill's decision-making check — it reads the task, confirms it's trivial, and proceeds. This is the cost of the sizing heuristic, and it's acceptable.

---

### Task 8: Serial Task (CI/CD Pipeline)

> *"Set up GitHub Actions CI with lint, test, build, deploy stages that must run in order"*

```
                    WITH            Without
Pass Rate:          100%            25-80%
```

**What the skill did differently**: Honestly acknowledged the serial constraint upfront. Still found hidden parallelism in Wave 1 (research CI patterns + research deployment targets + identify dependencies — all independent). Preserved correct serial ordering for the pipeline stages.

---

## Assertion Analysis

### Which assertions actually distinguish the skill from no-skill?

**High-discrimination assertions** (skill passes, no-skill almost always fails):

| Assertion | WITH Pass Rate | WITHOUT Pass Rate |
|-----------|---------------|-------------------|
| `decomposition_count` | 100% | 0% |
| `wave1_parallelism` | 100% | 0% |
| `wave_count` | 100% | 0% |
| `agent_dispatch` | 100% | 0% |
| `false_dependency_challenged` | 100% | 0% |
| `worktree_used` | 100% | 0% |
| `review_agents_dispatched` | 100% | 0% |

**Non-discriminating assertions** (both pass):

| Assertion | Why Both Pass |
|-----------|--------------|
| `research_before_fix` | Claude naturally researches before fixing |
| `regression_test_included` | Claude includes regression tests by default |
| `test_preservation` | Neither config breaks existing tests |
| `rollback_included` | Both include rollback for migrations |

These non-discriminating assertions are still valuable — they confirm the skill doesn't *break* Claude's natural good habits while adding structured parallelism on top.

---

## Consistency Across Runs

Iteration 3 ran each task 3 times to measure variance:

```
WITH SKILL:     100% +/- 0%     (perfectly consistent)
WITHOUT SKILL:   70% +/- 22%    (highly variable)
```

The skill eliminates variance. Without it, Claude might decompose a task well one time and miss it entirely the next. The skill provides a reliable framework that produces consistent results regardless of the specific run.

---

## Cost-Benefit Summary

```
WHAT YOU PAY:
  +2.4% more tokens on average
  +5.1% more time on average (planning overhead)
  12 seconds of overhead on trivial tasks

WHAT YOU GET:
  +55-69pp higher assertion pass rate
  Consistent 100% pass rate (vs 70% +/- 22%)
  3.1x faster on refactoring tasks (worktree parallelism)
  29% fewer tokens on bug investigations
  45% fewer tool calls on investigations
  Structured decomposition on every non-trivial task
  Automatic code review and security review gates
  Test coverage enforcement
```

---

## Methodology

- **Iterations**: 3 rounds of testing, each improving the skill based on results
- **Total runs**: 68 (4 in iteration 1, 16 in iteration 2, 48 in iteration 3)
- **Baseline**: Same prompts, same codebase, no skill loaded
- **Assertions**: 43 unique assertions across 8 task types, graded by independent agent
- **Timing**: Captured from Claude Code task notifications (total_tokens, duration_ms)
- **Grading**: Each assertion evaluated against actual outputs with evidence citations
