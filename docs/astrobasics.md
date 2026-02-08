# Astro Basics — Everything You Need to Proceed (Single-File Guide)

This is a practical beginner guide for **Astro** focused on building a **consultant-professional personal site** with:
- normal pages (`/about`, `/projects`, `/contact`)
- a Markdown blog
- a clean layout with navigation + footer
- deployment

It’s written so you can proceed without reading lots of docs.

---

## 1) What Astro is (in plain English)

Astro is a tool for building **fast websites**. You write:
- **pages** in `.astro` files
- **blog posts** in Markdown (`.md`)
- **components** (optional) to reuse UI
- **very little JavaScript** unless you need interactivity

Astro’s key idea: ship mostly **HTML + CSS** (fast + simple).

---

## 2) The mental model (3 things)

### A) Pages → URLs
Anything in `src/pages/` becomes a URL automatically.

Example mapping:

| File | URL |
|------|-----|
| `src/pages/index.astro` | `/` |
| `src/pages/about.astro` | `/about` |
| `src/pages/projects.astro` | `/projects` |
| `src/pages/contact.astro` | `/contact` |

---

### B) Layouts → shared wrapper (nav/footer)
A layout is a wrapper around pages that contains the common structure:
- HTML `<head>` (title, meta)
- navigation menu
- footer

Layouts live in `src/layouts/`.

---

### C) Content collections → blog posts with validation
Your Markdown blog posts live in:

```
src/content/blog/
```

And Astro validates the frontmatter using:

```
src/content/config.ts
```

This prevents broken posts and missing metadata.

---

## 3) Running Astro locally (development)

### Install dependencies (first time only)
```bash
npm install
```

### Start the dev server
```bash
npm run dev
```

Astro will print a local URL like:

```
http://localhost:4321
```

### Stop the server
Press:
`Ctrl + C`

---

## 4) Building and previewing production output

### Build a production version
```bash
npm run build
```

This generates a `dist/` folder (your final website output).

### Preview the production build locally
```bash
npm run preview
```

---

## 5) Your project folder structure (what matters)

A common structure:

```
my-site/
  src/
    pages/
      index.astro
      about.astro
      projects.astro
      contact.astro
      blog/               (optional folder for blog routes)
    layouts/
      Layout.astro
      BlogPost.astro
    content/
      config.ts
      blog/
        some-post.md
  public/
    images/
    favicon.svg
  package.json
  astro.config.mjs
```

**What each folder does:**
- `src/pages/` → site routes (URLs)
- `src/layouts/` → reusable wrappers
- `src/content/` → Markdown content + schemas
- `public/` → static files copied directly to the output

---

## 6) Pages: how `.astro` files work

An `.astro` file has two parts:

### Part A) Script/frontmatter (top section)
Between `---` and `---`:

```astro
---
import Layout from "../layouts/Layout.astro";
const pageTitle = "About";
---
```

This runs at build time / server-side rendering time.

### Part B) Template/HTML (below)
Normal HTML-like markup:

```astro
<Layout title={pageTitle}>
  <h1>About</h1>
</Layout>
```

---

## 7) Layouts: how to make a shared wrapper

A layout uses `<slot />` to insert page content.

Example: `src/layouts/Layout.astro`

```astro
---
const { title = "My Site", description = "" } = Astro.props;
---

<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>{title}</title>
    {description && <meta name="description" content={description} />}
  </head>

  <body>
    <header>
      <nav>
        <a href="/">Home</a>
        <a href="/projects">Projects</a>
        <a href="/blog">Blog</a>
        <a href="/about">About</a>
        <a href="/contact">Contact</a>
      </nav>
    </header>

    <main>
      <slot />
    </main>

    <footer>
      <small>© {new Date().getFullYear()}</small>
    </footer>
  </body>
</html>
```

Then every page can wrap itself:

```astro
---
import Layout from "../layouts/Layout.astro";
---

<Layout title="About">
  <h1>About</h1>
</Layout>
```

---

## 8) Styling in Astro (simple options)

### Option 1: Inline CSS in a page/layout (fastest)
Add this at the bottom of any `.astro` file:

```astro
<style>
  h1 { font-size: 2rem; }
</style>
```

### Option 2: Global styles (cleaner later)
Create a CSS file and import it in the layout.
But for your portfolio site, **inline layout CSS is fine**.

### Global selector in Astro
To style global elements like `body`, use:

```css
:global(body) {
  margin: 0;
}
```

---

## 9) Markdown blog posts (the correct format)

Your posts live in:

```
src/content/blog/
```

Example file: `src/content/blog/tank-dynamics-part-1.md`

```md
---
title: "Tank Dynamics Simulation (Part 1)"
description: "A first-principles walkthrough for simulating tank level dynamics."
pubDate: "2026-01-29"
---

## Overview
Write your post here.
```

### Important rules
- The frontmatter must start at the top of the file (no blank line above `---`)
- Frontmatter keys must match the schema exactly
- Use ISO dates: `"YYYY-MM-DD"` is safe

---

## 10) Your schema file (`src/content/config.ts`)

You showed this schema:

```ts
import { defineCollection, z } from 'astro:content';
import { glob } from 'astro/loaders';

const blog = defineCollection({
  loader: glob({ base: './src/content/blog', pattern: '**/*.{md,mdx}' }),
  schema: ({ image }) =>
    z.object({
      title: z.string(),
      description: z.string(),
      pubDate: z.coerce.date(),
      updatedDate: z.coerce.date().optional(),
      heroImage: image().optional(),
    }),
});

export const collections = { blog };
```

That means every blog post must include:

- `title` (string)
- `description` (string)
- `pubDate` (date or date-like string)

Optional:
- `updatedDate`
- `heroImage`

### Why you got errors earlier
Astro errors like:

- `title: Required`
- `description: Required`
- `pubDate: Invalid date`

happen when:
- the frontmatter is missing fields
- the YAML is broken (wrong `---`)
- the date is not parseable
- the filename or content includes odd characters
- the frontmatter keys are mistyped (case-sensitive)

---

## 11) Filenames and slugs (URLs)

A Markdown filename becomes the slug (URL path).

Example:

`src/content/blog/tank-dynamics-simulation-part-1.md`

→ URL will be:

`/blog/tank-dynamics-simulation-part-1`

### Use clean filenames
Good:
- `tank-dynamics-part-1.md`
- `hybrid-llm-workflow.md`

Avoid:
- spaces
- `=`
- `?`
- weird punctuation

---

## 12) Blog index and blog pages (how posts show up)

In Astro’s blog template, you usually get:
- a blog listing page (index)
- a dynamic page to render each post

Depending on the template, these are commonly:
- `src/pages/blog/index.astro`
- `src/pages/blog/[...slug].astro`

If you don’t have them, you can add them later.

---

## 13) Projects page (simple and professional)

For a consultant portfolio, a Projects page is easiest as a simple array:

```astro
---
import Layout from "../layouts/Layout.astro";

const projects = [
  {
    title: "Tank Dynamics Simulator",
    summary: "A first-principles dynamic model with interactive parameters and plots.",
    demo: "#",
    github: "#",
    writeup: "/blog/tank-dynamics-simulation-part-1",
  },
];
---

<Layout title="Projects">
  <main class="container">
    <h1>Projects & Apps</h1>

    <section class="grid">
      {projects.map((p) => (
        <article class="card">
          <h2>{p.title}</h2>
          <p>{p.summary}</p>
          <div class="links">
            <a href={p.writeup}>Write-up</a>
            <a href={p.demo}>Demo</a>
            <a href={p.github}>GitHub</a>
          </div>
        </article>
      ))}
    </section>
  </main>
</Layout>
```

This is clean, fast, and looks professional.

---

## 14) When you need JavaScript (and when you don’t)

Astro sites can be 100% static and still be excellent.

You only need JS if you want interactivity like:
- parameter sliders for simulations
- live plots
- filtering projects
- dynamic UI widgets

You can ignore this early on and add interactivity later.

---

## 15) Common errors and exact fixes

### Error: `FailedToLoadModuleSSR` / cannot import Layout
Cause: wrong import path OR file doesn’t exist.

Fix:
- confirm the file exists: `ls -la src/layouts`
- import the correct name:
  - `../layouts/Layout.astro`
  - or create `src/layouts/Layout.astro`

---

### Error: `InvalidContentEntryDataError` for blog posts
Cause: frontmatter doesn’t match schema.

Fix checklist:
- frontmatter starts at top of file
- `---` lines are correct
- keys exist: `title`, `description`, `pubDate`
- `pubDate` is parseable:
  - `"2026-01-29"` is safe
- filename uses normal characters (no `=`)

---

### Error: blog post not appearing
Fix:
- ensure it’s in `src/content/blog/`
- restart dev server (`Ctrl+C`, `npm run dev`)
- check schema fields

---

## 16) Deployment basics (quick)

### Netlify
- Connect GitHub repo
- Build command: `npm run build`
- Publish directory: `dist`

### Vercel
- Connect GitHub repo
- Framework: Astro (usually auto-detected)
- Build: `npm run build`
- Output: `dist`

### GitHub Pages
Works fine, but can require base path config.
Netlify/Vercel is easiest.

---

## 17) Minimum launch checklist (consultant-professional)

To launch your site quickly, aim for:

### Pages
- Home `/`
- Projects `/projects`
- Blog `/blog`
- About `/about`
- Contact `/contact`

### Content
- 3 blog posts (your prepared topics)
- 2–4 project entries (even if demos are “coming soon”)

### Design
- clean nav + footer
- consistent typography
- readable spacing
- simple buttons/links

---

## 18) My recommended workflow (Markdown-first)

1. Write posts in Markdown (`src/content/blog`)
2. Keep homepage concise (value + proof)
3. Keep Projects page structured (problem → approach → output)
4. Deploy early (Netlify/Vercel)
5. Improve gradually (don’t rebuild)

---

## 19) Cheatsheet

### Start dev server
```bash
npm run dev
```

### Add a new page
Create: `src/pages/newpage.astro`

### Add a new blog post
Create: `src/content/blog/my-post.md`

### Build site
```bash
npm run build
```

### Preview build
```bash
npm run preview
```

---

## 20) What you should do next (concrete next steps)

1. Ensure `src/layouts/Layout.astro` exists (site-wide layout)
2. Confirm `/` loads without import errors
3. Create the 3 blog Markdown files with correct frontmatter
4. Create `/projects` with 2 starter projects
5. Add real GitHub/LinkedIn links
6. Deploy to Netlify/Vercel

---

If you want a clean consultant-professional feel:
- keep content structured
- keep navigation simple
- prioritize “proof of work” (projects + posts)
- keep pages short and scannable
