# User Guide

Everything you need to get started, from installation to advanced usage.

---

## Table of Contents

- [Quick Start](#quick-start)
- [How It Works (Visual Walkthrough)](#how-it-works-visual-walkthrough)
- [Installation](#installation)
  - [Claude Code Skill](#claude-code-skill)
  - [OpenClaw Skill](#openclaw-skill)
  - [Global Rule (Auto-Activation)](#global-rule-auto-activation)
- [Usage Patterns](#usage-patterns)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)

---

## Quick Start

```bash
# 1. Clone
git clone https://github.com/RuneweaverStudios/divideandconquer.git

# 2. Install
cp -r divideandconquer/claude-code-skill ~/.claude/skills/divideandconquer

# 3. Use
# Just ask Claude to build something. The skill activates automatically.
```

That's it. Claude will now decompose and parallelize complex tasks automatically.

---

## How It Works (Visual Walkthrough)

### Without the Skill

You ask: *"Add notifications with a database, API, real-time streaming, and tests"*

Claude works through it like a single-lane road:

```
TIME ──────────────────────────────────────────────────────────►

 ┌──────────┐┌──────────┐┌──────────┐┌──────────┐┌──────────┐┌──────────┐
 │ Research  ││  Types   ││ Database ││   API    ││   SSE    ││  Tests   │
 │          ││          ││          ││          ││          ││          │
 └──────────┘└──────────┘└──────────┘└──────────┘└──────────┘└──────────┘

 Total: 6 steps x ~60s each = ~360 seconds
```

### With the Skill

Same request, but Claude maps the dependencies and runs independent work simultaneously:

```
TIME ──────────────────────────────────────────────►

 WAVE 1    ┌──────────┐
 (parallel)│ Research  │
           ├──────────┤
           │  Types   │
           ├──────────┤
           │  Config  │
           └──────────┘
                │
 WAVE 2    ┌──────────┐
 (parallel)│ Database │
           ├──────────┤
           │   API    │
           ├──────────┤
           │   SSE    │
           └──────────┘
                │
 WAVE 3    ┌──────────┐
 (parallel)│  Tests   │
           ├──────────┤
           │  Review  │
           └──────────┘

 Total: 3 waves x ~60s each = ~180 seconds (2x faster)
```

### The Dependency Graph

This is what Claude builds internally before executing:

```
  ┌──────────┐     ┌──────────┐     ┌──────────┐
  │ Research  │     │  Types   │     │  Config  │
  └─────┬────┘     └────┬─────┘     └──────────┘
        │               │
        ▼               ▼
  ┌──────────┐     ┌──────────┐     ┌──────────┐
  │   SSE    │     │ Database │     │   API    │
  │(needs    │     │(needs    │     │(needs    │
  │research) │     │ types)   │     │ types)   │
  └─────┬────┘     └────┬─────┘     └────┬─────┘
        │               │                │
        └───────────┬───┘────────────────┘
                    ▼
              ┌──────────┐
              │  Tests   │
              │(needs    │
              │ all code)│
              └─────┬────┘
                    ▼
              ┌──────────┐
              │  Review  │
              └──────────┘
```

Arrows mean "depends on". Everything at the same vertical level runs simultaneously.

---

## Installation

### Claude Code Skill

#### Option 1: Copy (simple)

```bash
git clone https://github.com/RuneweaverStudios/divideandconquer.git
cp -r divideandconquer/claude-code-skill ~/.claude/skills/divideandconquer
```

#### Option 2: Symlink (auto-updates with `git pull`)

```bash
git clone https://github.com/RuneweaverStudios/divideandconquer.git ~/tools/divideandconquer
ln -s ~/tools/divideandconquer/claude-code-skill ~/.claude/skills/divideandconquer
```

#### Verify

Start a new Claude Code session and type `/divideandconquer`. If the skill appears, you're set.

### OpenClaw Skill

```bash
git clone https://github.com/RuneweaverStudios/divideandconquer.git
cp -r divideandconquer/openclaw-skill/divideandconquer /path/to/your/openclaw/skills/
```

Requirements: Python 3.10+ (standard library only, no pip installs needed).

### Global Rule (Auto-Activation)

By default, Claude uses the skill when it detects a qualifying task. To make it *always* think in parallel first, add a global rule:

Create `~/.claude/rules/common/divideandconquer.md`:

```markdown
# Divide and Conquer — Default Execution Strategy

For ANY non-trivial task (3+ steps, multiple files, or independent subtasks),
use the `divideandconquer` skill BEFORE starting implementation.
Parallel is the default. Serial requires justification.
```

---

## Usage Patterns

### Pattern 1: "Just Build It"

You don't need to say anything special. Just describe what you want:

```
> Add a user settings page with profile editing, password change,
  notification preferences, and account deletion
```

The skill activates automatically and you'll see an execution plan before any code is written.

### Pattern 2: Explicit Trigger

Invoke it directly:

```
> /divideandconquer Build a REST API for managing blog posts with
  CRUD operations, pagination, search, and auth
```

### Pattern 3: Research Tasks

Works for non-coding tasks too:

```
> Compare React Server Components vs Astro vs Remix for our
  e-commerce site. Need performance data, DX comparison, and
  a recommendation.
```

The skill launches parallel research agents — one per framework — then synthesizes findings.

### Pattern 4: Bug Investigation

```
> Users are reporting 500 errors on the checkout page. It happens
  intermittently, maybe 5% of requests. Fix it.
```

The skill launches parallel investigation agents: logs, codebase search, git history, and docs lookup. Then converges on a diagnosis.

### Pattern 5: Override for Small Tasks

The skill automatically skips decomposition for trivial tasks. But if you want to force serial execution:

```
> Do this sequentially: refactor the auth module to use JWT
```

---

## Configuration

### OpenClaw: config.json

The OpenClaw version includes `config.json` for fine-tuning:

```json
{
  "routing": {
    "tiers": {
      "research":       { "model": "haiku",  "max_tokens": 4096  },
      "implementation": { "model": "sonnet", "max_tokens": 8192  },
      "architecture":   { "model": "opus",   "max_tokens": 16384 }
    }
  },
  "execution": {
    "max_parallel_agents": 6,
    "agent_timeout_seconds": 300,
    "retry_on_failure": true,
    "max_retries": 2
  },
  "decomposition": {
    "min_subtasks_for_parallel": 3,
    "max_subtasks": 25,
    "trivial_threshold": 2
  }
}
```

| Setting | What It Controls | Default |
|---------|-----------------|---------|
| `max_parallel_agents` | Max agents per wave | 6 |
| `agent_timeout_seconds` | How long before an agent is killed | 300s |
| `trivial_threshold` | Tasks with fewer steps skip decomposition | 2 |
| `min_subtasks_for_parallel` | Minimum subtasks to trigger parallelization | 3 |
| `max_subtasks` | Cap on decomposition depth | 25 |

### Claude Code: Sizing Heuristics

The Claude Code version uses built-in sizing logic (no config file needed):

| Task Size | Behavior |
|-----------|----------|
| Trivial (1-2 steps) | Executes directly, no overhead |
| Small (3-5 steps) | Quick decomposition, 2-3 agents if obvious |
| Medium (5-10 steps) | Full wave plan, shown to you before execution |
| Large (10+ steps) | Full plan with checkpoints between phases |

---

## Troubleshooting

### "The skill didn't activate"

1. Check it's installed: `ls ~/.claude/skills/divideandconquer/SKILL.md`
2. The task might be too small. The skill skips trivial tasks (1-2 steps) by design.
3. Add the global rule (see [Global Rule](#global-rule-auto-activation)) to make it more aggressive about activating.

### "Claude planned but didn't launch agents"

This happens when Claude falls back to executing the plan itself instead of dispatching subagents. The skill explicitly instructs against this, but it can happen. Nudge it:

```
> Execute the wave plan using parallel agents, not sequentially
```

### "Agents conflicted on the same file"

The skill should use worktree isolation when agents might edit the same file. If it didn't:

```
> Use worktree isolation for agents that touch the same files
```

### "The plan has too many waves"

Sometimes the decomposition is too granular. Tell Claude to consolidate:

```
> Combine small related subtasks into fewer, chunkier waves
```

### "It's slower than doing it without the skill"

Expected for:
- **Trivial tasks** (1-2 steps): ~12 seconds of overhead from the sizing check
- **Mostly-serial tasks**: The planning overhead doesn't pay off when there's little parallelism

Not expected for:
- **Medium/large tasks with independent subtasks**: If it's slower, the decomposition may be wrong. Check the dependency map for false dependencies.
