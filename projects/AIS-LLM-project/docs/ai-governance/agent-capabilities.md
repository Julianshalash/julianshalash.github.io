# AIS Agent Capabilities and Permissions

## 1. Purpose

This document defines the AI capabilities and actions visible in the AIS source implementation, distinguishing model-generated behavior from backend-enforced operations.

## 2. Agent Capabilities

AIS can answer sustainability questions, route intent, use relevant conversation/document context, search uploaded documents, retrieve local knowledge-base passages, conditionally search the web, run deterministic calculators, review documents, and stream responses. Document parsing and OCR occur in the backend upload pipeline, not as a model-selected tool.

## 3. Available Tools

### Registered chat tools

| Tool | Purpose / input | Output and data accessed | Permission / external service | Timeout, retry, logging, failure |
|---|---|---|---|---|
| `find_in_documents` | Search up to 15 supplied terms against document text already attached to the current chat context. | JSON hits with filename, page when available, and snippets; reads the in-memory per-turn document store. | Read-only, limited to context passed to that invocation; no external service. | Invoked through common tool runner with a 15-second timeout; no tool-specific retry. Errors are logged and returned as a tool error. |
| `leed_bicycle_storage` | Occupants, visitors, floor area, unit/use, dwelling units. | Required bicycle storage values and calculation steps. | Deterministic calculation; no external service. | Common 15-second timeout; invalid inputs raise tool errors. |
| `mostadam_bicycle_racks` | Occupants and floor area. | Long-/short-term rack calculation. | Deterministic calculation; no external service. | Common 15-second timeout; invalid inputs raise tool errors. |
| `shower_facilities` | Occupant count. | Shower-facility count and steps. | Deterministic calculation; no external service. | Common 15-second timeout. |
| `parking_requirements` | Total parking spaces. | Preferred parking and fueling station counts. | Deterministic calculation; no external service. | Common 15-second timeout. |
| `open_space_requirements` | Site area. | Required and vegetated open space. | Deterministic calculation; no external service. | Common 15-second timeout. |
| `outdoor_space_size` | Occupancy and optional dwelling occupants. | Outdoor-space quantities. | Deterministic calculation; no external service. | Common 15-second timeout. |
| `ratio_percentage` | Numerator, denominator, label. | Percentage and calculation. | Deterministic calculation; no external service. | Common 15-second timeout. |
| `energy_improvement` | Baseline/proposed annual energy. | Percentage improvement. | Deterministic calculation; no external service. | Common 15-second timeout. |
| `occupant_density` | Occupancy and area. | Occupants per square metre. | Deterministic calculation; no external service. | Common 15-second timeout. |
| `u_value_assembly` | Material layers and optional surface films. | Assembly U-value and calculation steps. | Deterministic calculation; no external service. | Common 15-second timeout. |
| `u_from_r` | R-value. | Converted U-value. | Deterministic calculation; no external service. | Common 15-second timeout. |
| `weighted_u_value` | Component areas and U-values. | Area-weighted U-value. | Deterministic calculation; no external service. | Common 15-second timeout. |
| `outdoor_air_flushout` | Floor area and unit. | Flush-out air quantity. | Deterministic calculation; no external service. | Common 15-second timeout. |
| `horton_infiltration` | Horton equation parameters and time. | Infiltration result. | Deterministic calculation; no external service. | Common 15-second timeout. |
| `runoff_rate` | Rainfall, depression storage, infiltration. | Runoff result. | Deterministic calculation; no external service. | Common 15-second timeout. |
| `energy_use_intensity` | Annual energy and floor area. | EUI result. | Deterministic calculation; no external service. | Common 15-second timeout. |
| `operational_carbon` | Energy and emissions factor. | Operational carbon estimate. | Deterministic calculation; no external service. | Common 15-second timeout. |
| `simple_payback` | Extra capital cost and annual savings. | Payback years. | Deterministic calculation; no external service. | Common 15-second timeout. |

All registered calculators validate numeric constraints in code and return result, unit, steps and a verification note. A shared tool runner applies the 15-second timeout and logs failures; automatic retry behavior is not implemented in that runner.

### Other bounded integrations

| Integration | Trigger and scope | Service and data | Failure/logging |
|---|---|---|---|
| Local knowledge retrieval | Evidence gathering for a routed request. | SQLite FTS5 local passages and source metadata. | Search errors/fallback behavior are handled in retrieval code; evidence counts/types are logged. |
| Tavily web search | `AIS_WEB_MODE` policy and request need/explicit ask. | Routed query sent to Tavily; results are filtered and included as evidence. | 25-second orchestration timeout; errors logged and treated as missing live evidence. |
| Legacy ContactOut search | Explicit `bd:` message prefix and handler availability. | Professional-search parameters sent to ContactOut; separate from registered calculator tools. | Integration has legacy module-level state; consult security review before enabling. |

## 4. Allowed Actions

The active AI flow may generate text, query the local knowledge base, search document text supplied for the turn, invoke registered calculators, and request optional web search through the evidence layer. These integrations are intended for read/retrieval/calculation, with tool calls bounded by a maximum number of rounds and timeout.

## 5. Prohibited Actions

No model-callable shell, arbitrary code execution, arbitrary filesystem browsing, payment execution, email sending, or general write-to-third-party API tool was identified in the active registered tool set. This is a statement about the reviewed code, not a guarantee about external infrastructure or future changes.

## 6. Human Approval

No explicit human approval step is wired into registered calculator, document-search, or web-search invocation. No consequential external write tool was identified. Human review remains necessary for engineering or regulatory decisions; the assistant should not be treated as an approval authority.

## 7. Loop, Context, and Failure Controls

- Tool iterations default to a maximum of four rounds (`AIS_MAX_TOOL_ROUNDS`).
- Each individual tool call has a 15-second timeout.
- The LLM clients configure timeout/retry behavior; final behavior after repeated provider failure depends on the calling route.
- The model is asked to select tools, but backend registered tools and their input schemas define executable capabilities.
- Uploaded documents and retrieved passages are labelled untrusted in prompts; this does not eliminate prompt injection.
- When the tool budget is exhausted, a final answer is requested without additional tools.

## 8. Known Limitations

The model can produce incorrect answers, cite sources that do not support a claim, misread documents/OCR, choose a wrong framework version, or be influenced by untrusted content. Calculator outputs depend on correct user-provided inputs and assumptions. Users should verify important results against authoritative guidance and qualified professionals.

