# x402scan Documentation

Welcome to the x402scan documentation! This directory contains comprehensive guides for understanding, configuring, and extending the x402scan ecosystem explorer.

## Documentation Index

### 📚 [Architecture Overview](./architecture.md)
**Comprehensive guide to x402scan's system design**

Learn about:
- System architecture and component relationships
- Workspace structure (scan, sync, facilitators, proxy)
- Core modules and services
- Data flows and communication patterns
- Database architecture (main + transfers)
- Technology stack
- Security and scalability considerations

**Best for:** Developers wanting to understand how x402scan works internally, architects planning integrations, or contributors getting started.

---

### ⚙️ [Configuration & Setup](./configuration.md)
**Complete setup guide from zero to running**

Covers:
- Prerequisites and required tools
- Environment variable configuration
- Database setup (Neon PostgreSQL)
- Development workflow
- Production deployment (Vercel, Trigger.dev, Docker)
- Troubleshooting common issues

**Best for:** New developers setting up the project locally, DevOps engineers deploying to production, or anyone troubleshooting setup issues.

---

### 🔌 [Adding New Facilitators](./adding-facilitators.md)
**Step-by-step guide to integrate new x402 facilitators**

Includes:
- Creating facilitator configurations
- Adding facilitator metadata and logos
- Supporting different authentication methods
- Enabling blockchain sync for facilitators
- Testing and validation
- Real-world examples (simple, API key, SDK-based)

**Best for:** Developers adding support for new payment facilitators, facilitator providers integrating with x402scan, or contributors expanding the ecosystem.

---

### 🌐 [Adding New Networks](./adding-networks.md)
**Complete guide to adding blockchain network support**

Explains:
- Adding networks to type definitions
- Database schema updates
- Data provider integration (CDP, BigQuery, BitQuery)
- Creating sync jobs for transfer indexing
- Testing new network implementations
- Examples for EVM and non-EVM chains

**Best for:** Developers adding new blockchain networks, contributors expanding multi-chain support, or teams implementing custom chain integrations.

---

## Quick Start

### For New Contributors

1. **Start here:** [Architecture Overview](./architecture.md)
   - Understand the system design
   - Learn about key components
   - Familiarize yourself with data flows

2. **Then setup:** [Configuration & Setup](./configuration.md)
   - Install dependencies
   - Configure environment
   - Start development servers

3. **Make changes:** Depending on your goal:
   - Adding a facilitator? → [Adding New Facilitators](./adding-facilitators.md)
   - Adding a network? → [Adding New Networks](./adding-networks.md)

### For Integrators

1. **Facilitator providers:**
   - Read: [Adding New Facilitators](./adding-facilitators.md)
   - Reference: [Architecture Overview](./architecture.md) (Facilitators section)

2. **Network/Chain teams:**
   - Read: [Adding New Networks](./adding-networks.md)
   - Reference: [Architecture Overview](./architecture.md) (Sync Flow section)

### For DevOps/SRE

1. **Start here:** [Configuration & Setup](./configuration.md)
   - Focus on "Production Deployment" section
   - Review "Troubleshooting" section

2. **Then review:** [Architecture Overview](./architecture.md)
   - Understand infrastructure components
   - Learn about scaling considerations
   - Review monitoring & observability

---

## Documentation Structure

```
docs/
├── README.md                    # This file - documentation index
├── architecture.md              # System design & architecture
├── configuration.md             # Setup & configuration guide
├── adding-facilitators.md       # Guide to add facilitators
└── adding-networks.md           # Guide to add networks
```

---

## Key Concepts

### x402 Protocol
x402 is a standard for digital payments that enables resources to accept payments via HTTP headers. Facilitators process these payments, and x402scan indexes and visualizes the ecosystem.

### Facilitators
Payment processors that implement the x402 protocol. Examples: Coinbase, Thirdweb, PayAI. Each facilitator has addresses on various blockchain networks.

### Networks
Blockchain networks supported by x402scan:
- **Base** - Layer 2 on Ethereum
- **Solana** - High-performance blockchain
- **Polygon** - Ethereum sidechain

### Workspaces
x402scan is a monorepo with four main workspaces:
- **apps/scan** - Next.js web application (main frontend)
- **apps/proxy** - HTTP proxy for x402 requests
- **packages/facilitators** - Shared facilitator configurations
- **sync/transfers** - Background sync service (Trigger.dev)

---

## Additional Resources

### Official Links
- **Repository:** [github.com/merit-systems/x402scan](https://github.com/merit-systems/x402scan)
- **Website:** [x402scan.com](https://x402scan.com)
- **x402 Spec:** [github.com/coinbase/x402](https://github.com/coinbase/x402)
- **Facilitators Package:** [npmjs.com/package/facilitators](https://npmjs.com/package/facilitators)

### Developer Resources
- **CDP Docs:** [docs.cdp.coinbase.com](https://docs.cdp.coinbase.com)
- **Trigger.dev Docs:** [trigger.dev/docs](https://trigger.dev/docs)
- **Next.js Docs:** [nextjs.org/docs](https://nextjs.org/docs)
- **Prisma Docs:** [prisma.io/docs](https://prisma.io/docs)

### Community
- **GitHub Issues:** Report bugs or request features
- **Discussions:** Ask questions, share ideas
- **Pull Requests:** Contribute code improvements

---

## Contributing to Documentation

Found an issue or want to improve the docs?

1. **File an Issue:**
   - Unclear sections
   - Missing information
   - Outdated content

2. **Submit a PR:**
   - Fix typos or errors
   - Add missing examples
   - Improve explanations

### Documentation Standards

- **Clear Structure:** Use headings, lists, and tables
- **Code Examples:** Include working, tested examples
- **Step-by-Step:** Break complex tasks into clear steps
- **Troubleshooting:** Include common issues and solutions
- **Keep Updated:** Document version and last updated date

---

## FAQ

### Q: Where do I start if I'm completely new?

**A:** Start with the [Architecture Overview](./architecture.md) to understand the system, then follow the [Configuration & Setup](./configuration.md) to get your local environment running.

### Q: How do I add my facilitator to x402scan?

**A:** Follow the complete guide in [Adding New Facilitators](./adding-facilitators.md). You'll need your facilitator's API endpoint, logo, and blockchain addresses.

### Q: Can x402scan support my blockchain network?

**A:** Probably! Check [Adding New Networks](./adding-networks.md) to see the requirements. You'll need access to a blockchain data provider (CDP, BigQuery, or BitQuery).

### Q: How do I deploy x402scan to production?

**A:** See the "Production Deployment" section in [Configuration & Setup](./configuration.md). The main app deploys to Vercel, and the sync service to Trigger.dev.

### Q: Where can I find the API documentation?

**A:** x402scan uses tRPC for type-safe APIs. API routes are defined in `apps/scan/src/trpc/routers/`. The [Architecture Overview](./architecture.md) explains the tRPC structure.

### Q: How often does transfer data sync?

**A:** Sync frequency varies by network and provider:
- Base (CDP): Every 5 minutes
- Solana (BigQuery): Hourly
- Polygon (BitQuery): Every 30 minutes

Configure in sync service configs. See [Adding New Networks](./adding-networks.md) for details.

---

## Document Versions

All documentation is versioned and includes "Last Updated" dates. If you notice outdated information, please file an issue or submit a PR.

- **Current Version:** 1.0
- **Last Updated:** 2025-11-05
- **Covers:** x402scan as of commit `f86561b`

---

## Need Help?

- **Setup Issues:** Check [Configuration & Setup](./configuration.md) → Troubleshooting
- **Architecture Questions:** Review [Architecture Overview](./architecture.md)
- **Integration Help:** See specific guides ([Facilitators](./adding-facilitators.md) / [Networks](./adding-networks.md))
- **Still Stuck:** Open a GitHub issue with:
  - What you're trying to do
  - Steps you've taken
  - Error messages
  - Environment details

---

Happy coding! 🚀
