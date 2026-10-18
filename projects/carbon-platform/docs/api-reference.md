Purpose: Inventory active FastAPI routes mounted by `Backend/app/main.py`.

Status: In active development. Scope 3 and commercial features are not yet implemented.

Every listed route is ✅ Implemented and mounted by the current FastAPI application. This is a source-derived inventory, not a versioned OpenAPI contract. Auth means the endpoint has a bearer-token dependency in the mounted handler; it does not imply complete resource ownership enforcement. Request and response summaries are intentionally brief. The legacy `/login_old` handler is commented out. Sheet and project-location-year routers exist in source but are not mounted by `main.py`, so they are excluded.

## Authentication (root paths)

| Method | Path | Purpose | Auth | Request → response summary |
|---|---|---|---|---|
| POST | `/login` | Authenticate and issue access/refresh tokens | No | Email/password → token pair and user summary |
| POST | `/refresh` | Exchange a valid stored refresh token | No | Refresh token → access token |
| POST | `/logout` | Revoke refresh token and remove session | No | Refresh token → message |
| POST | `/register` | Register company and user | No | Registration payload → login/token response |
| POST | `/forget-password` | Send password-reset email | No | Email payload → message |
| POST | `/set-new-password` | Set password using reset parameters | No | Email, password and security token → message |

## Users — `/api/users`

| Method | Path | Purpose | Auth | Request → response summary |
|---|---|---|---|---|
| GET | `/api/users/` | Get current user data | Yes | JWT → user data |
| POST | `/api/users/` | Create user | Yes (Admin+) | User payload → result |
| POST | `/api/users/register` | Register user | No | User payload → result |
| GET | `/api/users/list_users` | List users | Yes (Manager+) | Optional company ID → user list |
| GET | `/api/users/list_users_with_roles` | List users with roles | Yes (User+) | Optional company ID → user/role list |
| POST | `/api/users/{userid}/update_role` | Update user role | Yes (Admin+) | Role payload → result |
| GET | `/api/users/{userid}/get_user` | Fetch user profile | Yes (User+) | User ID → user |
| POST | `/api/users/{userid}/update_profile` | Update profile | Yes | Profile payload → result |
| POST | `/api/users/change-password` | Change current password | Yes | Current/new password → result |
| POST | `/api/users/{userid}/update_admin_role` | Update elevated role | Yes (Alpin Admin+) | Role payload → result |
| GET | `/api/users/list_alpin_employees` | List internal employees | Yes (Alpin Admin+) | JWT → employee list |
| POST | `/api/users/create_alpin_employee` | Create internal employee | Yes (Alpin Admin+) | User payload → result |
| POST | `/api/users/reset_password` | Reset user password | Yes (Alpin Admin+) | Reset payload → result |

## Companies — `/api/companies`

| Method | Path | Purpose | Auth | Request → response summary |
|---|---|---|---|---|
| POST | `/api/companies/` | Create company | Yes (Alpin Admin+) | Company payload → created record/result |
| PUT | `/api/companies/{company_id}/approve` | Approve company | Yes (Alpin Admin+) | Company ID → status |
| POST | `/api/companies/{company_id}/change_status` | Change company status | Yes (Alpin Admin+) | Status payload → status |
| PUT | `/api/companies/{company_id}/delete` | Soft-delete company | Yes (Alpin Admin+) | Company ID → result |
| PUT | `/api/companies/{company_id}/update` | Update company | Yes (Admin+) | Partial company payload → result |
| GET | `/api/companies/{company_id}/getcompany` | Fetch company | Yes (User+) | Company ID → company record |
| GET | `/api/companies/companies_list` | List companies | Yes (Alpin Manager+) | Optional filters → list |
| GET | `/api/companies/companies_list_all` | List all companies | Yes (Alpin Manager+) | JWT → list |
| POST | `/api/companies/register` | Company registration flow | No | Registration payload → registration result |

## Projects and locations

| Method | Path | Purpose | Auth | Request → response summary |
|---|---|---|---|---|
| POST | `/api/projects/{company_id}/` | Create project | Yes (Manager+) | Project payload → project |
| POST | `/api/projects/{company_id}/update` | Update project | Yes (Admin+) | Project payload → result |
| PUT | `/api/projects/{company_id}/{project_id}/approve` | Approve project | Yes (Alpin Manager+) | IDs → result |
| GET | `/api/projects/{company_id}/` | List projects | Yes | Company ID → projects |
| POST | `/api/projects/{company_id}/{project_id}/delete-project-with-associated-sheets` | Soft-delete project and associated sheet data | Yes (Admin+) | IDs → result |
| POST | `/api/projects/{project_id}/locations` | Create project location | Yes (User+) | Location payload → location |
| GET | `/api/projects/{project_id}/locations` | List project locations | Yes (User+) | Project ID → locations |
| DELETE | `/api/projects/{project_id}/locations/{location_id}` | Delete project location | Yes (User+) | IDs → result |
| POST | `/api/company-years/` | Add company reporting year | Yes (User+) | Company-year payload → result |
| GET | `/api/company-years/{Company_ID}` | List company years | Yes (User+) | Company ID → years |
| DELETE | `/api/company-years/{ID}/{Company_ID}/{Year}` | Remove company year | Yes (User+) | IDs/year → result |

## Scopes, activity and emissions

| Method | Path | Purpose | Auth | Request → response summary |
|---|---|---|---|---|
| POST | `/api/projects/scopes` | Create scope instance | Yes (User+) | Scope payload → instance |
| GET | `/api/projects/{project_id}/scopes` | List project scopes | Yes (User+) | Project ID → scopes |
| GET | `/api/projects/{project_id}/scopes/{scope_id}` | Fetch scope | Yes (User+) | IDs → scope |
| GET | `/api/projects/{project_id}/activity-rows/{year}` | List activity rows by project/year | Yes (User+) | Project ID/year → rows and totals |
| DELETE | `/api/projects/{project_id}/scopes/{scope_id}` | Delete scope | Yes (Manager+) | IDs → result |
| GET | `/api/projects/{project_id}/scopes/{scope_id}/rows/{category_key}` | List category activity rows | Yes (User+) | IDs/category → rows |
| POST | `/api/projects/{project_id}/scopes/{scope_id}/rows` | Create activity row | Yes (User+) | Row payload → row |
| POST | `/api/projects/rows/{row_id}` | Update activity row | Yes (User+) | Row payload → result |
| DELETE | `/api/projects/rows/{row_id}` | Delete activity row | Yes (User+) | Row ID → result |
| POST | `/emissions/{process_type}/upload` | Upload and validate process-emissions workbook | Yes (User+) | `.xlsx`, project/location → pending preview |
| POST | `/emissions/confirm-upload/{upload_id}` | Confirm pending process-emissions import | Yes (User+) | Upload ID and optional period/scope data → stored-row summary |
| GET | `/emissions/{process_type}` | Get process-emissions data | Yes | Process type → response |
| POST | `/api/category-emission-results/` | Create category result | Yes (User+) | Category result payload → result |
| POST | `/api/category-emission-results/update` | Update category result | Yes (User+) | Result payload → result |
| GET | `/api/category-emission-results/{result_id}` | Get category result | Yes (User+) | Result ID → result |
| GET | `/api/category-emission-results/project/{project_id}` | List project results | Yes (User+) | Project ID → results |
| GET | `/api/category-emission-results/scope/{scope_instance_id}` | List scope results | Yes (User+) | Scope ID → results |
| DELETE | `/api/category-emission-results/{result_id}` | Delete category result | Yes (User+) | Result ID → result |
| DELETE | `/api/category-emission-results/project/{project_id}` | Delete project results | Yes (Admin+) | Project ID → result |

## Templates, factors and reference data

| Method | Path | Purpose | Auth | Request → response summary |
|---|---|---|---|---|
| GET | `/api/templates/{scope}/download` | Download a stored template | Yes (User+) | Scope/category filters → workbook |
| GET | `/api/templates/{scope}/schema` | Read template schema | Yes (User+) | Scope/category filters → schema |
| POST | `/api/templates/{scope}/upload` | Upload template | Yes (User+) | Template file/metadata → result |
| POST | `/api/templates/confirm-upload/{upload_id}` | Confirm template upload | Yes (User+) | Upload ID → result |
| GET | `/conversion-factors/all` | List factors | No | Filters → factor records |
| GET | `/conversion-factors/scope/{scope}/year/{year}` | Filter factors by scope/year | No | Scope/year → factor records |
| GET | `/conversion-factors/` | Search/list factors | No | Query filters → factor records |
| GET | `/conversion-factors/filters` | Get factor filters | No | None/query → filter options |
| GET | `/conversion-factors/filters-years` | List factor years | No | None → years |
| GET | `/conversion-factors/filters-scopes` | List factor scopes | No | None → scopes |
| GET | `/conversion-factors/distinct-search` | Distinct factor-value search | No | Search params → matches |
| GET | `/conversion-factors/search-by-values` | Search factors by supplied values | No | Up to eight parameters → matches |
| GET | `/api/sectors/list` | List sectors | No | None → sector list |
| POST | `/api/emission-factor/edit` | Record factor edit | Yes (User+) | Edit payload → edit record |
| POST | `/api/emission-factor/edit-internal-file` | Record factor edit with internal reference file | Yes (User+) | Edit/file payload → edit record |
| GET | `/api/emission-factor/{row_id}/edits` | List factor edits for row | Yes (User+) | Row ID → edit list |
| GET | `/api/emission-factor/{edit_id}/download-reference` | Download factor-edit reference | Yes (User+) | Edit ID → file response |

## Dashboards and reporting

All dashboard routes below are authenticated; several require `User` or `Alpin Manager` minimum roles. See handler dependencies for exact role. Their responses are summary/aggregation payloads, generally query filters to dashboard data.

| Method | Path | Purpose | Auth |
|---|---|---|---|
| GET | `/api/dashboard/emissions-by-company` | Company emissions comparison | Yes (Alpin Manager+) |
| GET | `/api/dashboard/{company_id}/kpi` | Company KPI summary | Yes (User+) |
| GET | `/api/dashboard/{company_id}/emissions-trend` | Emissions trend | Yes |
| GET | `/api/dashboard/{company_id}/scope-contribution` | Scope contribution | Yes |
| GET | `/api/dashboard/{company_id}/category-hotspot` | Category hotspot | Yes |
| GET | `/api/dashboard/{company_id}/category-methodology-data` | Category methodology detail | Yes |
| GET | `/api/dashboard/{company_id}/gas-composition` | Gas breakdown | Yes |
| GET | `/api/dashboard/{company_id}/ef-override-kpi` | Emission-factor override KPI | Yes |
| GET | `/api/dashboard/{company_id}/ef-override-by-category` | Factor overrides by category | Yes |
| GET | `/api/dashboard/{company_id}/project-count` | Project count | Yes (User+) |
| GET | `/api/dashboard/company-comparison/total-emissions` | Compare company totals | Yes |
| GET | `/api/dashboard/project-comparison/total-emissions` | Compare project totals | Yes |
| GET | `/api/dashboard/project-comparison/entity-location-emissions` | Compare location/entity totals | Yes |
| GET | `/api/dashboard/company-comparison/scope-split` | Compare company scope split | Yes |
| GET | `/api/dashboard/company-comparison/trend` | Compare company trends | Yes |
| GET | `/api/dashboard/company-comparison/heatmap` | Company emissions heatmap | Yes |
| GET | `/api/dashboard/company-comparison/gas-composition` | Compare gas composition | Yes |
| GET | `/api/dashboard/{company_id}/yoy-emissions-kpi` | Year-on-year KPI | Yes |
| GET | `/api/dashboard/{company_id}/emissions-by-year-category` | Annual category totals | Yes |
| GET | `/api/dashboard/{company_id}/emissions-by-year-scope` | Annual scope totals | Yes |
| GET | `/api/dashboard/{company_id}/yoy-by-category` | Year-on-year category comparison | Yes |
| GET | `/api/dashboard/sector-avg-emissions` | Sector averages | Yes |
| GET | `/api/dashboard/sector-comparison/average-scope-split` | Sector-average scope split | Yes |
| GET | `/api/dashboard/sector-comparison/overall-emissions-trend` | Sector emissions trend | Yes |

Report endpoints use bearer auth and at least User role. Inputs are report requests or company/job identifiers; outputs are job metadata, status, a time-limited download token, or document bytes.

| Method | Path | Purpose | Auth |
|---|---|---|---|
| POST | `/api/reports/{company_id}/jobs/{job_id}/archive` | Archive report job | Yes (User+) |
| POST | `/api/reports/{company_id}/jobs/{job_id}/restore` | Restore report job | Yes (User+) |
| GET | `/api/reports/{company_id}/activity-period` | Infer activity date range | Yes (User+) |
| POST | `/api/reports/{company_id}/generate` | Queue methodology report generation | Yes (User+) |
| DELETE | `/api/reports/{company_id}/jobs/{job_id}` | Delete job/file | Yes (User+) |
| GET | `/api/reports/{company_id}/status/{job_id}` | Get job status/download token | Yes (User+) |
| GET | `/api/reports/{company_id}/download` | Download generated report | Yes; also uses download token |
| GET | `/api/reports/{company_id}/jobs` | List report jobs | Yes (User+) |

## AI and email

| Method | Path | Purpose | Auth | Request → response summary |
|---|---|---|---|---|
| POST | `/api/assistant/suggest` | Generate short form-field suggestions | Yes | Field/context → options array |
| POST | `/api/email/send-email` | Send email service request | No | Email payload → send result |

## Routes not mounted by `main.py`

The source includes `sheet_routes.py`, `sheet_data_api.py`, and `project_location_year_routes.py`, but their routers are not included in the current FastAPI app. They are omitted above. Some source files have duplicate route decorators (including duplicate sheet POST paths); these should not be treated as supported public API.

**TODO: confirm** response schemas/HTTP statuses and publish a generated OpenAPI specification as the authoritative contract.
