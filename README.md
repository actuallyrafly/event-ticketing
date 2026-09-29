
Event Ticketing System
A production-oriented event ticketing application built to explore backend correctness, transaction safety, concurrency control, idempotency, observability, and performance at scale.

Project status
Currently in M0 — Project Skeleton.

Technology stack
TypeScript
React and Vite
NestJS with Express
PostgreSQL
Prisma
Redis
BullMQ
pnpm workspaces
Docker Compose
k6
Repository structure
apps/
├── web/       # React frontend
├── api/       # NestJS REST API
└── worker/    # Background-job processor

packages/
└── shared/    # Shared types, contracts, and utilities

docs/          # Detailed technical documentation
compose.yaml   # Local PostgreSQL and Redis services
Prerequisites
Install the following tools:

Node.js LTS
pnpm
Git
Docker Desktop with Docker Compose
Verify the installation:

node --version
pnpm --version
git --version
docker --version
docker compose version
Local setup
Clone the repository and install dependencies:

git clone <repository-url></repository>
cd event-ticketing-system
pnpm install
Copy the example environment file:

cp .env.example .env
On Windows PowerShell:

Copy-Item .env.example .env
Start PostgreSQL and Redis:

docker compose up -d
Confirm that the services are running:

docker compose ps
Development
Commands will be added as the web, API, and worker applications are initialized.

pnpm dev
pnpm build
pnpm lint
pnpm test
Database
Database commands will be finalized after Prisma is initialized.

pnpm db:migrate
pnpm db:seed
pnpm db:studio
Testing
The project will include:

Unit tests
Integration tests
End-to-end tests
Concurrency tests
Load tests with k6
Documentation
Detailed documentation lives in docs/.

Planned documents include:

Architecture
Database design
API conventions
Testing strategy
Load-testing reports
Architecture Decision Records
