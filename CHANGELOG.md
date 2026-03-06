# Changelog

## v1.1.0 — Eval Pipeline & Benchmarking

### Features

- **Eval** — Run evaluation pipelines on skills with assertions, grading, and multi-agent execution (executor, grader, comparator, analyzer agents)
- **Benchmark** — Measure skill performance with variance analysis, multiple trials, threshold gating, and iteration comparison
- **Description Optimization** — Automated train/test split scoring to optimize skill descriptions for trigger accuracy
- **Eval Set Generation** — Auto-generate trigger eval sets from SKILL.md descriptions
- **Blind A/B Comparison** — Unbiased comparison between skill versions via comparator agent
- **Enhanced Review** — Review sub-skill now generates trigger eval sets and runs description optimization
- **Enhanced Evolve** — Evolve sub-skill now includes iteration workspace protocol and description optimization loop
- 4 new agents: `skill-forge-executor`, `skill-forge-grader`, `skill-forge-analyzer`, `skill-forge-comparator`
- 3 new scripts: `generate_eval_set.py`, `aggregate_benchmark.py`, `optimize_description.py`
- 2 new sub-skills: `skill-forge-eval`, `skill-forge-benchmark`

## v1.0.0 — Initial Release

### Features

- **Skill Forge orchestrator** — Tier 4 skill with routing table, 6 sub-skills, 4 agents, 4 scripts
- **Plan** — Architecture design with complexity tier detection (1-4), sub-skill decomposition, and file structure planning
- **Build** — Full skill scaffolding with SKILL.md generation, frontmatter writing, sub-skills, scripts, references, and agents
- **Review** — Quality auditing with 0-100 health score across 6 categories (structure, frontmatter, description, body, scripts, agents)
- **Evolve** — Skill improvement based on feedback, triggering issues, and testing results
- **Publish** — Packaging as `.skill` ZIP files with install script generation
- **Convert** — Multi-platform conversion to OpenAI Codex, Google Gemini CLI, Google Antigravity, and Cursor
- **10 reference files** — Comprehensive knowledge base covering anatomy, patterns, frontmatter spec, description guide, testing, 3-layer architecture, tools, hooks, activation, and platform specs
- **4 execution scripts** — `init_skill.py` (scaffold), `validate_skill.py` (validate), `package_skill.py` (package), `convert_skill.py` (convert)
- **4 agent definitions** — Architect, Writer, Validator, and Converter subagents for parallel delegation
- **4 skill templates** — Tier 1 (minimal), Tier 2 (workflow), Tier 3 (multi-skill), Tier 4 (ecosystem)
- **Progressive disclosure** — 3-level loading (frontmatter always, instructions on activation, references on demand)
- **Agent Skills standard** — Full compliance with the open standard at agentskills.io
