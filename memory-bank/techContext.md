# Technical Context — Brasaland

## Architecture

The project uses a monorepo structure, keeping all Brasaland applications, services, and shared resources in one repository. The Backend Architecture Proposal (`docs/ARCHITECTURE_PROPOSAL.md`) recommends a layered modular monolith: one centralized FastAPI backend, organized into domain routers and distinct architectural responsibilities.

The proposed layers separate HTTP routing, request/response schemas, business services, repositories, persistence/domain models, configuration, and dependencies. Domain routers are proposed for locations, menus, sales, customers, suppliers, telemetry, and reports, composed by the main FastAPI application. These are architectural plans only; no backend application or functionality has been implemented.

## Technology Stack

- Git and GitHub for version control.
- GitHub Codespaces for development.
- TypeScript for shared types and utilities.
- FastAPI for the planned centralized backend.
- Markdown for project documentation and AI instructions.

## Repository Structure

- uis/website/ — Public-facing website.
- uis/backoffice/ — Internal management application.
- services/ — Backend APIs.
- data/ — Datasets and data pipelines.
- agents/ — Product AI agents.
- skills/ — Product agent capabilities.
- packages/ — Shared libraries and types.
- memory-bank/ — AI project memory.
- .agents/ — AI coding assistant rules and skills.

## Technical Constraints

- Follow the existing monorepo structure.
- Read folder README files before making changes.
- Do not create duplicate application structures.
- Support USD and COP currencies.
- Follow the business requirements in BRIEFING.md.
- Keep the memory bank updated as the project evolves.
