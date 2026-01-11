---
layout: post
title: "Generating Interactive Minecraft Maps with uNmINeD"
date: 2026-01-02
categories: [minecraft, visualization]
tags: [unmined, maps, cli, minecraft]
---

# Generating Interactive Minecraft Maps with uNmINeD

Once backups were reliable, I wanted a better way to *see* change over time.

Logs tell you *what* happened. Maps show you *where*.

---

## Render from Backups, Not Live Worlds

All maps are rendered from backup ZIPs — never from the live server. This avoids:

- mid-write corruption
- performance impact during play

---

## Rendering Maps

```bash
unmined-cli web render \
  --world extracted/world \
  --dimension overworld \
  --output site/world
```

Each dimension is rendered separately and published together.
