# Trigger samples

These files demonstrate how every supported trigger source produces the same canonical
[`IncidentReport`](./incident_report.schema.json) consumed by **Container 1 — Problem Report**.
The flow can be replayed end-to-end without external systems by feeding these files into the trigger adapters.

## Files

| File | Source type | What it simulates |
|---|---|---|
| [`vision_aoi_defect.json`](./vision_aoi_defect.json) | `vision` | An AOI vision detector flags a misrouted cable on an aerospace control panel — AS9100 compliance class, evidence images attached |
| [`erp_deviation.json`](./erp_deviation.json) | `erp` | An ERP (SAP-shaped) raises an incoming-inspection material deviation: 14 units of 200 fail dimensional tolerance — IATF 16949 compliance class |
| [`manual_qa_report.json`](./manual_qa_report.json) | `manual` | An operator reports an intermittent label misprint via the QA web portal — ISO 13485 compliance class, low initial severity |
| [`supplier_deviation.json`](./supplier_deviation.json) | `supplier` | A supplier (ACME) requests a part substitution due to upstream discontinuation — AS9100, supplier-EDI-originated |

## How adapters use these

Each trigger source has its own raw payload format. The trigger adapter for that source is responsible for:

1. Validating the raw payload (vendor-specific shape).
2. Mapping it to the canonical `IncidentReport` shape (defined in `incident_report.schema.json`).
3. Preserving the raw payload under the `vendor.*` field so audit can reproduce the trigger.

The BPMN never sees the vendor-specific shape; it only sees the canonical object. This makes the
framework genuinely vendor-agnostic — adding a new trigger source means writing a new adapter, not
modifying the BPMN.

## How the Risk Agent reads these

The agentic Fast Track gateway in Container 3 reads `source.detectorConfidence`, `severity`,
`complianceClass`, and `affectedItems[].supplierId` to score each ECR. High-compliance-class items
(e.g. `as9100`, `iso13485`, `gmp`) bias toward Full Track (CRB HITL). Low-confidence detections
(`< 0.7`) bias toward HITL regardless of severity.

## Adding new samples

When a new trigger source is integrated (e.g. another vision system, AR-glasses, or a supplier EDI
feed), add a new sample here using the same canonical shape. The vendor-specific raw payload goes
under `vendor.<provider>`.
