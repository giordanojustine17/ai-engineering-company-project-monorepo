# Technical Context — Brasaland

## Architecture

The project uses a monorepo structure, keeping all Brasaland applications, services, and shared resources in one repository.

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
