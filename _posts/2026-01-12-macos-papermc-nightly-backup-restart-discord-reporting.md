---
layout: default
title: "macOS PaperMC Nightly Backup + Restart + Discord Reports"
date: 2026-01-12
categories: [minecraft, automation, macos]
tags: [papermc, launchd, backup, discord, unmined, scripting]
excerpt: "A nightly launchd job that backs up all worlds, restarts PaperMC safely, exports access logs, and posts summaries + map updates to Discord."
---

This post documents a **nightly macOS `launchd` script** I use to keep a **PaperMC (Java) Minecraft server** healthy and backed up. The goal: *hands-off reliability*—zip backups, clean restarts, log extraction, roster reporting, and optional Discord notifications.

> **Privacy note:** I’ve **redacted personal paths and webhook URLs**. Treat the config block below as the “public template.”

## What this script does

- **Creates zipped backups** of `world`, `world_nether`, and `world_the_end`
- **Rotates backups** (keeps the newest *N*)
- **Gracefully stops + restarts** PaperMC (with retries)
- **Clears stale `session.lock` files** to avoid world lock issues after crashes/restarts
- **Extracts “not whitelisted” join attempts** from logs → CSV
- **Builds a roster CSV** merging Essentials user data with not-whitelisted attempts (and preserves manual “blocked” flags)
- **Archives chat + commands** into daily + rolling logs (deduped)
- **Posts previews + full log file to Discord** (safe 2k char handling)
- **Restarts a playit.gg tunnel** (optional)
- **Triggers uNmINeD map render + web map deploy** (optional)
- **Posts a daily summary to Discord** (joins/leaves/blocked + backup status)

## Requirements

- macOS
- `screen`
- `zip`
- `python3`
- `curl`
- PaperMC server folder with standard `logs/`
- *(Optional)* EssentialsX plugin (for the roster merge)
- *(Optional)* uNmINeD CLI + your own render/deploy scripts

## Recommended folder layout

You can organize however you want; the key is to set the variables up top.

- `SERVER_DIR` → your PaperMC folder (contains `server.jar`, `world/`, `logs/`)
- `BACKUP_DIR` → where zips + reports go

## Configuration (edit these)

Set these at the top of the script:

- `SESSION` — screen session name for the server
- `SERVER_DIR` — PaperMC server directory
- `BACKUP_DIR` — backup/output directory
- `JAVA_PATH` — JDK Java binary
- `JAR_FILE` — Paper jar filename
- `MAX_BACKUPS` — backup rotation count
- `MAX_LOG_DAYS` — how long to keep script logs
- `MAX_RETRIES` — how many times to attempt restart
- `DISCORD_WEBHOOK_URL` — **don’t hardcode**; use env var or a local file

### Don’t hardcode secrets
In production I recommend **storing your webhook URL in a file** readable only by you, e.g.

- `~/.backup_webhook` containing only the webhook URL
- `chmod 600 ~/.backup_webhook`

## Running it nightly with launchd

Create a LaunchAgent or LaunchDaemon (depending on whether you need a GUI session). For a true nightly background job, **LaunchDaemon** is typical.

Example (skeleton) `~/Library/LaunchAgents/com.maker404.mcbackup.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key><string>com.maker404.mcbackup</string>
  <key>ProgramArguments</key>
  <array>
    <string>/bin/bash</string>
    <string>/path/to/minecraft_backup.sh</string>
  </array>
  <key>StartCalendarInterval</key>
  <dict>
    <key>Hour</key><integer>3</integer>
    <key>Minute</key><integer>0</integer>
  </dict>
  <key>StandardOutPath</key><string>/tmp/mcbackup.out</string>
  <key>StandardErrorPath</key><string>/tmp/mcbackup.err</string>
  <key>RunAtLoad</key><true/>
</dict>
</plist>
```

Load it:

```bash
launchctl load ~/Library/LaunchAgents/com.maker404.mcbackup.plist
```

## The script (public template)

> Replace the values in the SETTINGS section, and review the optional sections before enabling them.

```bash
#!/bin/bash
set -euo pipefail

# ====== SETTINGS ======
SESSION="mcserver"
SERVER_DIR="/path/to/MinecraftServer"
BACKUP_DIR="/path/to/Minecraft_Backups"
JAVA_PATH="/path/to/jdk-21/bin/java"
JAR_FILE="server.jar"
MAX_BACKUPS=7
MAX_LOG_DAYS=90
MAX_RETRIES=3

# Prefer env var or a local file for secrets
DISCORD_WEBHOOK_URL="${DISCORD_WEBHOOK_URL:-}"
WEBHOOK_FILE="$HOME/.backup_webhook"

LOOKBACK_HOURS=24

DATE=$(date +"%Y-%m-%d_%H-%M")
DAILY_LOG_FILE="$BACKUP_DIR/mc_log_$DATE.txt"
ROLLING_LOG_FILE="$BACKUP_DIR/mc_log_all.txt"

mkdir -p "$BACKUP_DIR"

log() {
  local msg="[$(date '+%Y-%m-%d %H:%M:%S')] $*"
  echo "$msg" | tee -a "$DAILY_LOG_FILE" >> "$ROLLING_LOG_FILE"
}

is_running() {
  pgrep -f "java.*${JAR_FILE}" >/dev/null 2>&1
}

# --- webhook helpers ---
WEBHOOK_URL="$DISCORD_WEBHOOK_URL"
if [[ -z "$WEBHOOK_URL" && -f "$WEBHOOK_FILE" ]]; then
  WEBHOOK_URL="$(/bin/cat "$WEBHOOK_FILE" | /usr/bin/tr -d '[:space:]')"
fi

# Wait briefly for network (max ~60s)
NET_OK=0
for _ in 1 2 3 4 5 6 7 8 9 10; do
  /sbin/ping -c1 -t1 1.1.1.1 >/dev/null 2>&1 && { NET_OK=1; break; }
  /bin/sleep 6
done

notify_discord() {
  local msg="$1"
  if [[ $NET_OK -eq 1 && -n "$WEBHOOK_URL" ]]; then
    /usr/bin/curl -fsS -H "Content-Type: application/json"       -d "{\"content\":\"${msg}\"}" "$WEBHOOK_URL"       >/dev/null 2>&1 || echo "⚠️ Discord webhook failed (continuing)"
  else
    echo "ℹ️ Skipping Discord notify (no network or no WEBHOOK_URL)."
  fi
}

# ====== BACKUP WORLDS ======
log "===== SCRIPT STARTED ====="
log "Starting backup..."

if screen -list | grep -q "\.${SESSION}\b"; then
  log "Sending 'save-all' to server..."
  screen -S "$SESSION" -p 0 -X stuff "save-all$(printf \\r)"
  sleep 2
fi

zip -r "$BACKUP_DIR/world_backup_$DATE.zip"   "$SERVER_DIR/world"   "$SERVER_DIR/world_nether"   "$SERVER_DIR/world_the_end" > /dev/null

log "Backup saved: $BACKUP_DIR/world_backup_$DATE.zip"
ls -1t "$BACKUP_DIR"/*.zip 2>/dev/null | tail -n +$((MAX_BACKUPS+1)) | xargs -r rm -f

# ====== STOP SERVER ======
if is_running; then
  log "Stopping Minecraft server..."
  screen -S "$SESSION" -p 0 -X stuff "stop$(printf \\r)"
  sleep 10
fi

if is_running; then
  log "Force killing leftover java process..."
  pkill -f "java.*${JAR_FILE}" || true
  sleep 2
fi

# ====== CLEAR STALE LOCKS ======
rm -f "$SERVER_DIR/world/session.lock"       "$SERVER_DIR/world_nether/session.lock"       "$SERVER_DIR/world_the_end/session.lock" || true

# ====== START SERVER (RETRY) ======
log "Starting server..."
cd "$SERVER_DIR"

for attempt in $(seq 1 $MAX_RETRIES); do
  screen -dmS "$SESSION" "$JAVA_PATH" -Xmx3G -Xms1G -jar "$JAR_FILE" nogui
  sleep 5
  if is_running; then
    log "Server started successfully on attempt $attempt."
    break
  else
    log "Server failed to start (attempt $attempt/$MAX_RETRIES)..."
    sleep 10
  fi
done

if ! is_running; then
  log "ERROR: Could not start server after $MAX_RETRIES attempts."
  exit 1
fi

# ====== EXTRA REPORTING (OPTIONAL) ======
# - Scan logs for not-whitelisted attempts → CSV
# - Merge Essentials roster + attempts → CSV
# - Extract chat + command lines → daily + rolling archive
# - Post previews + files to Discord
# - Restart playit.gg tunnel
# - uNmINeD render + deploy
# - Daily Discord summary
#
# (These sections are kept in my private version; copy them in as needed.)

log "===== SCRIPT FINISHED (SUCCESS) ====="
exit 0
```

## Notes and gotchas

- **`launchd` environment is “minimal”**. If Discord notifications mysteriously stop working under launchd, it’s usually because environment variables aren’t loaded—reading the webhook from a file is the most reliable fix.
- Avoid hardcoding secrets into public repos. Use env vars or a local file and `.gitignore` it.
- If your server doesn’t use `screen`, you can swap that control logic for a different process manager.

## Next improvements (ideas)

- Add a “health check” HTTP endpoint (or simple status file) that your monitoring can read
- Encrypt backups at rest (if storing off-machine)
- Push backups to cloud storage (S3/R2) for disaster recovery
