# Cybersecurity Portfolio

Minimal, dark-themed multi-page portfolio built with [Astro](https://astro.build). Write posts in **Markdown** — they auto-appear on the blog/writeups pages.

## Quick Start

```bash
npm install
npm run dev        # → http://localhost:4321
npm run build      # → static output in dist/
npm run preview    # → preview the build
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

