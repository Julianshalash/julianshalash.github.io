Purpose: State the emissions accounting concepts and calculation behavior supported by current code without overstating methodological coverage.

Status: In active development. Scope 3 and commercial features are not yet implemented.

## GHG Protocol scopes

The GHG Protocol Corporate Standard groups a company inventory into Scope 1 (direct emissions from owned/controlled sources), Scope 2 (indirect emissions from purchased energy), and Scope 3 (other value-chain indirect emissions). The implementation currently supports Scope 1 and 2 emissions data paths only. This conceptual summary is not a claim of conformity, assurance or verification.

## Implemented Scope 1 and Scope 2 functionality

| Area | Status | Verified behavior |
|---|---|---|
| Scope 1 data ingestion | ✅ Implemented | The system accepts configured Scope 1 workbooks/categories; process-emissions upload validates the workbook and extracts activity and total emission values from workbook summary/input cells. |
| Scope 1 category totals | ✅ Implemented | Activity/result models persist CO2, CH4, N2O and CO2e totals and optional factor/calculation provenance. Process upload stores extracted CO2e; it does not independently recalculate each gas. |
| Scope 2 data and reporting | ✅ Implemented | Scope 2 category/template data and separate location-based and market-based report inputs exist. |
| Generic Scope 1/2 calculation engine | 🚧 In progress | The source shows persisted results and template-specific parsing, but no single, complete shared calculation engine or factor catalog provenance rules could be verified. |

### Formula verified in report builder

For electricity where both activity and factor are provided:

```text
Scope 2 tCO2e = electricity activity (kWh) × emission factor (kgCO2e/kWh) ÷ 1,000
```

The report builder computes this independently for location-based and market-based factors. It also reports supplied Scope totals; it does not establish that these inputs are derived from all imported activity rows.

### Scope 1 calculation equation and factor source

**TODO: confirm** the complete Scope 1 activity × factor equations, unit conversion rules, greenhouse-gas-specific GWP values, factor datasets/versioning, and whether Excel formulas or user-entered results are authoritative. The process-emissions importer reads total values from the workbook and assigns default zeroes for gas-specific values when absent; that is not a general Scope 1 calculation specification.

### Emission-factor provenance

Factor records are searchable in MongoDB and activity-row models include per-gas factor value, unit, source and source reference. User edits are stored separately. **TODO: confirm** authoritative factor publishers, exact factor releases, geography/year selection, licensing, and update/approval process. No specific emission-factor source should be presented as universal based on this repository alone.

## Scope 3

📋 **Planned — not implemented as an emissions ingestion/calculation capability.** Report schemas and LLM prompts contain Scope 3 fields/categories, but that does not provide validated Scope 3 activity capture or calculations. A future implementation would need category-level data models and validators, activity/factor datasets with geography/year provenance, calculation and unit-conversion rules, boundary/overlap controls, factor versioning, and test/evaluation coverage.

## Assumptions and limitations

- Workbook calculations and summary values may be inputs to ingestion; the backend does not prove every spreadsheet formula was recalculated or audited.
- Factor choice and source provenance may be incomplete; document assumptions and data limitations in reports.
- Report schemas support dual Scope 2 totals, but dashboard aggregation may expose only a single Scope 2 total in some paths. **TODO: confirm** consistency of location- and market-based basis across every chart/report.
- A model’s output or a report narrative is not emissions verification.
- Scope 3 references in report prompts and schemas must not be interpreted as implemented Scope 3 methodology.
- **TODO: confirm** validation methodology, significant-digit policy, recalculation policy and independent assurance requirements.
