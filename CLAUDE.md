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

## Related

- GitHub: https://github.com/datafund/provenance-landing
- Server: `deploy@$DO_DROPLET_IP:/var/www/sites/provenance.datafund.io/`
- Parent space: `/Users/gregor/Data/1-datafund/`
