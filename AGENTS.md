# AGENTS.md — Dataset Pipeline

Repository of AI agent prompts and skills for a sequential 10-stage data pipeline.

## Pipeline architecture

The **orchestrator** (`agents/rules/dataset-pipeline-orchestrator.mdc`) delegates each stage to a sub-agent. The 10 stages run in strict order:

```
1. load → 2. schema → 3. explore → 4. nulls → 5. dedupe → 6. types → 7. categories → 8. validate-ranges → 9. quality → 10. export
```

## Skills (read before coding)

| Domain | Path |
|--------|------|
| I/O | `agents/skills/dataset-pipeline-io/SKILL.md` |
| Schema | `agents/skills/dataset-pipeline-schema/SKILL.md` |
| Profiling | `agents/skills/dataset-pipeline-profile/SKILL.md` |
| Cleaning (nulls, dedup) | `agents/skills/dataset-pipeline-clean/SKILL.md` |
| Types & categories | `agents/skills/dataset-pipeline-types-categories/SKILL.md` |
| Validation & quality | `agents/skills/dataset-pipeline-validate-quality/SKILL.md` |

## Sub-agents (one file per stage)

Each lives in `agents/agents/` with `name: dataset-pipeline-<stage>`.

## Key conventions

- Internal artifact format: **parquet**
- Output format: **User choice** (CSV or Delta Lake)
- Env vars: `DATA_INPUT_URI`, `DATA_OUTPUT_URI`; optional `DATABASE_URL`, `API_KEY`
- Fail-fast by default (stage 9 quality check can allow degraded export if user requests)
- Maintain `run_id`, `manifest.json` (URIs, schema_hash, policies), `metrics.json` per stage
- `manifest.execution_profile=chunked` triggers chunked read/write
- `manifest.exploration.depth` limits profiling cost

## Tool selection

**IMPORTANT:** Before starting the pipeline, ALWAYS ask the user to choose the analysis tool:

1. **pandas** - For datasets < 1M rows or when simplicity is preferred
2. **PySpark** - For datasets >= 1M rows or distributed processing needs

Present the options and let the user decide. Default to `pandas` if no preference is specified.

## Export format

**IMPORTANT:** Before exporting, ALWAYS ask the user to choose the output format:

1. **CSV** - For simple compatibility and easy opening in spreadsheets
2. **Delta Lake** - For ACID transactions, time travel, and scalable data lakes (requires `deltalake` Python library)

Present the options and let the user decide. Default to `CSV` if no preference is specified.

## Path note

The orchestrator references `.cursor/skills/...` but actual skill files are under `agents/skills/`. Adjust paths when invoking.
