---
layout: post
title: "Hosting Automated Minecraft Maps with Cloudflare Pages"
date: 2026-01-11
categories: [minecraft, web]
tags: [cloudflare, pages, automation]
---

# Hosting Automated Minecraft Maps with Cloudflare Pages

Once interactive maps existed, the next challenge was sharing them safely.

I didn’t want to run a web server or expose my home network.

---

## Why Cloudflare Pages

- Static hosting
- HTTPS by default
- No server maintenance
- Automated deploys

---

## One-Command Deployment

```bash
wrangler pages deploy maker404-maps-site \
  --project-name maker404-maps
```

The URL stays the same. The content updates automatically.
