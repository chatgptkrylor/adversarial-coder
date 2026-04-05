# Adversarial Coder

A GAN-inspired three-agent harness that separates **planning**, **building**, and **evaluation** into distinct AI agents. The evaluator's job is to **break** what the generator builds — creating adversarial tension that drives quality far beyond what a single agent can achieve.

## Why This Works

Most AI coding agents fail on complex tasks because a single agent that plans, builds, and evaluates its own work will reliably praise its own mediocre output. This is called **self-evaluation bias**.

This harness implements the fix: three agents, each with a focused job.

| Agent | Role | Model |
|-------|------|-------|
| **Planner** | Expands prompt into spec with sprints | Gemini (fast, cheap) |
| **Generator** | Builds features, commits to git | Codex (best coder) |
| **Evaluator** | Tries to break the implementation | Gemini (adversarial) |

The evaluator doesn't just review code — it's an adversary. It runs the application, probes for failures, tests edge cases, and scores ruthlessly. If any criterion fails (score < 7/10), the sprint goes back to the generator with detailed feedback.

## Installation

### Prerequisites

- **Codex CLI**: `npm install -g @openai/codex`
- **Gemini CLI**: `npm install -g @anthropic-ai/gemini-cli` or `brew install gemini-cli`
- **Git**: For commit tracking

### Install

```bash
# Clone or copy to your skills directory
git clone https://github.com/chatgptkrylor/adversarial-coder.git
cd adversarial-coder

# Or copy to OpenClaw skills
cp -r . ~/.nvm/versions/node/v24.14.0/lib/node_modules/openclaw/skills/adversarial-coder/

# Make executable
chmod +x adversarial-coder
```

## Quick Start

```bash
# Simple prompt
adversarial-coder "Build a CLI tool that searches PubChem by compound name"

# From a prompt file
adversarial-coder --file prompt.md

# With options
TIMEOUT=180 MAX_SPRINTS=3 MAX_RETRIES=2 adversarial-coder "Build a REST API"
```

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `MAX_SPRINTS` | 10 | Maximum number of sprints |
| `MAX_RETRIES` | 3 | Max evaluation retries per sprint |
| `PASS_THRESHOLD` | 7 | Minimum score (out of 10) for each criterion |
| `TIMEOUT` | 300 | Timeout per agent call in seconds |
| `CODEX_MODEL` | gpt-5.4 | Model for Generator |
| `GEMINI_MODEL` | gemini-2.5-flash | Model for Planner/Evaluator |

## The Flow

```
User Prompt (1-4 sentences)
         |
         v
   +-----------+
   |  PLANNER  |  --> writes spec.md (features, sprints, design language)
   +-----------+
         |
         v  (for each sprint)
   +---------------------+
   | CONTRACT NEGOTIATION |  Generator proposes criteria,
   | Generator <-> Eval   |  Evaluator tightens the screws,
   +---------------------+  both lock in "done"
         |
         v
   +-----------+     fail + feedback     +------------+
   | GENERATOR | <---------------------- | EVALUATOR  |
   | (build)   | ----------------------> | (attack)   |
   +-----------+     implementation      +------------+
         |                                      |
         v              pass                    |
    Next Sprint <-------------------------------+
```

## Sprint Contracts (The Innovation)

Before coding, Generator and Evaluator negotiate a **JSON contract**:

```json
{
  "sprintNumber": 1,
  "features": ["Basic output", "Error handling", "Exit codes"],
  "criteria": [
    {
      "name": "exact_output",
      "description": "Script outputs 'Hello World' followed by newline",
      "threshold": 7
    },
    {
      "name": "exit_code",
      "description": "Script exits with code 0 on success",
      "threshold": 7
    }
  ]
}
```

Evaluator uses contract negotiation to **set traps** — adding edge cases, tightening thresholds, demanding specifics.

## Files Created

```
workspace/
├── spec.md                    # Product specification from Planner
├── contracts/
│   └── sprint-{n}.json        # Negotiated sprint contracts
├── feedback/
│   └── sprint-{n}-round-{m}.json  # Evaluator feedback per attempt
├── progress.json              # Current harness state
└── app/                       # Built application
```

## The Scoring System

- **9-10**: Exceptional. Works perfectly, handles edge cases.
- **7-8**: Good. Core functionality works. (PASS threshold)
- **5-6**: Partial. Some functionality, significant gaps.
- **3-4**: Poor. Fundamental issues.
- **1-2**: Failed. Not implemented or broken.

**All criteria must pass (≥7) or sprint fails.**

## Example Run

```bash
$ adversarial-coder "Create hello.py that prints Hello World"

[HARNESS] ADVERSARIAL CODER - Three-Agent Harness
[HARNESS] Planner: Gemini | Generator: Codex | Evaluator: Gemini

[PLANNER] Product specification generated (1 sprints)

[HARNESS] SPRINT 1/1
[HARNESS] Contract agreed: 3 criteria

[GENERATOR] Sprint 1 (attempt 1) - Building...
[GENERATOR] Sprint 1 build complete

[EVALUATOR] Sprint 1: FAILED (1/3 criteria passed)
[EVALUATOR] - basic_functionality: 7 (PASS)
[EVALUATOR] - code_quality: 6 (FAIL - missing shebang)
[EVALUATOR] - error_handling: 6 (FAIL - generic catch)

[HARNESS] Sprint 1 failed attempt 1, retrying...

[GENERATOR] Sprint 1 (attempt 2) - Building...
[GENERATOR] Sprint 1 build complete

[EVALUATOR] Sprint 1: PASSED (3/3 criteria passed)

[HARNESS] All sprints completed successfully!
[HARNESS] Output in: /tmp/adv-test/app/
```

## Architecture

This is inspired by **Generative Adversarial Networks (GANs)**:
- GANs: Generator vs. Discriminator
- This: Generator vs. Evaluator (with Planner added)

The key insight: **separate generation from evaluation, then pit them against each other.**

From Anthropic's engineering article:
> "Every component in a harness encodes an assumption about what the model can't do on its own."

As models improve, harnesses simplify. But the **pattern** of separating roles is durable.

## Based On

This implementation is based on the [adversarial-dev](https://github.com/chatgptkrylor/adversarial-dev) architecture, adapted for OpenClaw with:
- Codex (instead of both Claude/Codex)
- Gemini (instead of both for Planner/Evaluator)
- Bash script instead of TypeScript
- Simplified contract negotiation
- Timeout handling for reliability

## License

MIT