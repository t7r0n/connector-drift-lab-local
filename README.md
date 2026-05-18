# Connector Drift Lab Local

Offline connector schema-drift, MAR-cost, and downstream-breakage simulator.

This is a local-first, synthetic-data prototype inspired by a company-specific project plan for **Fivetran**. It is built to demonstrate the engineering shape of `connectorforge` without private data, credentials, external APIs, or hosted services.

## Why it matters

Connector reliability is won when schema drift is detected before sync success becomes modeled-data failure.

## What it does

- Generates deterministic synthetic `connector` scenarios.
- Scores each scenario against domain-specific quality gates.
- Produces evidence-backed findings for realistic failure modes.
- Writes a static dashboard, JSON reports, benchmark output, and a portable demo pack.
- Exposes a JSONL tool loop for local agent integration.

## Metrics

- `drift_detection_latency`
- `downstream_breakage_avoided`
- `mar_cost_forecast`
- `quarantine_precision`

## Failure modes

- `silent_type_change`
- `metadata_query_spike`
- `api_deprecation`
- `mar_cost_surge`

## Quickstart

```bash
uv sync --extra dev
uv run connector-drift init-demo --force
uv run connector-drift run-suite
uv run connector-drift verify
uv run connector-drift dashboard
uv run connector-drift benchmark --iterations 100
uv run connector-drift export-demo-pack
```

## Expected outputs

- `data/scenarios.json`
- `outputs/summary.json`
- `outputs/reports.json`
- `outputs/evidence_pack.md`
- `outputs/dashboard.html`
- `outputs/benchmark.json`
- `outputs/demo-pack.zip`

## Validation

```bash
uv run ruff check .
uv run pytest -q
uv run connector-drift run-suite
uv run connector-drift verify
uv run connector-drift benchmark --iterations 100
```

## Demo hook

A drift preview shows which dbt models and dashboards would break before the connector sync runs.
