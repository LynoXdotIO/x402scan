# Adding New Networks

## Overview

x402scan currently supports: **Base**, **Polygon**, **Solana**

Adding a network involves:
1. Update type definitions
2. Update database schema
3. Configure data provider (CDP/BigQuery/BitQuery)
4. Create sync job
5. Update facilitator addresses

---

## Quick Guide

### Step 1: Add Network Type

**`packages/facilitators/src/types.ts`:**
```typescript
export enum Network {
  BASE = 'base',
  POLYGON = 'polygon',
  SOLANA = 'solana',
  ARBITRUM = 'arbitrum', // Add network
}
```

**`packages/facilitators/src/constants.ts`:**
```typescript
export const USDC_ARBITRUM_TOKEN: Token = {
  address: '0xaf88d065e77c8cC2239327C5EDb3A432268e5831',
  decimals: 6,
  symbol: 'USDC',
};
```

### Step 2: Update Database Schema

**`apps/scan/prisma/schema.prisma`:**
```prisma
enum AcceptsNetwork {
  base
  polygon
  solana
  arbitrum  // Add network
}
```

**Run migration:**
```bash
cd apps/scan
pnpm db:migrate:dev --name add_arbitrum
pnpm db:generate
```

### Step 3: Create Data Provider Config

Choose provider: **CDP** (EVM), **BigQuery** (historical), or **BitQuery** (GraphQL)

**For CDP (EVM chains):**

`sync/transfers/trigger/chains/evm/arbitrum/cdp/config.ts`:
```typescript
import { SyncConfig, PaginationStrategy, QueryProvider, Network } from '@/trigger/types';
import { FACILITATORS_BY_CHAIN } from '@/trigger/lib/facilitators';

export const arbitrumCdpConfig: SyncConfig = {
  cron: '*/5 * * * *',              // Every 5 mins
  maxDurationInSeconds: 600,
  chain: 'arbitrum',
  provider: QueryProvider.CDP,
  apiUrl: 'api.cdp.coinbase.com',
  paginationStrategy: PaginationStrategy.OFFSET,
  limit: 100_000,
  facilitators: FACILITATORS_BY_CHAIN(Network.ARBITRUM),
  buildQuery,                       // Implement query builder
  transformResponse,                // Implement response transformer
  enabled: true,
  machine: 'large-2x',
};
```

`sync/transfers/trigger/chains/evm/arbitrum/cdp/query.ts`:
```typescript
export function buildQuery(config, facilitatorConfig, since, now, offset = 0) {
  return JSON.stringify({
    network_id: 'arbitrum-mainnet',
    asset_id: facilitatorConfig.token.address,
    from_address: facilitatorConfig.address.toLowerCase(),
    start_time: since.toISOString(),
    end_time: now.toISOString(),
    limit: config.limit,
    page: offset.toString(),
  });
}

export async function transformResponse(data, config, facilitator, facilitatorConfig) {
  const transfers = data.data?.transfers ?? [];
  return transfers.map(t => ({
    address: t.contract_address.toLowerCase(),
    transaction_from: t.transaction_from.toLowerCase(),
    sender: t.sender.toLowerCase(),
    recipient: t.to_address.toLowerCase(),
    amount: parseFloat(t.amount),
    block_timestamp: new Date(t.block_timestamp),
    tx_hash: t.transaction_hash.toLowerCase(),
    chain: config.chain,
    provider: config.provider,
    decimals: facilitatorConfig.token.decimals,
    facilitator_id: facilitator.id,
  }));
}
```

### Step 4: Register Sync Task

`sync/transfers/trigger/chains/evm/arbitrum/index.ts`:
```typescript
import { createChainSyncTask } from '@/trigger/sync';
import { arbitrumCdpConfig } from './cdp/config';

export const arbitrumCdpSync = createChainSyncTask(arbitrumCdpConfig);
```

`sync/transfers/trigger/index.ts`:
```typescript
export * from './chains/evm/arbitrum'; // Add export
```

### Step 5: Update Facilitator Addresses

For each facilitator supporting the network:

`packages/facilitators/src/facilitators/coinbase.ts` (example):
```typescript
addresses: {
  [Network.BASE]: [ /* ... */ ],
  [Network.ARBITRUM]: [
    {
      address: '0x...',
      tokens: [USDC_ARBITRUM_TOKEN],
      dateOfFirstTransaction: new Date('2025-01-15'),
    },
  ],
}
```

### Step 6: Deploy

```bash
# Rebuild facilitators
cd packages/facilitators && pnpm build

# Update scan app
cd ../../apps/scan && pnpm build

# Deploy sync
cd ../../sync/transfers && pnpm trigger:deploy
```

---

## Data Providers

### CDP (Best for EVM)
- Real-time data
- High reliability
- Base, Arbitrum, Polygon support

### BigQuery
- Historical data
- Cost per query
- Many chains available

### BitQuery
- GraphQL API
- Flexible queries
- Good for Solana

---

## Provider Examples

### CDP Example (Optimism)
```typescript
export const optimismCdpConfig: SyncConfig = {
  cron: '*/5 * * * *',
  chain: 'optimism',
  provider: QueryProvider.CDP,
  apiUrl: 'api.cdp.coinbase.com',
  paginationStrategy: PaginationStrategy.OFFSET,
  // ... rest of config
};
```

### BigQuery Example (Avalanche)
```typescript
export const avalancheBigQueryConfig: SyncConfig = {
  cron: '0 */2 * * *',              // Every 2 hours
  chain: 'avalanche',
  provider: QueryProvider.BIGQUERY,
  paginationStrategy: PaginationStrategy.TIME_WINDOW,
  timeWindowInMs: ONE_DAY_IN_MS * 7, // 7-day windows
  // ... rest of config
};

// Query builder
export function buildQuery(...) {
  return `
    SELECT ...
    FROM \`bigquery-public-data.crypto_avalanche.token_transfers\`
    WHERE ...
  `;
}
```

### BitQuery Example (Polygon)
```typescript
export const polygonBitQueryConfig: SyncConfig = {
  cron: '0 * * * *',
  chain: 'polygon',
  provider: QueryProvider.BITQUERY,
  paginationStrategy: PaginationStrategy.OFFSET,
  // ... rest of config
};

// GraphQL query
export function buildQuery(...) {
  return `
    query ($network: evm_network!, ...) {
      EVM(network: $network) {
        Transfers(...) { ... }
      }
    }
  `;
}
```

---

## Sync Configuration

### Cron Schedules

| Volume | Recommended Schedule |
|--------|---------------------|
| High   | `*/5 * * * *` (every 5 min) |
| Medium | `*/30 * * * *` (every 30 min) |
| Low    | `0 * * * *` (hourly) |

### Machine Sizing

| Transfers/Run | Machine Size |
|---------------|-------------|
| < 1k          | `small-1x` |
| 1k - 10k      | `medium-1x` |
| 10k - 100k    | `large-2x` |

---

## Testing

### Local Testing
```bash
cd sync/transfers
pnpm trigger:dev

# Check logs for your network sync task
```

### Verify Data
```bash
cd apps/scan
pnpm db:studio:transfers

# Filter by chain = 'arbitrum'
```

### Monitor
- Trigger.dev dashboard: https://cloud.trigger.dev
- Check execution logs
- Verify transfer counts

---

## Troubleshooting

**Sync not running:**
- Check `enabled: true` in config
- Verify export in `trigger/index.ts`
- Redeploy: `pnpm trigger:deploy`

**No transfers syncing:**
- Verify facilitator addresses are correct
- Check `dateOfFirstTransaction` isn't future
- Test query in provider's console

**Query timeout:**
- Reduce time window
- Increase machine size
- Increase `maxDurationInSeconds`

---

## Checklist

- [ ] Network added to `Network` enum
- [ ] Token constant created
- [ ] Database schema updated
- [ ] Migration run
- [ ] Data provider config created
- [ ] Sync task registered
- [ ] Facilitator addresses updated
- [ ] Package rebuilt
- [ ] Types validated
- [ ] Local testing completed
- [ ] Sync deployed
- [ ] Transfers appearing in DB

---

**Document Version:** 1.0
**Last Updated:** 2025-11-05
