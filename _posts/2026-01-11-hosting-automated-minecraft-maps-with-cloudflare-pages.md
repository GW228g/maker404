---
layout: post
title: "Hosting Automated Minecraft Maps with Cloudflare Pages"
date: 2026-01-11
categories: [minecraft, web]
tags: [cloudflare, pages, automation]
---

# Hosting Automated Minecraft Maps with Cloudflare Pages

Once interactive maps existed, the final problem was obvious:

> “How do I share this without running my own web server?”

I didn’t want:
- open ports
- home-hosted HTTPS
- manual uploads
- another service to babysit

I wanted a place where maps could simply *appear*.

---

## Why Cloudflare Pages Was the Right Fit

Cloudflare Pages checked every box:

- static hosting
- HTTPS by default
- global CDN
- CLI-based deploys
- zero maintenance

Most importantly, it integrates cleanly into automation.

---

## Deployment as a Single Step

Once the site exists in a directory, deployment is a single command:

```bash
wrangler pages deploy maker404-maps-site   --project-name maker404-maps
```

No git commits.
No servers.
No downtime.

---

## Headless Authentication for Automation

Because this runs unattended, authentication uses an API token loaded from disk — not a browser login.

That makes the process reliable under cron, launchd, or SSH.

---

## A Stable URL That Never Changes

The maps live at a stable URL:

```text
https://maps.example.com/
```

Every deploy updates the content behind that address — bookmarks and links never break.

---

## The End Result

At this point, the system does something quietly powerful:

- backups protect the world
- maps show its history
- deployment shares it instantly

And I don’t have to think about any of it.
