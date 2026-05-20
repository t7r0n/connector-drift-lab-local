# Research And Plan Review

Project: `connectorforge`

## Refined Thesis

Connector reliability is won when schema drift is detected before sync success becomes modeled-data failure.

The implementation is intentionally local and synthetic, but the test harness is shaped around the real operating question: can the proposed artifact create evidence a founder, CTO, or product leader would immediately recognize as useful?

## Local Evidence Basis

- Synthetic connector traces.
- Deterministic schema-drift and pagination fixtures.
- Local replay reports for downstream-breakage prediction.

## Plan Excerpt Used

## The Gap

Long-tail SaaS connectors are brittle when schema inference, cursor selection, checkpoint cadence, and pagination handoff are authored without a closed-loop replay harness. A connector can appear to sync successfully while silently dropping rows, widening costs, or breaking modeled data after a schema drift. The missing artifact is a local proof harness that predicts those failures before a connector reaches production.

## The Project — `connectorforge`

> A connector-drift simulator that turns recorded source traces into schema-drift, replay-determinism, and downstream-breakage evidence.

**What it is.** A Python CLI and small web UI that ingests deterministic connector traces and emits a report proving correctness across four properties: incremental-sync idempotence, schema-drift survival, pagination completeness, and checkpoint resumption.

**Why it solves the gap.** ConnectorForge turns missing primary keys, mis-sized checkpoint cadence, and pagination-cursor handoff into property-based tests that run against recorded traces before deploy.

**The wow moment.** A recorded source trace is replayed through synthetic schema changes, and the report shows exactly which downstream tables, models, and dashboards would break before the sync ships.

## Prototype Plan (the shippable demo)

**Surface:**
```bash
pipx install connectorforge
cforge new --from-openapi specs/notion.yaml --target notion
cforge bench connectors/notion --cassette tests/cassettes/notion.yaml
cforge report connectors/notion
```
And a tiny browser UI at `cforge serve` that renders the report card.

**Five demo inputs:**
1. **Notion API** — cursor pagination, nested blocks, real schema drift across workspaces.
2. **Linear GraphQL** — non-REST, tests the synthesizer's GraphQL adapter.
3. **CRM export** — side-by-side row-fidelity replay against a canonical local fixture.
4. **A bespoke internal REST API** (we'll mock one) — proves the long-tail story.
5. **Stripe** — large schema, tests checkpoint cadence sizing.

**Expected output:** a green 4/4 report card with measurable numbers — synthesis under 60s, replay sync under 90s for 50k rows, zero row-count drift across schema-evolution events.

**Proof metrics:** synthesis latency p50/p95, percentage of property-test failures caught pre-deploy, row-fidelity versus canonical fixtures, and bytes of human-written connector code after synthesis.


## Build Acceptance Criteria

- Deterministic local fixtures.
- Domain-specific metrics and failure modes.
- Passing unit tests.
- Passing CLI verifier.
- Static dashboard generated locally.
- Benchmark output under the project `outputs/` folder.
- Public-safe README: no founder emails, no private outreach text, no credentials.
