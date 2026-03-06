---
name: skill-forge-eval
description: >
  Run evaluation pipelines on Claude Code skills to test triggering accuracy,
  workflow correctness, and output quality. Spawns executor, grader, comparator,
  and analyzer sub-agents for parallel evaluation. Generates eval_metadata.json,
  grading.json, and feedback reports. Use when user says "eval skill", "test skill",
  "run evals", "evaluate skill", "skill evals", "test skill quality",
  "run skill tests", or "skill evaluation".
---

# Skill Evaluation Pipeline

Run structured evaluations against Claude Code skills to verify triggering,
correctness, and quality using a multi-agent pipeline.

## Process

### Step 1: Define Eval Set

Accept eval definitions from:
- **Path to eval set JSON**: `evals/evals.json` or user-specified file
- **Inline prompts**: User provides eval queries directly
- **Auto-generated**: Generate from skill description (see Step 1b)

**Eval set JSON schema:**
```json
{
  "skill_name": "my-skill",
  "skill_path": "./my-skill",
  "evals": [
    {
      "eval_id": 0,
      "eval_name": "descriptive-name",
      "prompt": "The user's task prompt",
      "input_files": [],
      "assertions": [
        {
          "name": "output-has-score",
          "check": "Output contains a numeric score between 0-100",
          "weight": 1.0
        }
      ],
      "should_trigger": true
    }
  ]
}
```

#### Step 1b: Auto-Generate Eval Set

If no eval set exists, generate one:
1. Read the skill's SKILL.md description and instructions
2. Run `python scripts/generate_eval_set.py <skill-path>` to produce a starter set
3. Present the generated set to the user for review and editing
4. User approves or modifies before proceeding

### Step 2: Set Up Workspace

Create an iteration workspace:
```
eval-workspace/
  iteration-1/
    eval-0/
      with_skill/
        outputs/
      baseline/
        outputs/
    eval-1/
      with_skill/
        outputs/
      baseline/
        outputs/
    benchmark.json
    benchmark.md
```

### Step 3: Execute Eval Runs

For each eval in the set, spawn two parallel runs:

**With-skill run** (delegate to `agents/skill-forge-executor.md`):
```
Execute this task:
- Skill path: <path-to-skill>
- Task: <eval prompt>
- Input files: <eval files if any, or "none">
- Save outputs to: <workspace>/iteration-<N>/eval-<ID>/with_skill/outputs/
- Outputs to save: <what the assertions check>
```

**Baseline run** (delegate to `agents/skill-forge-executor.md`):
- For new skills: run without the skill loaded
- For improved skills: run with the previous version (snapshot it first)

Save timing data to `timing.json` in each run directory:
```json
{
  "total_tokens": 84852,
  "duration_ms": 23332,
  "total_duration_seconds": 23.3
}
```

### Step 4: Grade Results

Delegate to `agents/skill-forge-grader.md` for each completed run:

1. Grade against assertions defined in eval_metadata.json
2. Save results to `grading.json` per run:
```json
{
  "eval_id": 0,
  "run_type": "with_skill",
  "assertions": [
    {
      "name": "output-has-score",
      "passed": true,
      "evidence": "Found score: 87/100 on line 14"
    }
  ],
  "pass_rate": 1.0
}
```

### Step 5: Aggregate and Analyze

1. Run `python scripts/aggregate_benchmark.py <workspace>/iteration-<N> --skill-name <name>`
2. This produces `benchmark.json` and `benchmark.md` with:
   - Pass rate per eval (with_skill vs baseline)
   - Average time and token usage
   - Improvement ratio (with_skill / baseline)

3. Delegate to `agents/skill-forge-analyzer.md` to:
   - Surface patterns that aggregate stats might hide
   - Identify consistently failing assertion types
   - Flag regressions from previous iterations

### Step 6: Present Results

Generate a summary report:

```markdown
# Eval Report: [skill-name] — Iteration [N]

## Overall
| Metric | With Skill | Baseline | Delta |
|--------|-----------|----------|-------|
| Pass Rate | X% | Y% | +Z% |
| Avg Time | Xs | Ys | -Zs |
| Avg Tokens | X | Y | -Z |

## Per-Eval Results
| Eval | With Skill | Baseline | Status |
|------|-----------|----------|--------|
| eval-0 | PASS | FAIL | Improved |
| eval-1 | PASS | PASS | Maintained |

## Patterns & Insights
[From analyzer agent]

## Recommendations
[Specific improvements based on failures]
```

### Step 7: Collect Feedback

Save user feedback to `feedback.json`:
```json
{
  "reviews": [
    {
      "run_id": "eval-0-with_skill",
      "feedback": "the chart is missing axis labels",
      "timestamp": "2026-03-06T12:00:00Z"
    }
  ],
  "status": "complete"
}
```

Pass feedback to `/skill-forge evolve` for the next iteration.

## Advanced: Blind Comparison

For rigorous A/B testing between skill versions:
1. Delegate to `agents/skill-forge-comparator.md`
2. Comparator sees outputs from both versions without labels
3. Rates each output on assertion criteria
4. Returns preference scores without knowing which is "new" vs "old"

## Quality Gates

Before marking an eval run as complete:
- [ ] All evals executed (with_skill + baseline)
- [ ] Timing data captured for every run
- [ ] All assertions graded with evidence
- [ ] Benchmark aggregated with pass rate, time, tokens
- [ ] Analyzer patterns documented
- [ ] Results presented to user
