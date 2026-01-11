---
layout: post
title: "Generating Interactive Minecraft Maps with uNmINeD"
date: 2026-01-02
categories: [minecraft, visualization]
tags: [unmined, maps, cli, minecraft]
---

# Generating Interactive Minecraft Maps with uNmINeD

Once backups were reliable, I started asking a different question:

> “What changed today?”

Text logs can tell you who joined, left, or ran commands — but they can’t tell the story of a world evolving. Maps can.

This post covers how I generate **interactive HTML maps** from Minecraft worlds using the **uNmINeD CLI**, and why I render from backups instead of the live server.

---

## Why Render from Backups (Not the Live Server)

Rendering maps from a live server creates two problems:

- **consistency**: region files can be mid-write  
- **risk**: heavy reads can compete with gameplay  

So I decided:

> All map renders come from the newest backup ZIP.

That gives me a stable, frozen snapshot every time.

---

## The Basic Render Command

uNmINeD can produce a complete interactive map site:

```bash
unmined-cli web render   --world /path/to/world   --dimension overworld   --output /path/to/site/world
```

I run this three times:

- `overworld`
- `nether`
- `end`

---

## Extract → Locate Worlds → Render

The workflow looks like this:

1. Find newest `world_backup_*.zip`
2. Extract to a temp directory (usually `/tmp`)
3. Locate `world`, `world_nether`, and `world_the_end`
4. Render each dimension into its own output folder
5. Clean up temp files

The key: the renderer never touches the live server directories.

---

## Avoiding Partial Renders

If a render is interrupted, you can end up with a half-written site. The fix is to render into a temporary output directory and then sync atomically:

```bash
TMP_OUT="$(mktemp -d "/tmp/.unmined_web_out_world.XXXXXX")"

unmined-cli web render   --world "$WORLD_ROOT"   --dimension overworld   --output "$TMP_OUT"

rsync -a --delete "$TMP_OUT"/ "$SITE_DIR/world"/
```

`rsync --delete` ensures the destination exactly matches the new render.

---

## Why Interactive Maps Are Worth It

Interactive maps are genuinely useful:

- zoom in and explore without logging in
- spot new builds instantly
- see travel paths and “where people spend time”
- create a visual record of the world

Next: publishing these maps automatically to a stable HTTPS URL.
