# Adding New Facilitators

## Overview

Facilitators are x402 payment processors. Adding one involves:
1. Creating facilitator config file
2. Adding logo and metadata
3. Registering in central list
4. Optionally enabling blockchain sync

**Location:** `packages/facilitators/`

---

## Quick Guide

### Step 1: Add Logo

```bash
# Add square PNG/SVG to images/
cp logo.png packages/facilitators/images/newfacilitator.png
```

### Step 2: Create Configuration

**Simple facilitator (no auth):**

`packages/facilitators/src/facilitators/newfacilitator.ts`:
```typescript
import { Network } from '../types';
import { USDC_BASE_TOKEN } from '../constants';
import type { Facilitator, FacilitatorConfig } from '../types';

export const newfacilitator: FacilitatorConfig = {
  url: 'https://api.newfacilitator.com/x402',
};

export const newfacilitatorFacilitator = {
  id: 'newfacilitator',
  metadata: {
    name: 'New Facilitator',
    image: 'https://x402scan.com/newfacilitator.png',
    docsUrl: 'https://docs.newfacilitator.com',
    color: '#FF6B35',
  },
  config: newfacilitator,
  addresses: {
    [Network.BASE]: [
      {
        address: '0x1234...',
        tokens: [USDC_BASE_TOKEN],
        dateOfFirstTransaction: new Date('2025-01-15'),
      },
    ],
  },
} as const satisfies Facilitator<void>;
```

**With authentication:**
```typescript
type NewFacilitatorConfig = { apiKey: string };

export const newfacilitator: FacilitatorConfigConstructor<NewFacilitatorConfig> = ({
  apiKey,
}) => ({
  url: 'https://api.newfacilitator.com/x402',
  createAuthHeaders: async () => ({
    verify: { 'Authorization': `Bearer ${apiKey}` },
    settle: { 'Authorization': `Bearer ${apiKey}` },
  }),
});
```

### Step 3: Register Facilitator

**Export:** `packages/facilitators/src/facilitators/index.ts`
```typescript
export * from './newfacilitator';
```

**Add to list:** `packages/facilitators/src/lists/all.ts`
```typescript
import { newfacilitatorFacilitator } from '../facilitators';

const FACILITATORS = validateUniqueFacilitators([
  coinbaseFacilitator,
  // ... others
  newfacilitatorFacilitator, // Add here
]);
```

### Step 4: Build & Test

```bash
# Build package
cd packages/facilitators
pnpm build

# Update scan app
cd ../../apps/scan
pnpm postinstall

# Verify logo copied
ls public/newfacilitator.png

# Start dev server
pnpm dev
```

---

## Facilitator Types

### 1. Simple (No Auth)
```typescript
export const facilitator: FacilitatorConfig = {
  url: 'https://api.facilitator.com/x402',
};
```

### 2. With API Key
```typescript
type Config = { apiKey: string };

export const facilitator: FacilitatorConfigConstructor<Config> = ({
  apiKey,
}) => ({
  url: 'https://api.facilitator.com/x402',
  createAuthHeaders: async () => ({
    verify: { 'Authorization': `Bearer ${apiKey}` },
  }),
});
```

### 3. SDK-Based
```typescript
import { facilitator } from '@facilitator/x402';

export const facilitatorConfig: FacilitatorConfig = facilitator;
```

---

## Adding Network Support

For each network the facilitator supports:

```typescript
addresses: {
  [Network.BASE]: [
    {
      address: '0xYourAddress',
      tokens: [USDC_BASE_TOKEN],
      dateOfFirstTransaction: new Date('2025-01-15'),
    },
  ],
  [Network.SOLANA]: [
    {
      address: 'SolanaAddress123',
      tokens: [USDC_SOLANA_TOKEN],
      dateOfFirstTransaction: new Date('2025-01-20'),
    },
  ],
}
```

---

## Enabling Sync (Optional)

To sync blockchain transfers for your facilitator:

**In facilitator config, add:**
```typescript
addresses: {
  [Network.BASE]: [
    {
      address: '0x...',
      tokens: [USDC_BASE_TOKEN],
      dateOfFirstTransaction: new Date('2025-01-15'),
      enabled: true, // Enable sync
    },
  ],
}
```

Sync will automatically pick up facilitators from the `allFacilitators` list.

---

## Discovery Support (Optional)

If your facilitator supports listing resources:

**Add discovery config:**
```typescript
export const newfacilitatorDiscovery: FacilitatorConfig = {
  url: 'https://api.newfacilitator.com/x402',
};

export const newfacilitatorFacilitator = {
  // ... other config
  discoveryConfig: newfacilitatorDiscovery,
};
```

**Register in:** `packages/facilitators/src/discovery/facilitators.ts`
```typescript
export const discoverableFacilitators = [
  coinbaseDiscovery,
  newfacilitatorDiscovery, // Add here
];
```

---

## Token Constants

For new networks, add token constant:

`packages/facilitators/src/constants.ts`:
```typescript
export const USDC_ARBITRUM_TOKEN: Token = {
  address: '0xaf88d065e77c8cC2239327C5EDb3A432268e5831',
  decimals: 6,
  symbol: 'USDC',
};
```

---

## Validation

The `validateUniqueFacilitators` function ensures:
- No duplicate IDs
- No duplicate address/token pairs
- Compile-time type safety

If validation fails, you'll get clear error messages:
```
❌ COMPILE ERROR: Duplicate facilitator ID: 'newfacilitator'
❌ COMPILE ERROR: Duplicate address/token pair: 'id:chain:address:token'
```

---

## Troubleshooting

**Facilitator not showing:**
```bash
cd apps/scan
pnpm postinstall
pnpm dev
```

**Build errors:**
```bash
cd packages/facilitators
pnpm build
pnpm check:types
```

**Logo missing:**
```bash
ls packages/facilitators/images/newfacilitator.png
ls apps/scan/public/newfacilitator.png
```

---

## Checklist

- [ ] Logo added to `packages/facilitators/images/`
- [ ] Config file created
- [ ] Exported in `src/facilitators/index.ts`
- [ ] Added to `src/lists/all.ts`
- [ ] Package builds: `pnpm build`
- [ ] Logo appears in UI
- [ ] Discovery added (if supported)
- [ ] Sync enabled (if needed)

---

**Document Version:** 1.0
**Last Updated:** 2025-11-05
