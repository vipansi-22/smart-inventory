# StockFlow Pro - Product Requirements Document

## Overview
- **Summary**: Rebrand, refactor, and enterprise-harden an existing Spring Boot inventory-intelligence codebase into a polished, production-ready GitHub repository named "StockFlow Pro". Includes namespace migration to `com.vipan.stockflowpro`, Next.js 14 frontend scaffolding, staged conventional-commits history, and end-to-end CI/CD wiring.
- **Purpose**: Transform a raw template project into an organically grown enterprise repository that appears as a senior engineer built incrementally across 10 focused commits.
- **Target Users**: Vipan (author, the repository owner; hiring managers and technical reviewers evaluating the portfolio.

## Goals
- Migrate all Java source, build config, and resource files from `com.iie` namespace to `com.vipan.stockflowpro`.
- Strip all template placeholder branding (IIE, Project Antigravity, YOUR_USERNAME) and replace with StockFlow Pro / Vipan.
- Produce a frontend monorepo sibling: Next.js 14 (App Router) dashboard with auth guards and typed API client.
- Write a comprehensive README with Mermaid architecture, Docker local-setup, and API contract samples.
- Commit history: 10 sequential Conventional Commits on a fresh Git repo, pushed to `main` on GitHub.

## Non-Goals
- Rewrite business logic inside Spring services (preserve behavior.
- Implement additional backend feature work (no new REST endpoints beyond the existing).
- Deploy to cloud (K8s manifests remain untouched beyond rebrand-only edits).
- Commit secrets or push before the user provides their actual GitHub remote URL.

## Background & Context
Existing codebase at the workspace is a modular monolith (Spring Boot 3.4 + Java 21 Virtual Threads, Spring Modulith, PostgreSQL, Redis, Kafka, Flyway, JWT, Swagger, Prometheus) under package `com.iie.core`. No frontend exists yet. `.git` directory is absent; the repo must be freshly initialized.

## Functional Requirements
- **FR-1**: Every `package com.iie.*` declaration, `import com.iie.*` reference, and `com.iie` Maven/OpenAPI generator string is replaced with `com.vipan.stockflowpro`.
- **FR-2**: Project display name `inventory-intelligence-engine` and `IIE` replaced with `stockflowpro` / `StockFlow Pro` in pom, application.properties, compose.yaml, Dockerfile, README, CI, k8s, API YAMLs, and migration file headers.
- **FR-3**: `.gitignore` covers node_modules, .next, target, .env, .env.local, .DS_Store, IDE dirs, build artifacts, and Docker volumes.
- **FR-4**: `.github/workflows/ci.yml` builds/tests both Spring Boot (Maven) and Next.js (npm/pnpm), with PostgreSQL and Redis services.
- **FR-5**: README.md contains a Mermaid System Architecture Diagram, Tech Stack table, `docker compose up` local guide, and API contract (endpoints with JSON samples).
- **FR-6**: `src/main/resources/db/migration/V1__init_schema.sql` is present and rebranded from "Project Antigravity".
- **FR-7**: Frontend folder `frontend/` scaffolded: Next.js 14 App Router, `app/layout.tsx`, global Axios/fetch API client, client-side auth route guard, and dashboard page(s).
- **FR-8**: Git history contains exactly the 10 specified Conventional Commits staged/committed in the exact order requested, each with matching message.
- **FR-9**: Default branch named `main`.

## Non-Functional Requirements
- **NFR-1**: `mvnw clean compile` succeeds after refactor (syntax + package imports correct).
- **NFR-2**: OpenAPI generator config still generates code into the new namespace packages.
- **NFR-3**: Frontend `npm install && npm run build` succeeds (or at minimum scaffold with valid package.json/tsconfig).

## Constraints
- **Technical**: Java 21, Spring Boot 3.4, Maven, Next.js 14, Node LTS, PostgreSQL 17, Redis 7, Windows environment.
- **Business**: No author credit for original template; only Vipan + StockFlow Pro branding only.
- **Dependencies**: Existing Flyway migration file structure preserved (V1, V2, V3 retained but rebranded header comments.

## Assumptions
- Project name = "StockFlowPro" (dash-less for identifiers), display = "StockFlow Pro"), namespace = `com.vipan.stockflowpro` (as inferred from inventory theme).
- User will provide GitHub remote URL before the final `git push` step.
- User will confirm project name is acceptable if different; if not, we rename later.

## Acceptance Criteria

### AC-1: Namespace Refactor Complete
- **Type**: `rule`
- **Given**: All Java sources, pom.xml, OpenAPI YAML generator configs, application.properties
- **When**: We grep for the literal strings `com.iie` and `com/iie`
- **Then**: Zero matches remain, except possibly in comments under `target/generated-sources` (excluded by .gitignore)
- **Pass Condition**: `rg -n "com\.iie" src/ pom.xml src/main/resources --no-filename | wc -l` returns `0`
- **Evidence**: Shell grep output

### AC-2: Branding Sanitized
- **Type**: `rule`
- **Given**: pom.xml, README.md, application.properties, compose.yaml, Dockerfile, k8s/, .github/workflows/ci.yml, db/migration SQL headers
- **When**: We grep for `iie`, `IIE`, `inventory-intelligence-engine`, `Project Antigravity`, `YOUR_USERNAME`
- **Then**: Zero matches remain (case-insensitive for iie except where it's a coincidental English word; we check file contents after excluding those are rebranded)
- **Pass Condition**: Case-insensitive grep returns 0 matches across the above paths
- **Evidence**: Grep command output

### AC-3: .gitignore Coverage
- **Type**: `rule`
- **Given**: `.gitignore` at repo root
- **When**: Inspect the file contents
- **Then**: It must contain ignore patterns for node_modules, .next/, target/, .env*, .DS_Store, .idea/, .vscode/, *.log, Docker volumes
- **Pass Condition**: Each pattern present as a line
- **Evidence**: Read of `.gitignore` file

### AC-4: CI Backend + Frontend
- **Type**: `rule`
- **Given**: `.github/workflows/ci.yml`
- **When**: Read the YAML
- **Then**: It has a job (or jobs) for Maven build/test and a job/step for npm install && npm run build on the frontend; services postgres + redis
- **Pass Condition**: Workflow file exists and contains both steps
- **Evidence**: Read of ci.yml

### AC-5: README Content
- **Type**: `rubric`
- **Dimension**: README comprehensiveness and enterprise quality
- **Scale**: 1-5
- **Anchors**: 1 = minimal; 3 = has sections + diagram; 5 = Mermaid C4-style architecture, tech stack with concurrency/JWT/caching highlights, Docker compose guide, API endpoint table with request/response JSON samples, badge
- **Pass Threshold**: >= 4
- **Evidence**: Manual review of README.md

### AC-6: Flyway V1 Initial Schema Present
- **Type**: `rule`
- **Given**: `src/main/resources/db/migration/V1__init_schema.sql`
- **When**: File exists and contains valid SQL (no "Project Antigravity" header)
- **Then**: File path exists, header mentions StockFlow Pro
- **Pass Condition**: File exists; grep "Antigravity" returns 0; grep "StockFlow" returns >= 1
- **Evidence**: File read + grep

### AC-7: Next.js 14 Frontend Scaffolded
- **Type**: `rule`
- **Given**: `frontend/` directory at repo root
- **When**: ls frontend/
- **Then**: Contains package.json (next 14, react 18), app/layout.tsx, app/page.tsx, app/dashboard/page.tsx, app/login/page.tsx, api client lib (src/lib/api.ts or similar, with route guard middleware.ts or client HOC
- **Pass Condition**: All files exist; package.json `"next": "^14.x" 
- **Evidence**: directory listing + read of key files

### AC-8: 10 Conventional Commits
- **Type**: `rule`
- **Given**: freshly `git log --oneline` after Phase 3
- **When**: Count of commits equals 10 and messages match the exact 10 specified in order
- **Then**: `feat`, `fix`, `refactor`, `chore`, `test`, `docs`, `ci` scopes exactly as specified; 10 lines; on branch main
- **Pass Condition**: `git log --oneline | wc -l` == 10 and `git branch --show-current` == main
- **Evidence**: git log output

### AC-9: Compile Passes
- **Type**: `rule`
- **Given**: completed Phase1-2 code
- **When**: `./mvnw.cmd clean compile -DskipTests` (Windows)
- **Then**: Exit code 0, BUILD SUCCESS
- **Pass Condition**: Exit code 0
- **Evidence**: Maven stdout tail BUILD SUCCESS

## Open Questions
- [x] Project name → StockFlowPro (assumed; user may override later
- [x] Author name → Vipan (confirmed)
- [ ] GitHub remote URL → pending user input at Phase 4
