# x402scan Configuration & Setup

## Quick Start

### Installation

```bash
pnpm install

# This automatically:
# - Generates Prisma clients (both DBs)
# - Builds facilitators package
# - Copies facilitator logos to public/
```

---

## Environment Configuration

### Required Environment Variables

Create `apps/scan/.env.local`:

```bash
cp apps/scan/.env.example apps/scan/.env.local
```

### Sync Service Environment

Create `sync/transfers/.env`:

```bash
# Database
DATABASE_URL="postgresql://..."  # Same as TRANSFERS_DB_URL

# Trigger.dev
TRIGGER_SECRET_KEY="tr_dev_..."

# Data providers
GOOGLE_APPLICATION_CREDENTIALS="/path/to/gcp-key.json"
BIGQUERY_PROJECT_ID="your-gcp-project"
BITQUERY_API_KEY="your-bitquery-key"

# CDP (for Base chain sync)
CDP_API_KEY_NAME="organizations/{org_id}/apiKeys/{key_id}"
CDP_API_KEY_ID="your-key-id"
CDP_PRIVATE_KEY="-----BEGIN EC PRIVATE KEY-----..."
```

### Proxy Service Environment

Create `apps/proxy/.env`:

```bash
cp apps/proxy/.env.example apps/proxy/.env
```

---

## Database Setup

### 1. Create Databases

Create two databases: `x402scan_main` and `x402scan_transfers`

### 2. Run Migrations

```bash
cd apps/scan

# Development - apply migrations
pnpm db:migrate:dev

# Production - deploy migrations
pnpm db:migrate:prod

# Generate Prisma clients (both DBs)
pnpm db:generate
```

### 3. Verify Setup

```bash
# Open Prisma Studio
pnpm --filter scan db:studio          # Main DB
pnpm --filter scan db:studio:transfers # Transfers DB
```

---

## Development

### Start All Services

```bash
# From root
pnpm dev
```

### Individual Services

```bash
# Scan app only
pnpm dev:scan

# Proxy service
cd apps/proxy && pnpm dev

# Sync service (Trigger.dev local)
pnpm dev:sync
```

### Common Commands

```bash
# Type checking
pnpm check:types

# Linting
pnpm lint

# Formatting
pnpm format

# Run all checks
pnpm check

# Database operations
pnpm --filter scan db:push          # Push schema changes
pnpm --filter scan db:migrate:dev   # Create migration
pnpm --filter scan db:studio        # View data
```
