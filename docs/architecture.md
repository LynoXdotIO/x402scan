# x402scan Architecture Overview

## System Overview

x402scan is an ecosystem explorer for the x402 protocol (digital payments standard), built as a **pnpm monorepo** with three main workspaces:

```
x402scan/
├── apps/scan/         # Next.js web application
├── apps/proxy/        # HTTP proxy service
├── packages/facilitators/  # Shared facilitator configs
└── sync/transfers/    # Background sync (Trigger.dev)
```

### High-Level Architecture

```
                     Users/Clients
                          │
                          ▼
              ┌───────────────────────┐
              │   Next.js App (scan)  │
              │   - Frontend + tRPC   │
              │   - Auth + AI Agent   │
              └──────┬────────┬───────┘
                     │        │
          ┌──────────▼──┐  ┌─▼────────┐
          │  Postgres   │  │  Redis   │
          │  (2 DBs)    │  │  Cache   │
          └──────▲──────┘  └──────────┘
                 │
        ┌────────┴─────────────┐
        │  Background Sync     │
        │  (Trigger.dev)       │
        └──────────┬───────────┘
                   │
        ┌──────────▼─────────────┐
        │  Blockchain Networks   │
        │  Base/Solana/Polygon   │
        └────────────────────────┘
```

---

## Workspaces

### apps/scan - Main Web Application

**Tech Stack:** Next.js 15, React 19, Turbopack, tRPC, Prisma, NextAuth, Tailwind v4

**Purpose:**
- User-facing web interface
- Type-safe API (tRPC)
- Authentication & user management
- AI agent chat interface
- Resource discovery

**Key Services:** (`apps/scan/src/services/`)
- `agent/` - AI-powered chat with tool calling
- `cdp/` - Coinbase wallet integration
- `db/` - Database operations (organized by domain)
- `facilitator/` - Facilitator utilities
- `labeling/` - Auto-tagging resources
- `scraper/` - Discover x402 resources from URLs
- `transfers/` - Transfer analytics

### apps/proxy - HTTP Proxy

**Tech Stack:** Hono, ClickHouse

**Purpose:**
- Proxy x402 HTTP requests
- Log analytics to ClickHouse
- Enable CORS for browser clients

### packages/facilitators - Shared Config

**Purpose:** Centralized facilitator configurations (Coinbase, Thirdweb, etc.)

**Structure:**
```typescript
interface Facilitator {
  id: string;
  metadata: { name, image, docsUrl, color };
  config: FacilitatorConfig; // x402 SDK config
  addresses: {
    base?: Address[];
    solana?: Address[];
    polygon?: Address[];
  };
}
```

### sync/transfers - Background Jobs

**Tech Stack:** Trigger.dev v4, Prisma, CDP SDK, BigQuery, BitQuery

**Purpose:**
- Scheduled jobs to sync blockchain transfers
- Index transactions from facilitator addresses
- Store in separate transfers database

---

## Core Data Flows

### 1. Resource Discovery
```
User → tRPC → scraper service → parse x402 headers →
store in DB → auto-tag → return to UI
```

### 2. Transfer Sync
```
Cron trigger → query blockchain API (CDP/BigQuery/BitQuery) →
transform data → batch insert to DB → log completion
```

### 3. AI Agent
```
User message → load config → generate tools from resources →
stream AI response → execute x402 payments → return results
```

### 4. Authentication
```
User login → NextAuth → create session → store in Prisma →
issue session token → validate on requests → inject into tRPC context
```

---

## Database Architecture

### Two Separate PostgreSQL Databases

**1. Main DB** (`prisma/schema.prisma`) - Application data
- Users, Sessions, Accounts
- Resources (x402 resources with Accept-402 configs)
- Chats, Messages, AgentConfigurations
- Tags, ServerWallets, OnrampSessions

**2. Transfers DB** (`prisma-transfers/schema.prisma`) - Blockchain data
```prisma
model TransferEvent {
  id, address, sender, recipient, amount
  block_timestamp, tx_hash, chain, provider
  facilitator_id, decimals
}
```

**Scaling:** Supports read replicas via `@prisma/extension-read-replicas`

---

## Key Features

1. **x402 Resource Management** - Discover, store, and invoke x402 resources
2. **Blockchain Transfer Tracking** - Index transfers across Base/Solana/Polygon
3. **AI Agent (Composer)** - Chat interface for x402 interactions with tool calling
4. **Analytics** - Transfer volume, network stats, developer insights
5. **Developer Tools** - API explorer, resource testing

---

## Technology Stack

### Frontend
- Next.js 15 (App Router), React 19, Turbopack
- Tailwind CSS v4, Radix UI
- TanStack Query, React Hook Form

### Backend
- tRPC v11 (type-safe API)
- Prisma (PostgreSQL ORM)
- NextAuth 5 (authentication)
- Redis (caching)
- Trigger.dev v4 (background jobs)

### Web3/Blockchain
- CDP SDK (Coinbase)
- x402 SDK & x402-fetch
- Solana Web3.js, Viem, Wagmi

### AI/ML
- Vercel AI SDK, OpenAI
- Laminar AI (observability)

### Data Sources
- CDP API (Base transfers)
- Google BigQuery (blockchain data)
- BitQuery (Solana/Polygon)
- ClickHouse (proxy analytics)

---

## Communication Patterns

### Internal
- **Monorepo:** Workspaces share `facilitators` package
- **tRPC:** Type-safe client-server communication
- **Redis:** Caching layer between app and DB
- **Prisma:** Database access with read replicas

### External
- **Blockchain APIs:** CDP, BigQuery, BitQuery for transfer data
- **x402 Resources:** HTTP requests via proxy service
- **AI Services:** OpenAI API for chat, Laminar for observability

---

## Security & Scalability

### Security
- HTTP-only session cookies
- Role-based access control (admin/user)
- Environment variable validation (`@t3-oss/env-nextjs`)
- Parameterized queries (Prisma)

### Scalability
- Stateless Next.js (horizontal scaling)
- Read replicas (distribute DB load)
- Redis caching (reduce queries)
- Background jobs (offload heavy ops)
- Turbopack (fast builds)

---

## Deployment

- **Scan App:** Vercel (auto-deploy on push)
- **Sync Service:** Trigger.dev
- **Proxy:** Docker container
- **Databases:** Neon PostgreSQL with pooling
- **Cache:** Redis Cloud

---

**Document Version:** 1.0
**Last Updated:** 2025-11-05
