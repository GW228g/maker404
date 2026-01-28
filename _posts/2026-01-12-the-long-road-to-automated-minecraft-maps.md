---
layout: post
title: "The Invisible Infrastructure: Building a Self-Healing Minecraft Backup System"
date: 2026-01-12
categories: [minecraft, automation, infrastructure]
tags: [bash, python, cloudflare-pages, unmined, papermc, devops]
excerpt: "What started as 'my kids want to play Minecraft' became a months-long meditation on building systems that fade into the background. Here's how a 500-line Bash script became the invisible guardian of a family's digital world."
---

At 2:47 AM on a Tuesday in November, my phone buzzed with a Discord notification:

> 🚨 Minecraft backup ran, but the server FAILED to restart after 3 attempts. Check logs on the Mac mini.

I was awake instantly. Not because the server was down—but because **the system had caught it**.

This is a story about that system. But really, it's about the slow, unglamorous work of building infrastructure that disappears.

---

## How It Actually Started

"Dad, can we play Minecraft together?"

That's how all great infrastructure projects begin, right? With a simple request from your kids?

I didn't want to put them on a public server. Too many variables I couldn't control. Too many strangers. Too much risk for something that should just be... fun.

So I did what any reasonable parent would do: I spun up a PaperMC server on a Mac mini in my office.

Simple. Private. Safe.

**Initial requirements:**
- Don't lose the kids' builds
- Don't let the server eat itself
- Don't make this a second job

That last one turned out to be the hardest.

---

## When "Just Works" Becomes "Just Enough"

For the first few weeks, everything was fine.

Then someone built a castle that took three days. Then someone else built a hidden base in the nether. Then there were shared projects, inside jokes, landmarks that became part of family lore.

The world was no longer replaceable.

And that's when I realized: **backups aren't optional anymore.**

At first, I did it manually. Stop the server, copy the files, zip them up, restart everything. It worked. Until it didn't.

Because "manual" means:
- Remembering to do it (I forgot)
- Having time to do it (I didn't)
- Not making mistakes (I did)

One night after the third time I forgot, I thought: *I already know how to solve this.*

---

## The Realization That Changed Everything

I had been thinking about this wrong.

I didn't need to figure out how to backup a *running* server safely. That's hard. File locks, data corruption, race conditions—all the things that make live backups terrifying.

But I was already running nightly restarts to clear memory and apply updates.

**The backups didn't need to be live. They needed to be part of the restart.**

That single realization turned an impossible problem into a solvable one.

---

## What the System Actually Does

The script runs every night at 2 AM. Here's what happens in those quiet hours:

### 1. **Graceful World Preservation**
```bash
# Tell the server to flush everything to disk
screen -S "$SESSION" -p 0 -X stuff "save-all$(printf \\r)"
sleep 2

# Then backup all three dimensions
zip -r "$BACKUP_ZIP" \
  "$SERVER_DIR/world" \
  "$SERVER_DIR/world_nether" \
  "$SERVER_DIR/world_the_end"
```

The server's still running. Players could still be online (they're not at 2 AM, but the system doesn't assume that). The `save-all` command tells Minecraft to write everything to disk first. We wait two seconds—long enough for it to finish, short enough that it doesn't matter.

Then we zip. The files are consistent because Minecraft just told us they are.

### 2. **The Stop-Clear-Start Dance**
```bash
# Stop the server
screen -S "$SESSION" -p 0 -X stuff "stop$(printf \\r)"
sleep 10

# If it didn't stop, force it
if is_running; then
  pkill -f "java.*${JAR_FILE}"
fi

# Clear stale locks that cause corruption
rm -f "$SERVER_DIR/world/session.lock" \
      "$SERVER_DIR/world_nether/session.lock" \
      "$SERVER_DIR/world_the_end/session.lock"
```

This is the part that took the longest to get right.

Minecraft doesn't always shut down cleanly. Sometimes it hangs. Sometimes it leaves lock files. Sometimes those lock files prevent restarts.

The script handles all of it. Graceful shutdown first. Force kill if needed. Clean up the locks. Every time.

### 3. **Self-Healing Restart with Retries**
```bash
for attempt in $(seq 1 $MAX_RETRIES); do
  screen -dmS "$SESSION" "$JAVA_PATH" -Xmx3G -Xms1G -jar "$JAR_FILE" nogui
  sleep 5
  if is_running; then
    log "Server started successfully on attempt $attempt."
    break
  fi
done

if ! is_running; then
  notify_discord "🚨 Server FAILED to restart after ${MAX_RETRIES} attempts"
  exit 1
fi
```

This is what caught that 2:47 AM failure.

Most backup scripts stop the server, make a copy, start it back up, and hope for the best. This one *checks*. If the server doesn't come back up, it tries again. Three times.

If all three fail, it screams for help.

### 4. **The Intelligence Layer**

The backup is just the foundation. The system also:

**Tracks who tried to join:**
```python
# Scan all logs for connection attempts
PAT = re.compile(r"""(?P<name>[A-Za-z0-9_]{3,16})\s+
                    (?:lost\s+connection|disconnect(ed)?):\s+
                    .*not\s+whitelisted.*""")
```

I wanted to know if someone was trying to get in. Not for security theater—for actual security. If a friend's kid wants to join, I want to see that request, not have it disappear into log files.

**Builds a player roster with last positions:**
```python
# Parse Essentials data to track where everyone logged out
# Useful for: "Where did I leave my stuff?"
# Useful for: "Did the kids actually go to bed?"
logout_x, logout_y, logout_z = extract_position(essentials_yaml)
```

**Generates daily activity summaries:**
```text
Date: 2026-01-11
Window: last 24h

✅ Backup: SUCCESS
   • File: world_backup_2026-01-11_02-00.zip
   • When: 2026-01-11 02:00:34
   • Size: 847.23 MB

🗺 Web maps: success
   • https://maps.maker404.com/

👥 Joined (3): Player1, Player2, Player3
🚪 Left   (3): Player1, Player2, Player3
🚫 Not whitelisted (1): RandomGriefer123
```

Every morning, this appears in our family Discord. Nobody asked for it. But now people notice when it doesn't show up.

### 5. **Automated Map Generation**

This deserves its own section because it's the part that makes people ask "wait, you built *what*?"

After the backup completes, the script calls a second script that:
- Extracts the latest backup
- Runs [uNmINeD](https://unmined.net/) to render interactive maps
- Deploys them to Cloudflare Pages
- Updates the live site at [maps.maker404.com](https://maps.maker404.com/)

Every morning, the kids can see what got built the night before. From space. With zoom. On any device.

I didn't plan for this to be the part everyone cares about most. But it is.

---

## What Actually Took the Time

Looking back through my commit history, this took **four months** to stabilize.

Not four months of work. Four months of:
- 20 minutes here
- An hour there
- "Why did it fail last night?"
- "Oh right, lock files"
- "Wait, why is the timestamp wrong?"
- "I should write that down"
- "Where did I write that down?"

### The Hard Parts

**Understanding uNmINeD's expectations:** It wants the world root, not the worlds themselves. It cares about directory structure. It silently fails if you get it wrong.

**Cloudflare Pages upload limits:** 25MB per file, 20,000 files total. My first map render was 47,000 files at varying sizes. I spent two weeks optimizing tile generation settings and implementing progressive rendering before I figured out the right balance.

**Log rotation timing:** Minecraft compresses yesterday's logs at startup. My script runs at 2 AM. Startup is at 2:01 AM. I was parsing logs that didn't exist yet. Took me three days to notice.

**Session lock corruption:** Sometimes the server crashes mid-save. The lock files stay behind. Next restart fails mysteriously. Took me a week to connect those dots.

**The playit.gg tunnel:** We use playit.gg so the kids' friends can connect from outside our network. It's great. Except it doesn't always reconnect after a server restart. Now the script handles that too.

### The Invisible Details

```bash
# Wait for network before trying to send Discord notifications
NET_OK=0
for _ in 1 2 3 4 5 6 7 8 9 10; do
  /sbin/ping -c1 -t1 1.1.1.1 >/dev/null 2>&1 && { NET_OK=1; break; }
  /bin/sleep 6
done
```

This 60-second retry loop exists because sometimes macOS hasn't fully initialized networking when the script runs. Without it, Discord notifications silently fail. Took me two weeks to realize they weren't sending.

```bash
# Keep 7 days of backups, automatically delete older ones
ls -1t "$BACKUP_DIR"/world_backup_*.zip | tail -n +8 | xargs -r rm -f
```

This single line prevents the disk from filling up. I found out I needed it when I woke up to a full disk and no backups from the previous three nights.

---

## Why This Matters

This system exists so my family can enjoy something together without friction.

It means:
- **The world feels permanent** – Nobody worries about losing their builds
- **Exploration is documented** – The maps show everywhere we've been
- **Someone's always watching** – If it breaks, I know immediately
- **I don't have to remember** – It just runs

But more than that, it means I get to be part of something without being the bottleneck.

The kids don't know this script exists. They don't need to. It's infrastructure.

**Good infrastructure disappears.**

---

## The Technical Stack

For anyone building something similar:

**Core Components:**
- **PaperMC** – The Minecraft server (Paper is Spigot with better performance)
- **Essentials** – Player data and tracking
- **uNmINeD** – Map rendering (the free version is shockingly good)
- **Cloudflare Pages** – Static site hosting (free tier, perfect for map tiles)
- **playit.gg** – Tunneling service for external access
- **Discord webhooks** – Notifications and daily summaries
- **Bash + Python** – The glue holding it all together

**Development Environment:**
- Mac mini (2018, i7, 32GB RAM)
- macOS launch daemon for scheduling
- GNU screen for process management
- Git for version control (because of course)

**Key Dependencies:**
- `zip` – World archival
- `screen` – Server management
- `curl` – Discord API calls
- `python3` – Data processing and log parsing
- `find`/`grep`/`awk` – Log analysis

---

## What I'd Do Differently

If I started over today:

1. **I'd use Docker** – Not because it's better, but because it's more portable. Right now this script is deeply tied to macOS paths and behaviors.

2. **I'd extract the Python scripts** – They're embedded in heredocs. They work, but they're a pain to test and modify.

3. **I'd add metrics** – Backup size over time, player activity patterns, server performance. I have the data, I'm just not visualizing it.

4. **I'd document more** – I have inline comments, but I don't have a "why I made this decision" journal. Future me would appreciate that.

5. **I wouldn't change the core approach** – Building around backups instead of live state was the right call. Everything else flows from that.

---

## The Part I'm Most Proud Of

Not the map rendering (though that's cool).

Not the self-healing restart logic (though that saved me).

Not even the Discord notifications (though my kids love them).

**It's the logging.**

```bash
log() {
  local msg="[$(date '+%Y-%m-%d %H:%M:%S')] $*"
  echo "$msg" | tee -a "$DAILY_LOG_FILE" >> "$ROLLING_LOG_FILE"
}
```

Every action gets logged twice: to a daily file and to a rolling file.

When something goes wrong at 2 AM, I don't have to guess. I have a timestamped record of exactly what happened.

When someone asks "when did PlayerX last join?" I can tell them.

When I'm debugging a weird issue three weeks later, the logs are still there.

**Good logging is a gift to your future self.**

---

## What This Actually Taught Me

Building this system taught me more about infrastructure design than any tutorial could.

**Systems thinking over features:**
- I didn't build a backup script. I built a system that happens to include backups.

**Reliability through simplicity:**
- Every piece can fail. The system handles it.

**Observability as a feature:**
- If I can't see it, I can't fix it. Logs and notifications aren't nice-to-have.

**Automation as respect:**
- Not just for my time—for everyone's time. Nobody should have to think about this.

**Personal infrastructure is real infrastructure:**
- Just because it's not running a company doesn't mean it's not important.

---

## The Quiet Success

This system has run **every single night** for the last four months.

It's survived:
- Server crashes
- Network outages  
- Disk space warnings
- macOS updates
- Power failures (thanks, UPS)
- My own coding mistakes

**And nobody in my family knows.**

They just know the server works. The maps update. Their builds are safe.

That's what success looks like for infrastructure: **invisibility**.

---

## For Anyone Building Something Similar

If you're running a home server—Minecraft or otherwise—here's what I'd tell you:

1. **Start with backups.** Everything else can be rebuilt. Data can't.

2. **Make it automatic.** If it requires you to remember, it will fail.

3. **Add notifications.** You won't check logs. You will read Discord.

4. **Handle failure gracefully.** Assume everything will break. Code for it.

5. **Log everything.** Future you is debugging at 2 AM with one eye open. Help them.

6. **Iterate slowly.** This isn't a weekend project. It's a system that grows with use.

7. **Let it disappear.** When it stops needing attention, you've won.

---

## The Source

The full script is [on my GitHub](https://github.com/maker404/minecraft-automation) (or will be soon).

It's not beautiful. It's not clever. But it works every night without complaint.

And sometimes, that's the highest compliment you can give code.

---

## Final Thought

Someone asked me recently: "Isn't this overkill for a kids' Minecraft server?"

Maybe.

But here's what I know:

My kids have a place they built together. That place is permanent. That place has maps. That place never loses progress.

**And I don't have to think about it.**

That's not overkill.

That's exactly enough.

---

*This is what Maker404 is really about: building systems that support the people you care about, then letting those systems fade into the background so you can focus on what matters.*

*The infrastructure isn't the point. What it enables is.*
