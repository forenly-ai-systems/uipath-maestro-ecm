# Documentation

Curated index of project documentation.

| Doc | What you get from it |
|---|---|
| [`ARCHITECTURE.md`](./ARCHITECTURE.md) | UiPath-core BPMN architecture: how the 8 containers translate to Maestro subprocesses, how vendor task nodes attach, and the swap-ability matrix. |

See also [`../samples/triggers/`](../samples/triggers/) for the canonical `IncidentReport` schema and source samples.

## How to keep these current

These documents are the project's spec, not marketing material. When the architecture or data model changes:

- Update the relevant doc in the same PR that makes the change real.
- Use commit message prefixes: `docs(architecture)`, `docs(readme)`, `samples(triggers)`.

## What is intentionally not in `docs/`

- **Sample data** lives in [`samples/`](../samples/) so it sits alongside the schema it conforms to.
- **BPMN diagrams, agent definitions, API Workflow configs** live under `artifacts/` once implementation starts — machine-readable artifacts, not narrative docs.
- **Credentials, `.env` files, secrets** are never committed; see [`../CONTRIBUTING.md`](../CONTRIBUTING.md).
