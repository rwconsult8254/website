# CLAUDE.md — rogerwibrew.com

## Project Overview

Astro static site for rogerwibrew.com — a professional portfolio focused on chemical engineering, process simulation, and AI-enabled engineering tools. Deployed to a Hostinger VPS via GitHub Actions (push to master triggers build + SCP to VPS).

## Stack

- **Framework:** Astro 5 with MDX and sitemap integrations
- **Hosting:** Hostinger VPS (Ubuntu), Traefik reverse proxy, nginx container
- **Deployment:** GitHub Actions → builds Astro → SCPs dist/ to VPS
- **Repo:** git@github.com-rwconsult:rwconsult8254/website.git (master branch)

## Site Cleanup Plan

The site was scaffolded from the Astro blog starter template. Custom pages exist (index, about, contact, projects, Layout.astro) but all content is placeholder/stub. The following needs doing to make the site presentable:

### 1. Fix site config
- `astro.config.mjs`: Change `site` from `https://example.com` to `https://rogerwibrew.com`
- `src/consts.ts`: Update `SITE_TITLE` and `SITE_DESCRIPTION` to reflect Roger's site

### 2. Delete template blog posts
Remove all Astro starter filler:
- `src/content/blog/first-post.md`
- `src/content/blog/second-post.md`
- `src/content/blog/third-post.md`
- `src/content/blog/using-mdx.mdx`
- `src/content/blog/markdown-style-guide.md`

Keep the real post stubs (titles and frontmatter are good, content not yet written):
- `hybrid-llm-workflow.md`
- `tank-dynamics-simulation-part-1.md`
- `thermodynamics-behind-simple-process-simulation-part-1.md`

These should show on the blog page with a "Coming soon" indication rather than "(Write your post here.)"

### 3. Fix Header and Footer components
- `src/components/Header.astro`: Remove or replace — still has Astro template nav linking to Astro's Mastodon/Twitter/GitHub
- `src/components/Footer.astro`: Says "Your name here" with Astro's social links — remove or replace
- Wire `BlogPost.astro` layout to use `Layout.astro` so blog posts match the site's dark theme

### 4. Remove non-existent web apps
Only the Tank Dynamics simulator is planned. Remove:
- `src/pages/apps/amine.astro`
- `src/pages/apps/nrtl-vle.astro`
- `src/pages/apps/tennessee-eastman.astro`
- The corresponding entries from `src/components/WebAppsSection.astro`

Keep only:
- `src/pages/apps/tank.astro` (already says "coming soon")

### 5. Fix placeholder links
- `src/pages/contact.astro`: LinkedIn and GitHub links point to `#` — remove until real URLs are available
- `src/pages/projects.astro`: Remove the thermodynamics project (not started). Keep tank dynamics but remove "Demo" and "GitHub" `#` links — keep only the write-up link

### 6. Clean up assets
- Remove unused `blog-placeholder-*.jpg` images from `src/assets/`
- Update the fallback OG image in `BaseHead.astro` (currently uses a placeholder)

### 7. Consolidate layouts
- Two layout systems exist: `Layout.astro` (custom dark theme) and `BlogPost.astro` (uses old BaseHead/Header/Footer with light theme)
- Unify everything under the dark theme

### 8. Styling
- Find and apply a style that suits the brand: traditional chemical engineering meets modern AI
- Clean, professional, technical feel

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
