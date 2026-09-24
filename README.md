# BILO

A rental backend prototype that connects property discovery, tenant interest, landlord decisions and lease records. The product's initial focus is student housing in Costa Rica.

Built with **NestJS, TypeScript, Prisma and SQLite**. This repository contains the runnable prototype and a separate set of technical, product and business plans for its next stages.

[Español](./README.es.md) · [Run locally](#run-locally) · [Documentation](#documentation)

![BILO flow from property discovery to match decisions and lease records, with the prototype boundaries marked.](./docs/assets/overview.svg)

## What the prototype does

- **Discover properties:** browse active listings, filter by location, price and amenities, and receive preference-based recommendations.
- **Express interest:** record a like or superlike, then create a tenant–landlord match request.
- **Make a decision:** the landlord accepts or rejects the request. Acceptance attempts to create a shared REST conversation.
- **Track a lease:** the landlord can create a draft from a pending or active match; the service records lease status and initial deposit/rent payments.
- **Keep context:** persist messages, ratings, trust events, disputes, notifications and an audit trail.

JWT authentication, role and ownership checks, DTO validation and OpenAPI documentation support these flows. They remain prototype capabilities; the demo login is public and is not disabled by the production environment flag.

## Architecture and current limits

The API is a **modular monolith**. Domain controllers call application services; Prisma manages SQLite persistence. In-process events connect workflows to trust, notifications and audit modules.

| Current implementation | Boundary |
| --- | --- |
| SQLite through Prisma | Production PostgreSQL migration is documented, not implemented here. |
| Payment-provider interface | Only the simulated `stripe_mock` provider is available; no real charges. |
| AI-provider interface | Responses use local context and rules through a mock provider. |
| Prisma recommendation queries | The Neo4j adapter is a placeholder that throws `NotImplementedException`. |
| REST conversations and stored notifications | No WebSocket chat or external notification delivery. |
| JWT access/refresh tokens and demo login | No production security guarantee; `mock-login` accepts an email without credential proof. |

Multi-step lease creation is not a single database transaction. Designs in `docs/design/` describe intended hardening and later capabilities, not proof of current delivery.

## Run locally

Use a disposable local demo database. The Docker bootstrap runs `prisma db push --accept-data-loss`; it can change the schema and data. Do not point this setup at a database you need to preserve or expose the demo publicly.

### Docker Compose

Requires Docker with Compose.

```bash
cp .env.example .env
docker compose up --build
```

The service is exposed at `http://localhost:3001`; SQLite is mounted from `./data`. Before starting, set `SEED_MODE=basic` in `.env` for the built-in seed without external listing enrichment. With an empty property catalog, automatic seeding otherwise defaults to live enrichment with fallbacks; `AUTO_SEED=false` disables seeding.

- OpenAPI: `http://localhost:3001/api/v1/docs`
- Health: `http://localhost:3001/api/v1/health`
- Database health: `http://localhost:3001/api/v1/health/db`

### Node.js

Requires Node.js 20.

```bash
cp .env.example .env
npm install --legacy-peer-deps
npm run prisma:generate
npm run prisma:push
npm run prisma:seed
npm run start:dev
```

The local server uses `http://localhost:3000`. To explore protected routes without Google OAuth credentials, call `POST /api/v1/auth/mock-login` with an email and role, then use the returned access token in Swagger. Google OAuth requires real credentials and a callback URL matching the port in use.

## Build and verification

`npm run build` compiles TypeScript. There is currently **no automated test suite or test script** in `package.json`; the [testing strategy](./docs/design/11-testing-strategy.md) describes planned coverage, not passing tests.

Other development commands:

| Command | Purpose |
| --- | --- |
| `npm run prisma:studio` | Inspect the local Prisma data. |
| `npm run prisma:seed:live` | Try external listing enrichment, with fallback data. |
| `npm run seed:sqlite:inside-airbnb` | Build a separate exploratory SQLite catalog. |

The Inside Airbnb catalog does not automatically populate the application's Prisma tables. See its [seeding notes](./docs/sqlite-realistic-seeding.md).

## Team and contributions

BILO began as a hackathon team project. [Joshua Corrales](https://github.com/joshuacorraless) co-developed the prototype and contributed the subsequent architecture, requirements and product-planning documentation. [José Fabián Zumbado](https://github.com/JoseZum) contributed the initial backend implementation.

## Documentation

| Guide | Scope |
| --- | --- |
| [Technical design](./docs/design/README.md) | Target architecture, module contracts, data, security and roadmap. |
| [Requirements](./docs/requirements/README.md) | Functional requirements and delivery traceability. |
| [Product and business](./docs/business/README.md) | Initial market, operating assumptions and staged product scope. |
| [Costa Rica legal research](./docs/legal/costa-rica/README.md) | Research and questions for qualified counsel; not legal advice. |

## License

The package is marked `UNLICENSED`.
