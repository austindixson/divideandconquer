# Decomposition Heuristics & Reference

## Task-Type Wave Templates

### Feature Implementation
```
Wave 1: [types/interfaces] [test skeletons] [config/env setup] [research/docs lookup]
Wave 2: [implementation modules - each independent file in parallel] [mock data/fixtures]
Wave 3: [integration points] [fill in test assertions]
Wave 4: [code review] [security review] [build verification]
```

### Bug Investigation
```
Wave 1: [read error logs] [search codebase for related code] [check recent git changes] [look up docs]
Wave 2: [analyze each suspect area in parallel]
Wave 3: [implement fix] [write regression test]
```

### Refactoring
```
Wave 1: [analyze current code] [identify all callers/dependents] [check test coverage]
Wave 2: [refactor independent modules in parallel using worktrees]
Wave 3: [update integration points] [update tests]
Wave 4: [verify build] [run full test suite]
```

### Research / Analysis
```
Wave 1: [search topic A] [search topic B] [search topic C] [fetch relevant docs]
Wave 2: [deep-dive findings from Wave 1 - each in parallel]
Wave 3: [synthesize and present]
```

### Project Setup
```
Wave 1: [init project structure] [research best practices] [identify dependencies]
Wave 2: [install deps] [create config files] [set up CI] [create initial types]
Wave 3: [scaffold main modules in parallel] [write initial tests]
```

## Anti-Patterns to Avoid

- **Over-decomposition**: Don't split a 5-line function into 3 agents. Use agents for meaningful chunks of work.
- **False parallelism**: Don't launch agents that will immediately block waiting for each other's files.
- **Context duplication**: Don't have 5 agents all read the same 10 files. Have one agent do the reading and share findings.
- **Ignoring the merge**: Parallel execution is pointless if merging the results takes longer than serial would have.
- **Skipping the plan**: Always show the user the wave plan before executing. They may spot dependencies you missed or want to adjust scope.

## Worked Example: Analytics Endpoint

**User request**: "Add a new /analytics endpoint that reads from the events table, includes rate limiting, and has full test coverage"

**Decomposition**:
```
1. Research: Check existing API patterns in codebase
2. Research: Look up rate limiting middleware in use
3. Types: Define analytics request/response types
4. DB: Write the events table query function
5. Handler: Implement the /analytics route handler
6. Middleware: Configure rate limiting for the endpoint
7. Tests: Unit tests for query function
8. Tests: Integration tests for the endpoint
9. Tests: Rate limiting tests
```

**Dependency map**:
```
1 → []        (root)
2 → []        (root)
3 → []        (root - types can be defined from requirements alone)
4 → [1, 3]    (needs patterns + types)
5 → [1, 3]    (needs patterns + types)
6 → [2]       (needs rate limit research)
7 → [4]       (needs query function)
8 → [5, 6]    (needs handler + middleware)
9 → [6]       (needs middleware)
```

**Execution plan**:
```
Wave 1 (parallel): [1] [2] [3]           — 3 agents: research + types
Wave 2 (parallel): [4] [5] [6]           — 3 agents: implementation
Wave 3 (parallel): [7] [8] [9]           — 3 agents: tests
Wave 4 (parallel): [10] code-review [11] security-review — 2 review agents
                                           Total: 11 tasks in 4 waves (~2.75x speedup)
```

## Platform Notes

| Platform | Agent Dispatch | Worktrees | Background Agents |
|----------|---------------|-----------|-------------------|
| **Claude Code CLI** | Full Agent tool with all subagent types | Yes | Yes |
| **Claude.ai** | No subagents — execute waves sequentially yourself | No | No |
| **OpenClaw / API** | Use `sessions_spawn` for parallel agents | Depends on runtime | Depends on runtime |

On platforms without subagents, the skill still provides value through structured decomposition and wave planning — you execute each wave's tasks yourself in optimal order.
