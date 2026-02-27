# Cybersecurity Portfolio

Minimal, dark-themed multi-page portfolio built with [Astro](https://astro.build). Write posts in **Markdown** — they auto-appear on the blog/writeups pages.

## Quick Start

```bash
npm install
npm run dev        # → http://localhost:4321
npm run build      # → static output in dist/
npm run preview    # → preview the build
```

## Deploy to GitHub Pages

1. Push this repo to GitHub
2. Go to **Settings → Pages → Source** and select **GitHub Actions**
3. The included `.github/workflows/deploy.yml` handles the rest — every push to `main` auto-deploys

If your repo is **not** `yourusername.github.io`, uncomment the `base` line in `astro.config.mjs`:
```js
base: '/your-repo-name',
```

## File Structure

```
src/
├── components/
│   ├── Card.astro          # Reusable card component
│   └── Nav.astro           # Navigation bar
├── layouts/
│   ├── Base.astro          # Page shell (nav + footer)
│   └── Post.astro          # Markdown post layout
├── pages/
│   ├── index.astro         # Home page
│   ├── projects.astro      # Projects listing
│   ├── writeups.astro      # Auto-lists writeup posts
│   ├── blog.astro          # Auto-lists blog posts
│   └── posts/
│       ├── htb-example-machine.md
│       └── getting-started-malware-analysis.md
└── styles/
    └── global.css          # All styling (edit to retheme)
```

## Adding Content

### New Blog Post

Create `src/pages/posts/my-post.md`:

```md
---
layout: ../../layouts/Post.astro
title: "My Post Title"
description: "Short description for the card."
date: "2026-03-01"
type: blog
tags: ["tag1", "tag2"]
backLink: /blog
backLabel: Back to blog
---

Your markdown content here...
```

It automatically appears on the Blog page.

### New Writeup

Same as above, but set `type: writeup` and add a `platform` field:

```md
---
layout: ../../layouts/Post.astro
title: "HTB: Machine Name"
description: "Short description."
date: "2026-03-01"
type: writeup
platform: htb
tags: ["sqli", "linux"]
backLink: /writeups
backLabel: Back to writeups
---
```

### New Project

Edit `src/pages/projects.astro` and add a `<Card />` component.

## Customisation

### Colours & Fonts

Edit the CSS variables at the top of `src/styles/global.css`:

```css
--bg-primary:    #0a0a0f;   /* Main background */
--accent:        #00d4aa;   /* Accent colour */
--text-primary:  #e2e2ea;   /* Main text */
--font-body:     'Outfit', sans-serif;
--font-mono:     'JetBrains Mono', monospace;
```

### Your Info

Search and replace across the project:
- `Your Name` → your name
- `yourusername` → GitHub/LinkedIn username
- `you@email.com` → your email
- `[Your University]` → your uni

### Adding Nav Links

Edit `src/components/Nav.astro` — add entries to the `links` array.
