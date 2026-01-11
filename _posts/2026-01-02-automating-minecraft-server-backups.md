---
layout: post
title: "Automating Minecraft Server Backups"
date: 2026-01-02
categories: [minecraft, homelab, automation]
tags: [minecraft, backups, bash, automation]
---

# Automating Minecraft Server Backups (So I Could Stop Worrying)

At some point, every long-running Minecraft server reaches the same moment:

> “If this world disappears, there will be tears.”

That moment came after our family server had been running long enough to accumulate real history: builds, landmarks, inside jokes, and shared memories. Manual backups were no longer acceptable. I needed something that ran **every day**, safely, without me thinking about it.

This post covers the foundation of the entire system: a **reliable, automated backup and restart script** on macOS.

---

## Why “Just Zip the World” Is Not Enough

Minecraft keeps chunks in memory. If you zip the world while the server is running, you can end up with:

- partially written region files  
- corrupted chunks  
- inconsistent world state  

The first rule of this project became:

> **Never back up a world that hasn’t been flushed to disk.**

---

## The Core Flow

My daily pipeline is:

1. Tell the server to flush world data to disk (`save-all`)
2. Zip the three world folders (Overworld, Nether, End)
3. Rotate old backups
4. Restart the server (graceful → forceful fallback)
5. Confirm the server is actually running again

---

## Sending `save-all` Before Backing Up

Because my server runs in a `screen` session, the script can inject commands directly:

```bash
screen -S mcserver -p 0 -X stuff "save-all$(printf \r)"
sleep 2
```

---

## Creating Timestamped Backups

Each backup includes all three dimensions:

```bash
DATE="$(date +'%Y-%m-%d_%H-%M')"
BACKUP_ZIP="$BACKUP_DIR/world_backup_$DATE.zip"

zip -r "$BACKUP_ZIP"   "$SERVER_DIR/world"   "$SERVER_DIR/world_nether"   "$SERVER_DIR/world_the_end"   > /dev/null
```

---

## Rotating Old Backups Automatically

I keep the most recent 7 backups:

```bash
MAX_BACKUPS=7
ls -1t "$BACKUP_DIR"/world_backup_*.zip 2>/dev/null   | tail -n +$((MAX_BACKUPS+1))   | xargs -r rm -f
```

---

## Restarting the Server Safely

After the backup is created, the script restarts the server.

Graceful stop:

```bash
screen -S mcserver -p 0 -X stuff "stop$(printf \r)"
sleep 10
```

If the Java process is still around, it gets terminated:

```bash
pkill -f "java.*server.jar" || true
sleep 2
```

And then we clear stale `session.lock` files:

```bash
rm -f "$SERVER_DIR/world/session.lock"       "$SERVER_DIR/world_nether/session.lock"       "$SERVER_DIR/world_the_end/session.lock"       || true
```

Finally, we start the server and retry a few times if needed:

```bash
MAX_RETRIES=3
for attempt in $(seq 1 "$MAX_RETRIES"); do
  screen -dmS mcserver /path/to/java -Xmx3G -Xms1G -jar server.jar nogui
  sleep 5
  pgrep -f "java.*server.jar" >/dev/null 2>&1 && break
  sleep 10
done
```

---

## What This Enabled Next

Once this backup pipeline was stable, it became the backbone for everything else:

- better notifications
- log auditing (who joined / who got blocked)
- automated map generation

Next: turning backups into something fun and useful — interactive maps.
