# Research And Plan Review

Company: Fivetran
Project: `connectorforge`

## Refined Thesis

Connector reliability is won when schema drift is detected before sync success becomes modeled-data failure.

The implementation is intentionally local and synthetic, but the test harness is shaped around the real operating question: can the proposed artifact create evidence a founder, CTO, or product leader would immediately recognize as useful?

## Fresh Sources Checked

- https://www.fivetran.com/learn/data-connectors
- https://www.fivetran.com/press/data-pipeline-failures-cost-enterprises-3-million-per-month-fivetran-benchmark-finds
- https://fivetran.com/docs/using-fivetran/fivetran-dashboard/connectors/schema

## Plan Excerpt Used

## The Gap

Fivetran's bet is that **every customer's long-tail SaaS becomes a Connector SDK connector authored by an agent**. But the current authoring path is brittle: a developer reads an OpenAPI/REST spec, hand-writes the `schema()` function, picks a cursor column, decides when to `checkpoint()`, and prays the first 10k-record sync doesn't hit pagination edge cases — issues the Fivetran docs warn about ([state mgmt blog](https://www.fivetran.com/blog/data-pipeline-state-management-an-underappreciated-challenge)). The freshly published `fivetran_csdk_tools` repo (May 17, 2026) is a first stab at agentic authoring but, as of this writing, it ships as agent-side helpers — not as a *closed-loop synthesizer* that proves a connector is correct before a customer wires it into production. There is no public benchmark, no fuzz-replay harness for state transitions, no semantic diff that catches the classic "I shipped a connector that silently drops rows on schema drift" failure mode that the docs themselves call out. **That gap is the single biggest threat to the connector long-tail flywheel** — and to the dbt-Labs-merger thesis that depends on customers self-serving novel sources into the unified platform.

## The Project — `connectorforge`

> An agentic connector synthesizer that turns an OpenAPI spec + 3 example API calls into a working `fivetran_connector_sdk` connector, then *proves* it: schema-drift fuzzer, replay-determinism harness, and a `cforge bench` score Fivetran can publish on the SDK README.

**What it is.** A Python CLI + small web UI built on top of `fivetran_connector_sdk` and `fivetran_csdk_tools`. Input: an OpenAPI 3.x spec (or a Postman collection), API credentials, and the customer's target schema hint. Output: a deploy-ready connector directory matching the `connectors/<name>/{connector.py, requirements.txt, configuration.json}` layout the SDK enforces ([template](https://github.com/fivetran/fivetran_connector_sdk/tree/main/template_connector)), *plus* a `cforge_report.json` proving correctness across four properties: incremental-sync idempotence, schema-drift survival, pagination completeness, checkpoint-resumption.

**Why it solves the gap.** The current SDK puts the human on the hook for the three failure modes Fivetran's own docs flag: missing primary keys causing duplicate `_fivetran_id` rows on schema drift, mis-sized checkpoint cadence, and pagination-cursor handoff. ConnectorForge turns each into a property-based test that runs against a recorded API trace before deploy. It is *the missing CI step* between `fivetran_csdk_tools` (authoring) and `fivetran-mcp` (operating).

**The wow moment.** Demo: I paste a Notion API OpenAPI spec into `cforge new --from-openapi notion.yaml`. Sixty seconds later there's a working connector in `connectors/notion/` and a green report card showing 4/4 properties pass on a 50k-record replay, with a one-line `fivetran deploy` ready to run. Fraser sees a connector authored, fuzzed, and benched — by a Python repo that respects his "AI should assist developers" line — faster than his eng team's onboarding deck reads.

## Prototype Plan (the shippable demo)

**Surface:**
```bash
pipx install connectorforge
cforge new --from-openapi specs/notion.yaml --target notion
cforge bench connectors/notion --cassette tests/cassettes/notion.yaml
cforge deploy connectors/notion  # wraps `fivetran` CLI
```
And a tiny browser UI at `cforge serve` that renders the report card.

**Five demo inputs (deliberately representative of long-tail Fivetran customers):**
1. **Notion API** — cursor pagination, nested blocks, real schema drift across workspaces.
2. **Linear GraphQL** — non-REST, tests the synthesizer's GraphQL adapter.
3. **HubSpot CRM** — known Fivetran connector — we run side-by-side and show our synthesis matches row-for-row.
4. **A bespoke internal REST API** (we'll mock one) — proves the long-tail story.
5. **Stripe** — large schema, tests checkpoint cadence sizing.

**Expected output:** a green 4/4 report card with measurable numbers — synthesis under 60s, replay sync under 90s for 50k rows, zero row-count drift across schema-evolution events.

**Proof metrics:** (a) synthesis latency p50/p95, (b) % of property-test failures caught pre-deploy vs. the Fivetran example connectors, (c) row-fidelity vs. the canonical HubSpot Fivetran connector, (d) bytes of human-written Python per connector (target: <40 LOC after synthesis).


## Build Acceptance Criteria

- Deterministic local fixtures.
- Domain-specific metrics and failure modes.
- Passing unit tests.
- Passing CLI verifier.
- Static dashboard generated locally.
- Benchmark output under the project `outputs/` folder.
- Public-safe README: no founder emails, no private outreach text, no credentials.
