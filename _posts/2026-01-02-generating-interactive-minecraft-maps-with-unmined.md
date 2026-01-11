---
layout: post
title: "Generating Interactive Minecraft Maps with uNmINeD"
date: 2026-01-02
categories: [minecraft, visualization]
tags: [unmined, maps, cli, minecraft]
---

# Generating Interactive Minecraft Maps with uNmINeD

After backups were solved, I started wanting something different.

Not logs.
Not charts.
Not spreadsheets.

I wanted to *see* the world.

---

## Why Maps Are Different

Maps answer questions logs never can:

- Where are people building?
- How far have players explored?
- What changed since last week?

Static screenshots help — but interactive maps change how you explore a world.

---

## Why Rendering from Backups Matters

Rendering maps directly from a live server is risky:

- region files can be mid-write  
- disk-heavy reads compete with gameplay  
- a crash during rendering can corrupt output  

Instead, every map render starts from the **latest backup ZIP**.

This guarantees:
- consistent input
- zero impact on players
- reproducible results

---

## The Rendering Process

The workflow is simple but intentional:

1. Locate the newest backup ZIP
2. Extract to a temporary directory
3. Render each dimension separately
4. Publish the finished output atomically
5. Clean up everything temporary

The live server is never touched.

---

## Interactive HTML, Not Images

uNmINeD generates full HTML sites, not static PNGs:

```bash
unmined-cli web render   --world extracted/world   --dimension overworld   --output site/world
```

The result is zoomable, pannable, explorable — usable by anyone with a browser.

---

## Seeing History Instead of Reading It

The real value of these maps isn’t just convenience.

It’s history.

You can scroll back through time and *see* how the world grew — where paths formed, bases expanded, and exploration slowed.

That’s something logs will never capture.
