# Products Launcher MS

> **Important:** This project is still under active development.

A **NestJS** microservices backend orchestrated with Docker Compose and connected via **NATS** message broker. This repository is a monorepo-launcher that composes multiple independent microservice repositories through Git submodules.

## Architecture

```
                    ┌─────────────────┐
                    │   HTTP Clients   │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │ client-gateway  │  (API Gateway – Port 3000)
                    │     (BFF)       │
                    └────────┬────────┘
                             │ NATS
              ┌──────────────┼──────────────┐
              │              │              │
     ┌────────▼───────┐ ┌───▼────────┐ ┌───▼──────────┐
     │  products-ms   │ │ orders-ms  │ │   nats-server │
     │  (Port 3001)   │ │ (Port 3002)│ │  (Message     │
     │  SQLite        │ │ PostgreSQL │ │   Broker)     │
     └────────────────┘ └─────┬──────┘ └──────────────┘
                              │
                     ┌────────▼────────┐
                     │    orders-db    │
                     │   PostgreSQL    │
                     └─────────────────┘
```

## Services

| Service | Role | Port | Database |
|---------|------|------|----------|
| **client-gateway** | API Gateway (BFF) | 3000 | — |
| **products-ms** | Product catalog CRUD | 3001 | SQLite |
| **orders-ms** | Order lifecycle management | 3002 | PostgreSQL |
| **nats-server** | Message broker | 4222 | — |
| **orders-db** | PostgreSQL instance | 5432 | — |

## Technologies

- **Runtime:** Node.js 24, TypeScript
- **Framework:** NestJS 11
- **Message Broker:** NATS
- **ORM:** Prisma 7
- **Databases:** SQLite, PostgreSQL 17
- **Package Manager:** pnpm
- **Containerization:** Docker & Docker Compose
- **Validation:** class-validator, class-transformer
- **Testing:** Jest, Supertest
- **Linting:** Biome

## Getting Started

### Prerequisites

- Node.js >= 20
- pnpm (`npm install -g pnpm`)
- Docker & Docker Compose

### Setup

```bash
# Clone with submodules
git clone https://github.com/dportilla/products-launcher-ms.git
cd products-launcher-ms
git submodule update --init --recursive

# Configure environment
cp .env.example .env

# Start all services
docker compose up --build
```

The API gateway will be available at `http://localhost:3000`.

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `CLIENT_GATEWAY_PORT` | `3000` | Host port for the API gateway |

Each submodule has its own `.env.example` with service-specific variables.

## API Endpoints

### Products

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/v1/products` | Create a product |
| `GET` | `/api/v1/products` | List products (paginated) |
| `GET` | `/api/v1/products/:id` | Get product by ID |
| `PATCH` | `/api/v1/products/:id` | Update a product |
| `DELETE` | `/api/v1/products/:id` | Soft-delete a product |

### Orders

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/v1/orders` | Create an order |
| `GET` | `/api/v1/orders` | List orders (paginated) |
| `GET` | `/api/v1/orders/:id` | Get order by UUID |
| `PATCH` | `/api/v1/orders/:id` | Change order status |

## Development

Each service is mounted as a volume with hot-reload via `pnpm start:dev`, so changes in the submodule directories are reflected immediately inside the container.

```bash
# Rebuild a single service
docker compose build products-ms

# View logs for a service
docker compose logs -f client-gateway
```

## Repository Structure

```
.
├── compose.yaml              # Docker Compose orchestration
├── .env.example              # Root environment template
├── .gitmodules               # Git submodule references
├── client-gateway-ms/        # API Gateway (submodule)
├── products-ms/              # Products microservice (submodule)
└── orders-ms/                # Orders microservice (submodule)
```
