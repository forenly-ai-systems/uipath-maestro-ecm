# uipath-maestro-ecm — Agentic Engineering Change Management on UiPath Maestro

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](./LICENSE)

> An open-source framework for running **Engineering Change Management (ECM)** as an
> **agentic, human-in-the-loop workflow** on **UiPath Maestro BPMN**.

Engineering changes in regulated industries (aerospace, medical, automotive, pharma) are detected
in seconds but resolved in days. This project orchestrates the full ECM lifecycle — from the moment
a defect or deviation is detected to the audited release — as a BPMN process where AI agents do the
pre-analysis and humans approve at the decisions that matter.

## What it does

- **8 process containers** modelling the ECM lifecycle (problem report → change analysis → implementation → quality control) as Maestro BPMN subprocesses.
- **9 roles** as BPMN swimlanes — some realized as LLM agents (e.g. the Analyst), others as human review boards via Action Center HITL.
- **An agentic "Fast Track" gateway** — a Risk Agent scores each change request and routes low-risk, reversible changes around the full review board automatically.
- **Vendor-agnostic triggers** — vision/AOI inspection, ERP events, a manual portal, or supplier deviation feeds all normalize to one canonical `IncidentReport` shape.
- **Audit-grade traceability** — immutable log entries at every gateway and task.

## Architecture at a glance

```
                ╔══════════════════════════════════════════╗
                ║   UiPath Maestro BPMN (core)             ║
                ║   • 8 ECM containers as subprocesses      ║
                ║   • 9 roles as swimlanes / agents         ║
                ║   • Action Center HITL (board approvals)  ║
                ║   • Agentic Fast Track gateway            ║
                ║   • Immutable audit log                   ║
                ╚══════════════════════════════════════════╝
                          ↑ task nodes invoke ↓
   ┌──────────────┬──────────────┬──────────────┬──────────────┐
   │ Trigger      │ LLM Agents   │ PLM /        │ Supplier     │
   │ source       │ (swappable)  │ Enterprise   │ Systems      │
   │ (swappable)  │              │ (swappable)  │ (swappable)  │
   ├──────────────┼──────────────┼──────────────┼──────────────┤
   │ AOI vision   │ Claude /     │ PLM mock     │ Supplier     │
   │ ERP event    │ Gemini via   │ (Aras /      │ ERP / PLM    │
   │ Manual form  │ LangChain    │ Windchill-   │ via API or   │
   │ Deviation    │              │ shaped)      │ RPA Robot    │
   └──────────────┴──────────────┴──────────────┴──────────────┘
```

Full architecture: [`docs/ARCHITECTURE.md`](./docs/ARCHITECTURE.md).

## The 8 ECM containers

```
Process 1 · Change Trigger
  1. Problem Report        Requestor → Analyst agent → IRB HITL → disposition (5-way)
  2. Customer Requirement  Customer-raised path + IRB feedback loop

Process 2 · Change Solution
  3. Change Analysis       ECR review → Risk Agent → Fast Track OR CRB HITL
  4. Supplier Business     CRB analyzes cost / hours / lead time / effectivity
     Analysis

Process 3 · Change Implementation
  5. Plan Change Implement Analyst + Change Specialist draft ECN → CIB HITL
  6. Release Change        Product data released; downstream systems notified
  7. Supplier Change       RPA Robot / API Workflows write to supplier systems
     Implementation
  8. Quality Control       Audit-summary agent + immutable log + signature
```

## How it's built

| Layer | Choice |
|---|---|
| **Orchestration** | UiPath Automation Cloud + Maestro BPMN |
| **UiPath components** | Agent Builder · Maestro BPMN · API Workflows · Action Center · RPA Robot |
| **External agent framework** | LangChain (Analyst agent, Risk Agent, audit-summary agent) |
| **LLM** | Claude (primary) + Gemini (fallback) |
| **Trigger demo source** | AOI (automated optical inspection) vision system; interchangeable |
| **PLM target (mocked)** | Open-source PLM-style mock service (Aras Innovator–shaped API) |
| **Supplier targets (mocked)** | Two mock endpoints — one API-driven, one RPA-driven |

## Getting started

The orchestration layer runs on **UiPath Automation Cloud** (Maestro BPMN is the mandatory core).
External frameworks (LangChain, custom Python agents) are welcome **as task nodes invoked from the BPMN flow**.

You can explore and validate the canonical data model locally without any UiPath access:

```bash
python -m venv .venv && source .venv/bin/activate
pip install jsonschema
python - <<'PY'
import json, glob
from jsonschema import Draft202012Validator
schema = json.load(open('samples/triggers/incident_report.schema.json'))
v = Draft202012Validator(schema)
for f in glob.glob('samples/triggers/*.json'):
    if 'schema' in f: continue
    v.validate(json.load(open(f)))
    print('OK', f)
PY
```

See [`samples/triggers/`](./samples/triggers/) for the `IncidentReport` schema + 4 source samples (vision / ERP / manual / supplier) that let you replay the flow without external systems.

## Contributing

Contributions are welcome — see [`CONTRIBUTING.md`](./CONTRIBUTING.md). Day-to-day discussion happens on the project **Discord**.

## License

[Apache License 2.0](./LICENSE)
