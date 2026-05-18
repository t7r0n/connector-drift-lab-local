# Connector Drift Lab Local

Connector reliability is won when schema drift is detected before sync success becomes modeled-data failure.

The implementation is a laptop-scale proof of the workflow behind that claim, with `connector` fixtures and falsifiable gates.

## Intent

Offline connector schema-drift, MAR-cost, and downstream-breakage simulator.

## What the code proves

- Compiles 200 replayable `connector` fixtures that make the `connectorforge` assumptions observable.
- Treats `drift_detection_latency`, `downstream_breakage_avoided`, `mar_cost_forecast`, and `quarantine_precision` as release gates, not dashboard decoration.
- Plants degraded cases for `silent_type_change`, `metadata_query_spike`, `api_deprecation`, and `mar_cost_surge` and checks whether the harness catches them.
- Exports the `Connector Drift Lab Local` run as structured reports, static HTML, benchmark numbers, and a shareable package.

## Local run

```bash
uv sync --extra dev
uv run connector-drift init-demo --force
uv run connector-drift run-suite
uv run connector-drift verify
uv run connector-drift dashboard
uv run connector-drift benchmark --iterations 100
uv run connector-drift export-demo-pack
```

## Produced files

- `data/scenarios.json`
- `outputs/summary.json`
- `outputs/reports.json`
- `outputs/evidence_pack.md`
- `outputs/dashboard.html`
- `outputs/benchmark.json`
- `outputs/demo-pack.zip`

## Gatekeeping

```bash
uv run ruff check .
uv run pytest -q
uv run connector-drift run-suite
uv run connector-drift verify
uv run connector-drift benchmark --iterations 100
```

## Operational boundary

The `connector-drift-lab-local` public surface is source, tests, lockfile, and docs. It does not need credentials, browser state, customer records, or hosted services.
