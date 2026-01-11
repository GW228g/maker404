---
layout: post
title: "Automating Minecraft Server Backups"
date: 2026-01-02
categories: [minecraft, homelab, automation]
tags: [minecraft, backups, bash, automation]
---

# Automating Minecraft Server Backups (So I Could Stop Worrying)

Every long-running Minecraft server eventually reaches the same realization:

> If this world disappears, people will be genuinely upset.

That’s when manual backups stop being acceptable.

---

## Why “Just Zip the World” Is Not Enough

Minecraft keeps world data in memory. Zipping while the server is running can result in:

- partially written region files
- corrupted chunks
- inconsistent state

Rule number one:

> Never back up a world that hasn’t been flushed to disk.

---

## Safe Backup Flow

1. Flush world data (`save-all`)
2. Create timestamped ZIP backups
3. Rotate old archives
4. Restart the server cleanly

---

## Sending `save-all`

```bash
screen -S mcserver -p 0 -X stuff "save-all$(printf \r)"
sleep 2
```

---

## Timestamped Backups

```bash
zip -r "$BACKUP_DIR/world_backup_$DATE.zip" \
  "$SERVER_DIR/world" \
  "$SERVER_DIR/world_nether" \
  "$SERVER_DIR/world_the_end"
```

---

## Why This Script Matters

Once backups were reliable, everything else — notifications, maps, and publishing — became safe to automate.
