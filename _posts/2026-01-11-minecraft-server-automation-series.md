---
layout: post
title: "Minecraft Server Automation Series"
date: 2026-01-11
categories: [minecraft, homelab, automation]
tags: [minecraft, automation, series]
---

# Minecraft Server Automation Series: Backups, Notifications, and Live Maps

This is a four-part series documenting how I turned a family Minecraft server into something I don’t have to babysit:

- automated backups
- safe restarts
- high-signal Discord notifications
- interactive maps rendered from backups
- automated hosting of those maps on Cloudflare Pages

The goal was simple: **keep the world safe and make the server boring to operate**.

---

## Part 1: Automating Minecraft Server Backups

The foundation: a daily script that flushes the world to disk, creates timestamped backups, rotates old archives, and restarts the server safely.

➡️ **Read:** [Automating Minecraft Server Backups]({% post_url 2026-01-02-automating-minecraft-server-backups %})

---

## Part 2: Discord Notifications Without Spam

How I went from “post everything” to **one useful summary message** per run, with local logs and CSVs for detail.

➡️ **Read:** [Adding Discord Notifications Without Spam]({% post_url 2025-12-28-adding-discord-notifications-without-spam %})

---

## Part 3: Interactive Maps with uNmINeD

Rendering HTML maps from the **latest backup ZIP** (not the live server), with an atomic publish step to avoid partial renders.

➡️ **Read:** [Generating Interactive Minecraft Maps with uNmINeD]({% post_url 2026-01-02-generating-interactive-minecraft-maps-with-unmined %})

---

## Part 4: Cloudflare Pages for Automated Hosting

Using Wrangler + Cloudflare Pages to deploy the generated site to a stable HTTPS URL with minimal maintenance.

➡️ **Read:** [Hosting Automated Minecraft Maps with Cloudflare Pages]({% post_url 2026-01-11-hosting-automated-minecraft-maps-with-cloudflare-pages %})

---

## What’s Next

If I extend this system, it’ll probably be:

- a simple admin dashboard page (links to roster CSVs, latest backup, and maps)
- alerting only on failures (silent on success)
- a weekly “world change” highlight using map diffs
