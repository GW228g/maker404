---
layout: post
title: "Hosting Automated Minecraft Maps with Cloudflare Pages"
date: 2026-01-11
categories: [minecraft, web]
tags: [cloudflare, pages, automation, wrangler]
---

# Hosting Automated Minecraft Maps with Cloudflare Pages

After generating interactive HTML maps, the next question was obvious:

> “How do I share this without running a web server?”

I didn’t want to:
- host nginx
- expose ports on my home network
- deal with SSL renewals
- manually copy files

I wanted:

> Push files → get a stable HTTPS URL.

This post covers how I used **Cloudflare Pages + Wrangler** to publish maps automatically.

---

## Why Cloudflare Pages

Cloudflare Pages is a great fit for generated static sites because:

- it’s fast (CDN everywhere)
- HTTPS is automatic
- it’s low maintenance
- deploys are one CLI command
- it’s free for this kind of usage

---

## Deploying with Wrangler

Once the map site is built in a folder, deployment is one command:

```bash
wrangler pages deploy "/Users/jamescrawford/maker404-maps-site"   --project-name "maker404-maps"
```

No web server. No SSL work. No downtime.

---

## Headless Authentication

Because this runs on a remote Mac mini, I authenticate using an API token loaded from a file:

```bash
source "$HOME/.config/maker404/cloudflare.env"
```

That avoids interactive browser logins and works under launchd/cron/SSH.

---

## Stable Custom Domain

The end result is a stable URL:

```text
https://maps.maker404.com/
```

Each deploy updates content behind that domain. Bookmarks never break.

---

## Bringing It All Together

The automation chain now looks like:

1. Backup worlds into a timestamped ZIP
2. Restart server safely
3. Render maps from the latest ZIP
4. Deploy maps to Cloudflare Pages
5. Send one Discord summary with the map link

The system runs quietly, and the world’s history is always viewable in a browser.
