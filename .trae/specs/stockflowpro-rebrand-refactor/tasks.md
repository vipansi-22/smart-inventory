# StockFlow Pro - Implementation Plan

## Task 1: Deep-Scan & Refactor Namespaces (com.iie → com.vipan.stockflowpro)
- **Status**: `pending`
- **Priority**: high
- **Depends On**: None
- **Description**:
  - Refactor all `package com.iie.*` declarations and `import com.iie.*` in every Java file (src/main/java, src/test/java).
  - Update pom.xml: `<groupId>`, `<artifactId>`, `<name>`, and OpenAPI generator `apiPackage`/`modelPackage` entries.
  - Rename directories on disk: `src/main/java/com/iie/**` → `src/main/java/com/vipan/stockflowpro/**` (same for test).
  - Update package-info.java and any other textual references.
- **Acceptance Criteria Addressed**: AC-1, AC-9
- **Test Requirements**:
  - `rule` TR-1.1: `rg "com\.iie|com/iie" src/ pom.xml --glob '!target/**'` returns 0 lines of output
  - `rule` TR-1.2: `.\mvnw.cmd clean compile -DskipTests` exits 0 (BUILD SUCCESS)
- **Notes**: Rename directories via PowerShell commands; case-sensitive.

## Task 2: Brand Sanitization (IIE / Project Antigravity / YOUR_USERNAME → StockFlow Pro / Vipan)
- **Status**: `pending`
- **Priority**: high
- **Depends On**: None
- **Description**:
  - pom.xml: `<name>`, `<description>`, artifactId, and any branding strings.
  - application.properties: `spring.application.name=stockflowpro`, DB name `stockflowpro`.
  - compose.yaml: container names, networks, DB env vars (POSTGRES_DB, CLUSTER_ID, container_name prefixed `stockflow-`).
  - Dockerfile: non-root user `stockflow`, group `stockflow`.
  - README.md: title, description, badge URL, architecture diagram labels.
  - .github/workflows/ci.yml: DB name, badge comments.
  - k8s/*.yaml: app labels, image references if present.
  - src/main/resources/api/*.yaml: title/description ("StockFlow Pro - Inventory API" etc.).
  - db/migration/V1*.sql, V2*, V3*: headers change from "Project Antigravity" to "StockFlow Pro".
  - deploy.sh, logback-spring.xml: any references.
- **Acceptance Criteria Addressed**: AC-2
- **Test Requirements**:
  - `rule` TR-2.1: Case-insensitive grep over pom.xml, README.md, src/main/resources/, compose.yaml, Dockerfile, .github/, k8s/, deploy.sh, logback-spring.xml for `iie|inventory-intelligence-engine|Project Antigravity|YOUR_USERNAME` (excluding English word coincidences) returns 0 matches.
  - `rule` TR-2.2: grep for `StockFlow` in README.md >= 5 occurrences.
- **Notes**: Preserve `IIE` only if it appears inside a generated code folder (excluded via target/ .gitignore).

## Task 3: Harden .gitignore
- **Status**: `pending`
- **Priority**: medium
- **Depends On**: None
- **Description**:
  - Ensure .gitignore covers: `node_modules/`, `.next/`, `.out/`, `target/`, `.env`, `.env.*`, `.DS_Store`, `*.log`, `.idea/`, `.vscode/`, `*.swp`, `postgres_data/`, `redis_data/`, `kafka_data/`, `*.iml`, `*.iws`, `*.ipr`, `frontend/.next/`, `frontend/node_modules/`.
- **Acceptance Criteria Addressed**: AC-3
- **Test Requirements**:
  - `rule` TR-3.1: Each of the patterns listed above exists as a line (or glob) in `.gitignore`.
- **Notes**: Do not duplicate entries.

## Task 4: Rewrite GitHub Actions CI (Backend + Frontend)
- **Status**: `pending`
- **Priority**: high
- **Depends On**: Task 7 (frontend must exist with package.json first; but CI can be written generically)
- **Description**:
  - Rewrite `.github/workflows/ci.yml` with two jobs: `build-backend` (Maven + JDK21 + Postgres/Redis services + mvn test), `build-frontend` (setup node LTS + cd frontend/ + npm ci + npm run build).
  - Or single job with parallel matrix; prefer matrix-less with explicit jobs.
  - Keep postgres/redis service in backend job.
- **Acceptance Criteria Addressed**: AC-4
- **Test Requirements**:
  - `rule` TR-4.1: YAML file parses (can check via `yamllint`-style structure) → jobs: `build-backend`, `build-frontend` both present; contains `npm run build` inside a step referencing `frontend/`.
  - `rule` TR-4.2: Services (postgres, redis) defined under backend job; env vars for datasource/redis host.

## Task 5: Comprehensive README.md with Mermaid, Docker, API Contracts
- **Status**: `pending`
- **Priority**: high
- **Depends On**: Task 7 (so we can mention frontend setup)
- **Description**:
  - Title "StockFlow Pro" with CI badge URL placeholder pointing to the user's future GH repo.
  - System Architecture Diagram using Mermaid (C4-style: Client → Next.js Frontend → Spring Boot API → Postgres/Redis/Kafka).
  - Tech Stack table (Java 21 Virtual Threads, Spring Boot 3.4, Spring Modulith, Spring Security 6 JWT stateless, Flyway, Redis caching/Lua, Kafka events, Next.js 14 App Router).
  - Engineering Highlights bullet list (Concurrency via virtual threads + Redis Lua atomicity, Stateless JWT filter chain, Redis-cached stock reads).
  - Local Setup guide: `docker compose up -d`; `./mvnw spring-boot:run`; `cd frontend && npm install && npm run dev`.
  - API Contract Overview: table of endpoints (POST /api/auth/login, GET/POST /inventory/{sku}, POST /inventory/{sku}/reserve) with JSON request/response samples.
  - Demo users table.
- **Acceptance Criteria Addressed**: AC-5
- **Test Requirements**:
  - `rubric` TR-5.1: README comprehensiveness; scale 1-5; 1 = missing sections, 3 = present but sparse, 5 = Mermaid diagram renders + Docker setup + JSON request/response samples for login and reserve endpoints + Tech Stack highlights concurrency/JWT/caching; threshold >= 4.
  - `rule` TR-5.2: README contains ```mermaid code block with at least `flowchart` or `C4Context` keyword.

## Task 6: Flyway Migration Headers Rebrand + Confirm V1 Present
- **Status**: `pending`
- **Priority**: medium
- **Depends On**: None
- **Description**:
  - Rewrite comment header of V1__init_schema.sql, V2__audit_logs_partitioned.sql, V3__event_publication.sql to "StockFlow Pro - Schema Migration".
  - Ensure V1 path exists and is well-formed SQL (already does; rebrand only).
- **Acceptance Criteria Addressed**: AC-6
- **Test Requirements**:
  - `rule` TR-6.1: grep -i "Antigravity" src/main/resources/db/migration/*.sql returns 0
  - `rule` TR-6.2: grep -i "StockFlow" src/main/resources/db/migration/V1__init_schema.sql returns >= 1
  - `rule` TR-6.3: File path `src/main/resources/db/migration/V1__init_schema.sql` exists

## Task 7: Scaffold Next.js 14 Frontend in ./frontend
- **Status**: `pending`
- **Priority**: high
- **Depends On**: None
- **Description**:
  - Create `frontend/` directory with Next.js 14 App Router project skeleton (package.json with next@14, react@18, react-dom@18, typescript, @types/node, @types/react, @types/react-dom, eslint, eslint-config-next, tailwindcss@3, postcss, autoprefixer, axios).
  - tsconfig.json strict, next.config.ts (or .mjs).
  - app/layout.tsx with basic metadata (StockFlow Pro title), Tailwind import globals.css.
  - app/page.tsx (Landing: sign-in CTA link).
  - app/login/page.tsx with login form stub.
  - app/dashboard/page.tsx with dashboard layout skeleton (sidebar nav, stats cards).
  - `lib/api.ts` or `app/_lib/apiClient.ts` axios-based client with base URL http://localhost:8080 and Authorization Bearer token from localStorage/cookie.
  - Client-side route guard via middleware.ts (protect /dashboard*) redirecting /login when token absent.
  - .env.example with NEXT_PUBLIC_API_BASE_URL=http://localhost:8080.
  - next-env.d.ts auto-generated placeholder.
- **Acceptance Criteria Addressed**: AC-7
- **Test Requirements**:
  - `rule` TR-7.1: Files exist: frontend/package.json, frontend/tsconfig.json, frontend/app/layout.tsx, frontend/app/page.tsx, frontend/app/login/page.tsx, frontend/app/dashboard/page.tsx, frontend/middleware.ts, frontend/lib/api.ts or frontend/app/_lib/apiClient.ts, frontend/.env.example, frontend/next.config.ts or .mjs
  - `rule` TR-7.2: frontend/package.json "dependencies"."next" matches string containing "14." (semver range ^14.x acceptable)
  - `rule` TR-7.3: middleware.ts contains matcher guarding /dashboard* (or equivalent guard pattern)

## Task 8: Initialize Git, Stage & Commit in 10 Sequential Steps
- **Status**: `pending`
- **Priority**: high
- **Depends On**: Task 1, Task 2, Task 3, Task 4, Task 5, Task 6, Task 7 (all code changes complete before staging commits)
- **Description**:
  - `git init -b main` in project root.
  - Stage subset of files matching the scope of each commit message, then `git commit -m "exact message"`.
  - Commit order + message exact:
    1. `chore(setup): scaffold initial monorepo structure with spring boot 3 and nextjs`
    2. `feat(config): configure application properties, docker-compose and environment variables`
    3. `feat(db): establish flyway schema migrations and entity models`
    4. `feat(security): implement spring security 6 filter chain with stateless jwt validation`
    5. `feat(api): implement core service layer, dtos, and restful controllers`
    6. `test(backend): add junit5 and mockito unit tests for service business logic`
    7. `feat(frontend): setup nextjs app router, layout, and global api client`
    8. `feat(ui): implement responsive dashboard and client-side route guards`
    9. `ci: configure github actions build and automated test workflow`
    10. `docs(readme): add system architecture diagram, api contracts, and setup instructions`
- **Acceptance Criteria Addressed**: AC-8
- **Test Requirements**:
  - `rule` TR-8.1: `git log --oneline | measure-object -line` (PowerShell) equals exactly 10
  - `rule` TR-8.2: `git branch --show-current` outputs `main`
  - `rule` TR-8.3: git log messages, in order, match the 10 specified strings (allow for commit message + description; exact first line)

## Task 9: Compile Backend Final Verification
- **Status**: `pending`
- **Priority**: high
- **Depends On**: Task 1, Task 2, Task 6
- **Description**:
  - Run `.\mvnw.cmd clean compile -DskipTests` on the refactored codebase.
  - Inspect output; fix any import/package errors that leak through Task 1.
- **Acceptance Criteria Addressed**: AC-9
- **Test Requirements**:
  - `rule` TR-9.1: BUILD SUCCESS printed and exit code 0.

## Task 10: Remote Push Preparation (wait for user GitHub URL)
- **Status**: `pending`
- **Priority**: medium
- **Depends On**: Task 8, Task 9
- **Description**:
  - When user provides repo URL (e.g. `git@github.com:vipan/stockflowpro.git`):
    - `git remote add origin <URL>`
    - `git push -u origin main`
  - If URL is not yet provided, document the expected command block to run later.
- **Acceptance Criteria Addressed**: Goal (Phase 4)
- **Test Requirements**:
  - `rule` TR-10.1: `git remote -v` shows `origin` when added.
