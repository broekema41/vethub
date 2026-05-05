# Agent Notes

## What is VetHub
- VetHub is a demo pet clinic/pet store application with a SvelteKit frontend and a Spring Boot backend.
- It manages owners, pets, vets, specialties, and visits through REST APIs.
- The backend exposes OpenAPI docs and the frontend can generate typed client models from that spec.
- The default dev setup uses an H2 in-memory database with Liquibase-managed schema and seed data.

## Tooling versions (mise)
- Java 25 (Temurin), Node 22.20.0, Bun 1.3.0.

## Repo layout
- `client/` is a SvelteKit app (Vite). `server/` is a Spring Boot app built with Gradle Kotlin DSL.

## Architecture (concise)
- Tech stack: SvelteKit + Vite (frontend), Spring Boot + Gradle Kotlin DSL + H2 + Liquibase + MapStruct + springdoc-openapi (backend).
- Structure: UI in `client/`; API/domain/data in `server/src/main/java`; Liquibase changelogs in `server/src/main/resources/db/changelog`.
- Patterns/conventions: controllers -> services -> repositories; MapStruct mappers for request/response mapping; OpenAPI JSON drives typed client models.
- Layer connections: frontend calls REST endpoints under `/api`; OpenAPI at `/api/v1/public/docs` feeds `client/src/lib/types/api.d.ts`.

## Backend (server/)
- Run the API: `./gradlew bootRun` (serves under `/api`; OpenAPI JSON at `http://localhost:8080/api/v1/public/docs`).
- Tests: `./gradlew test` (JUnit 5; uses `src/test/resources/application-test.yml`).
- Formatting + lint: `spotlessApply` runs before Java compile; `-Werror` is enabled, so warnings fail builds.
- Liquibase: default `application.yml` uses `contexts: prd`; dev profile (`application-dev.yml`) switches to `contexts: tst` and `drop-first: true`.

## Testing (server/)
- JUnit 5 tests live under `server/src/test/java`.
- Unit tests extend `dev.ilionx.workshop.support.UnitTest` for Mockito-based tests without Spring/DB.
- Integration tests extend `dev.ilionx.workshop.support.IntegrationTest` for Spring + DB tests (extends `WebMvcConfigurator` and cleans DB state between tests).

## Frontend (client/)
- Dev server: `bun run dev` (or `npm run dev` if you install with npm).
- Type checks: `bun run check` (runs `svelte-kit sync` then `svelte-check`).
- API types: `bun run generate:api` reads `server/openapi.json` and writes `client/src/lib/types/api.d.ts`.

## OpenAPI sync (repo root)
- Use `scripts/openapi-sync.sh` to refresh `server/openapi.json` and regenerate frontend types.
- Script expects backend on port 8080 and uses Bun for type generation.

## Rules
- When updating functionality we first add a test so that the new feature has a failing test that conforms with the requirements.