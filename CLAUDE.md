# Datafund Provenance Landing

Landing page for [provenance.datafund.io](https://provenance.datafund.io)

## Overview

Standalone Astro + Tailwind landing page for the Datafund Provenance product.

## Tech Stack

- Framework: Astro
- Styling: Tailwind CSS
- Build: npm

## Development

```bash
npm install
npm run dev       # http://localhost:4321
npm run build     # astro check + astro build → dist/
npm run preview
```

## Deployment

**CI/CD:** Push to `main` → GitHub Actions builds and rsyncs `dist/` to campaigns server

**Manual (emergency):**
```bash
npm run build  # or: npx astro build
~/.datacore/modules/campaigns/scripts/deploy-site.sh provenance.datafund.io
```

## Provenance Ecosystem Repos

The landing page describes the Datafund Provenance Toolkit. Sibling repos live one level up at `../` and are the source of truth for package features, versions, and documentation:

| Repo | Type | Language | Package | Description |
|------|------|----------|---------|-------------|
| `swarm_provenance_SDK/` | SDK | TypeScript | `@datafund/swarm-provenance` | High-level upload/download, notary signing, blockchain anchoring |
| `swarm_provenance_CLI/` | CLI | Python | `swarm-provenance-uploader` | Full-featured CLI with stamps, collections, x402 payments, GDPR deletion |
| `swarm_provenance_mcp/` | MCP Server | Python | `swarm-provenance-mcp` | AI agent interface (Claude Desktop integration) |
| `ConsentsBasedDataProvenance/` | Smart Contracts | Solidity | `consents-based-data-provenance` | 9 contracts: consent, provenance, access control, GDPR deletion |
| `provenance-smasher/` | Test Suite | Python | - | Multi-layer QA: gateway, SDK, CLI, MCP, fuzz, load, security |
| `provenance-fellowship/` | Docs/PM | Markdown | - | Swarm Fellowship program coordination and milestones |
| `swarm_connect/` | Gateway | Python | - | FastAPI gateway bridging clients to Swarm network |

When updating landing page content (features, install commands, code snippets, version numbers), read the relevant sibling repo's README and source for accurate information.

## Related

- GitHub: https://github.com/datafund/provenance-landing
- Server: `deploy@$DO_DROPLET_IP:/var/www/sites/provenance.datafund.io/`
- Parent space: `../../` (1-datafund)
