# CLAUDE.md — rogerwibrew.com

## Project Overview

Astro static site for rogerwibrew.com — a professional portfolio focused on chemical engineering, process simulation, and AI-enabled engineering tools. Deployed to a Hostinger VPS via GitHub Actions (push to master triggers build + SCP to VPS).

## Stack

- **Framework:** Astro 5 with MDX and sitemap integrations
- **Hosting:** Hostinger VPS (Ubuntu), Traefik reverse proxy, nginx container
- **Deployment:** GitHub Actions → builds Astro → SCPs dist/ to VPS
- **Repo:** git@github.com-rwconsult:rwconsult8254/website.git (master branch)

## Site Status

The site has been cleaned up from the original Astro blog starter template. All template content has been removed and replaced with real content.

### What's in place
- **Config:** Site URL, title, and description are correct
- **Theme:** Unified dark theme across all pages (Layout.astro + BlogPost.astro)
- **Header/Footer:** Custom nav with Projects, Blog, About, Contact links; clean footer with copyright
- **Favicon:** Custom RW hexagon logo in `/public/favicon/` (SVG, ICO, PNG, apple-touch-icon, webmanifest)
- **OG card:** Social sharing image at `/public/og-image.svg` (needs PNG conversion for full platform support)
- **Blog:** One published article (DHSV detection). No stub/placeholder posts.
- **Projects:** Tank Dynamics Simulator with live demo at tank.rogerwibrew.com
- **Web apps:** Only Tank Simulation in WebAppsSection
- **Contact:** Real LinkedIn, GitHub, and email links
- **Comments:** Giscus integration on blog posts
- **RSS/Sitemap:** Both configured and working

### Branding
- **Accent colour:** `#3b82f6` (blue) — used in favicon, links, buttons, accents
- **Backgrounds:** `#111318` (primary), `#1a1d24` (secondary), `#21242c` (surface)
- **Fonts:** Inter (sans), JetBrains Mono (code) via Google Fonts

## Commands

```bash
npm run dev      # Local dev server
npm run build    # Production build
npm run preview  # Preview production build
git push         # Triggers deployment to VPS
```

## Notes

- Domain: rogerwibrew.com (DNS managed at registrar, pointing to 153.92.221.139)
- SSH to VPS: `ssh roger@153.92.221.139`
- GitHub Actions secrets: VPS_SSH_KEY, VPS_HOST, VPS_USER
- Two GitHub accounts configured via SSH: personal (id_ed25519) and rwconsult8254 (id_ed25519_rwconsult)
