# Brasaland Backend Architecture Proposal

**Status: Proposed — not implemented**  
**Scope:** Centralized backend architecture for Brasaland Digital. This proposal uses `BRIEFING.md` as the business source of truth. Endpoint examples and folder names below are design proposals, not existing functionality or commitments to unspecified integrations.

## 1. Architectural pattern

Propose a **layered architecture inside a modular monolith**, delivered as one centralized FastAPI application under `services/api/`. It is one deployable backend with business capabilities separated into domain modules and clear responsibilities across layers.

This is a suitable starting point for Brasaland's 14 restaurants, small technology team, shared business data, and multiple departments. A common backend can provide consistent access to locations, menus, sales, customers, suppliers, and reports, while addressing the company's Colombia/Florida operations and COP/USD needs in shared rules rather than duplicating them across independently deployed services.

Starting with microservices would add service boundaries, deployment and monitoring overhead, inter-service communication, and data consistency decisions before the team or requirements establish a need for them. A modular monolith keeps the initial system simpler while preserving domain boundaries; a module could be extracted later if demonstrated scale, ownership, or operational requirements justify it.

## 2. Proposed backend structure

```text
services/
└── api/
    ├── README.md
    ├── main.py
    ├── config.py
    ├── dependencies.py
    ├── routers/
    │   ├── __init__.py
    │   ├── locations.py
    │   ├── menus.py
    │   ├── sales.py
    │   ├── customers.py
    │   ├── suppliers.py
    │   ├── telemetry.py
    │   └── reports.py
    ├── services/
    │   ├── locations.py
    │   ├── menus.py
    │   ├── sales.py
    │   ├── customers.py
    │   ├── suppliers.py
    │   ├── telemetry.py
    │   ├── reports.py
    │   └── currency.py
    ├── models/
    │   ├── locations.py
    │   ├── menus.py
    │   ├── sales.py
    │   ├── customers.py
    │   └── suppliers.py
    ├── schemas/
    │   ├── locations.py
    │   ├── menus.py
    │   ├── sales.py
    │   ├── customers.py
    │   ├── suppliers.py
    │   ├── telemetry.py
    │   └── reports.py
    ├── repositories/
    │   ├── locations.py
    │   ├── menus.py
    │   ├── sales.py
    │   ├── customers.py
    │   └── suppliers.py
    └── tests/
        ├── routers/
        ├── services/
        └── repositories/
```

This is an illustrative proposal; exact files and whether all domains need every layer remain to be confirmed.

| Layer                    | Responsibility                                                                                                                                                                                                                                       |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `main.py` and `routers/` | Create the FastAPI app, register API routers, and define HTTP paths, methods, and status/response behavior. Routers translate HTTP requests into application-service calls; they should not contain substantial business rules or persistence logic. |
| `schemas/`               | Define and validate API request and response data shapes. Keep the external API contract distinct from persistence models.                                                                                                                           |
| `services/`              | Apply business rules and coordinate operations across repositories. Shared rules—particularly monetary handling—should have a single owner rather than being repeated in route handlers or clients.                                                  |
| `repositories/`          | Encapsulate reading and writing persisted data. Services use repositories rather than depending on database details directly.                                                                                                                        |
| `models/`                | Represent persistence/domain records as appropriate to the chosen data technology. These are not automatically the same as public request/response schemas.                                                                                          |
| `config.py`              | Load and expose validated application configuration from environment-specific settings; avoid hard-coding credentials.                                                                                                                               |
| `dependencies.py`        | Provide reusable FastAPI dependencies, such as configuration, database access, and—once designed—authentication/authorization context.                                                                                                               |
| `tests/`                 | Test API behavior and business/service rules, with tests grouped by layer or domain. Test layout and tooling are yet to be selected.                                                                                                                 |

Domain separation means each domain's router, schemas, services, repositories, and models focus on its own concepts. Cross-domain work should be coordinated through service-level interfaces and shared rules, not by importing route handlers into one another.

## 3. Proposed API endpoints and routers

These examples translate the briefing's stated needs into possible resource paths. They are not implemented contracts. Authentication, authorization, pagination, filtering, payload details, and exact response formats require follow-up design.

| Domain router | Method | Example path                      | Purpose                                                                                          |
| ------------- | -----: | --------------------------------- | ------------------------------------------------------------------------------------------------ |
| Locations     |  `GET` | `/locations`                      | List restaurant locations for operational views.                                                 |
| Locations     |  `GET` | `/locations/{location_id}`        | Retrieve one location's details.                                                                 |
| Menus         |  `GET` | `/locations/{location_id}/menu`   | Retrieve a location's applicable menu.                                                           |
| Menus         |  `GET` | `/menus/{menu_id}`                | Retrieve a menu and its items.                                                                   |
| Sales         |  `GET` | `/sales`                          | Query sales records using agreed filters such as location and date range.                        |
| Sales         |  `GET` | `/sales/reports/summary`          | Retrieve an aggregated sales summary for reporting.                                              |
| Customers     | `POST` | `/customers`                      | Register a customer, subject to an agreed privacy and consent model.                             |
| Customers     |  `GET` | `/customers/{customer_id}`        | Retrieve an authorized customer profile.                                                         |
| Suppliers     |  `GET` | `/suppliers`                      | List suppliers for procurement visibility.                                                       |
| Suppliers     |  `GET` | `/suppliers/{supplier_id}/prices` | Retrieve supplier price history, if the required data is available.                              |
| Telemetry     | `POST` | `/telemetry/events`               | Receive a location event or measurement; event format and POS/device integration are unresolved. |
| Reports       |  `GET` | `/reports/operations`             | Retrieve an operations report for an authorized scope.                                           |
| Reports       |  `GET` | `/reports/executive/sales`        | Retrieve an executive sales view with currency treatment defined by approved business rules.     |

Each domain module can expose an `APIRouter`. `main.py` creates the FastAPI application and includes the routers (for example, using `include_router`), optionally applying shared prefixes, tags, and dependencies. This composes domain routes into one API while allowing the route definitions to remain in separate files. It does not by itself provide access control; authorization must be designed and enforced explicitly.

## 4. FastAPI documentation and architectural implications

The following distinguishes **official framework guidance** from **project-specific choices** in this proposal.

| Official documentation                                                                             | FastAPI convention or guidance                                                                                                                               | Project-specific application here                                                                                                                                                                                       |
| -------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Bigger Applications — Multiple Files](https://fastapi.tiangolo.com/tutorial/bigger-applications/) | FastAPI documents splitting an application across modules, defining routes with `APIRouter`, and composing them with `include_router`.                       | Use one centralized FastAPI app with domain routers for locations, menus, sales, customers, suppliers, telemetry, and reports. The exact package layout and modular-monolith choice are Brasaland design proposals.     |
| [Request Body](https://fastapi.tiangolo.com/tutorial/body/)                                        | FastAPI uses Pydantic models to declare request bodies and provide parsing and validation, including JSON input.                                             | Keep API request/response schemas in `schemas/`, separate from persistence models. The domain schemas and validation rules must be worked out from approved requirements.                                               |
| [Settings and Environment Variables](https://fastapi.tiangolo.com/advanced/settings/)              | FastAPI shows settings based on Pydantic Settings, environment-variable loading, dependency use, and test overrides.                                         | Centralize backend configuration in `config.py`, inject it through dependencies, and supply environment-specific values outside source code. Choosing the exact settings package and deployment mechanism remains open. |
| [CORS](https://fastapi.tiangolo.com/tutorial/cors/)                                                | FastAPI describes CORS as browser cross-origin policy, defines an origin by scheme/host/port, and demonstrates `CORSMiddleware` with explicit configuration. | Configure only approved frontend origins for each environment. Do not treat CORS as authentication or authorization. The actual origins are not specified yet.                                                          |

## 5. Frontend and backend separation

### Repository organization

A **monorepo** keeps related frontends, backend, data work, shared packages, and documentation in one repository. Separate repositories can provide independent ownership and release cycles, but require additional coordination for API changes, shared contracts, permissions, and development setup.

Keeping Brasaland's existing monorepo is recommended: the company is starting from near-zero, the engineering team is small, and the system's data and workflows cross departments. It also follows the root README's layout, which places user interfaces in `uis/`, backend APIs in `services/`, and reusable code in `packages/`. This recommendation does not require frontend and backend to be deployed as one application.

### Communication and configuration

A browser-based frontend should communicate with the backend over HTTP, exchanging JSON request and response bodies for the API operations that use JSON. The frontend should call an environment-configured API base URL; it should not connect directly to backend databases or hold server credentials.

Development and production should use separate configuration values, including the frontend's API URL and the backend's allowed CORS origins. Store environment-specific values outside committed source code, protect secrets in the appropriate server-side environment, and avoid using production credentials for local development. The actual development/production URLs and secret-management platform are not decided here.

CORS is needed when browser code at one origin calls an API at another origin. Configure the backend's allowed origins deliberately per environment; avoid broad wildcard access, especially when credentials are involved. CORS controls browser access behavior, not identity, permissions, or protection against non-browser clients.

The [Next.js environment variables guide](https://nextjs.org/docs/app/guides/environment-variables) explains that variables prefixed with `NEXT_PUBLIC_` are included for browser use, while non-prefixed variables are available to the server environment. Therefore, only values intended to be public—such as a public API base URL—may use a public prefix. Database passwords, signing keys, private API tokens, and other credentials must remain server-side and must never be exposed to frontend bundles. A frontend-visible value is not secret merely because it originated in an environment file.

## 6. Risks and points of attention

| Risk                                                                                | Prevention strategy                                                                                                                                                                                                                                                                                                                                                                                 |
| ----------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Responsibilities become mixed between routers, services, repositories, and schemas. | Keep HTTP handling in routers, business rules in services, persistence in repositories, and request/response validation in schemas. Review changes against these boundaries and test important service rules independently.                                                                                                                                                                         |
| Business logic is duplicated, especially currency conversion.                       | Define one authoritative backend service/rule for any approved conversion behavior and have reports and other domains reuse it. Record the source, effective date, precision/rounding, and whether values are stored or merely displayed only after the business supplies those requirements. Do not hard-code or invent exchange rates.                                                            |
| Sensitive information is exposed or access is unauthorized.                         | Keep secrets on the server; validate inputs; require authentication and authorization appropriate to each operation; restrict access to customer and employee data; avoid returning unnecessary fields; and review logs and error responses for sensitive content. The specific identity system, roles, privacy rules, and retention requirements are unresolved. CORS alone is not access control. |

## 7. References and unresolved decisions

### References

- Brasaland source of truth: [`BRIEFING.md`](../BRIEFING.md)
- [`memory-bank/projectbrief.md`](../memory-bank/projectbrief.md) — business goals and source-of-truth reminder.
- [`memory-bank/techContext.md`](../memory-bank/techContext.md) — planned FastAPI stack and technical constraints.
- [`README.md`](../README.md), [`services/README.md`](../services/README.md), and [`uis/README.md`](../uis/README.md) — monorepo placement and centralized API guidance.
- [FastAPI: Bigger Applications — Multiple Files](https://fastapi.tiangolo.com/tutorial/bigger-applications/)
- [FastAPI: Request Body](https://fastapi.tiangolo.com/tutorial/body/)
- [FastAPI: Settings and Environment Variables](https://fastapi.tiangolo.com/advanced/settings/)
- [FastAPI: CORS](https://fastapi.tiangolo.com/tutorial/cors/)
- [Next.js: How to use environment variables](https://nextjs.org/docs/app/guides/environment-variables)

### Decisions requiring clarification

Before implementation, confirm or specify:

- Persistence technology, data model, migrations, and how test data will be managed.
- Authentication, role/permission model, and handling of customer and employee personal data.
- POS systems in each country, available integration interfaces, ownership of sales data, and telemetry/event formats and delivery expectations.
- Currency rules: which currency is recorded per transaction, conversion source and timing if required, rounding, and reporting presentation. No exchange rate is assumed by this proposal.
- API versioning, error format, pagination/filter conventions, and endpoint contracts.
- Development and production hosting, environment/secrets management, logging, monitoring, and deployment process. No deployment target is assumed.
- Which report calculations and operational thresholds are approved, and what latency “real-time” requires for each use case.
- Frontend origins per environment and whether credentialed cross-origin requests are required.
- Test, lint, and dependency-management conventions for the planned Python service.

These are open technical or product decisions, not inferred requirements. The architecture remains a proposal until reviewed and approved.
