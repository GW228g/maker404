---
layout: post
title: "Automating Minecraft Server Backups"
date: 2026-01-02
categories: [minecraft, homelab, automation]
tags: [minecraft, backups, bash, automation]
---

# Automating Minecraft Server Backups (So I Could Stop Worrying)

Every long-running Minecraft server eventually reaches a moment of clarity:

> “If this world disappears, people will be genuinely upset.”

By the time I reached that point, the server already had history — builds that took weeks, landmarks everyone recognized, and shared memories that couldn’t be recreated.

Manual backups were no longer acceptable.

---

## Why Backups Are Harder Than They Look

Minecraft worlds are not static files. The server keeps region data in memory and flushes it to disk asynchronously.

That means a naive backup — simply zipping the world folder — can result in:

- partially written region files  
- corrupted chunks  
- subtle world damage that only appears later  

So the very first rule became:

> **Never back up a world that hasn’t been flushed to disk.**

---

## Forcing the World to Disk

Before any backup happens, the script sends a `save-all` command directly into the server’s console session:

```bash
screen -S mcserver -p 0 -X stuff "save-all$(printf \r)"
sleep 2
```

That small pause is intentional. It gives the server enough time to finish disk writes before compression starts.

---

## Creating Consistent, Timestamped Backups

Each backup includes all three dimensions and uses a strict naming format:

```bash
world_backup_YYYY-MM-DD_HH-MM.zip
```

That consistency becomes extremely important later — especially for automation that *finds* the newest backup without guessing.

---

## Backup Retention Without Thinking

Old backups are automatically rotated:

```bash
ls -1t world_backup_*.zip | tail -n +8 | xargs rm -f
```

I always know:
- the last week is safe
- disk usage won’t grow forever
- I don’t have to clean this up manually

---

## Restarting as a Health Check

After the backup completes, the server is restarted intentionally.

This isn’t just housekeeping — it’s validation.

If the server can’t restart cleanly, I want to know *now*, not during a real crash.

Graceful stop first, forceful fallback second, retry if needed.

---

## The Backbone of Everything Else

Once this script proved reliable, it unlocked everything that came after:

- notifications
- player auditing
- map generation
- automated publishing

Nothing else works without trustworthy backups.
