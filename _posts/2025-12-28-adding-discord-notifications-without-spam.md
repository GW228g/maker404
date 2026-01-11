---
layout: post
title: "Adding Discord Notifications Without Spam"
date: 2025-12-28
categories: [minecraft, automation]
tags: [discord, notifications, webhooks, scripting]
---

# Adding Discord Notifications Without Creating Noise

The first version of my server automation worked — but it was silent.

Silence sounds nice until you realize it’s ambiguous:
- Did the backup run?
- Did the server restart?
- Did anything fail?

So I added Discord notifications.

Then I added too many.

This post is about how I moved from “post everything” to a setup that’s actually useful: **one high-signal summary**, and the details saved locally when I need them.

---

## The Trap of Over-Notification

Once you have a webhook, it’s tempting to post:
- chat previews
- command logs
- screenshots and map images
- file uploads
- debug output

I did that. It worked for a week, and then it became noise.

Lesson learned:

> Notifications should summarize, not narrate.

---

## What Actually Matters Daily

After watching the server for a while, I narrowed it down to five things:

1. Did the backup succeed?
2. Did the server restart cleanly?
3. Who joined and left recently?
4. Were there whitelist blocks?
5. Did the maps publish successfully?

Everything else belongs in local files.

---

## Posting to a Discord Webhook

A Discord webhook is just an HTTP POST. A minimal helper in bash looks like:

```bash
notify_discord() {
  local msg="$1"
  curl -fsS -H "Content-Type: application/json"     -d "{"content":"${msg}"}"     "$DISCORD_WEBHOOK_URL" >/dev/null
}
```

In the real script, I made it “production-safe”:

- **Network-aware**: skip if the network isn’t up yet  
- **Non-fatal**: webhook failures never break backups  

---

## Waiting for the Network (Quietly)

On headless machines, `curl` can fail during boot even though things are fine. I added a short loop to avoid false alarms:

```bash
NET_OK=0
for _ in 1 2 3 4 5 6 7 8 9 10; do
  ping -c1 -t1 1.1.1.1 >/dev/null 2>&1 && { NET_OK=1; break; }
  sleep 6
done
```

If `NET_OK` stays 0, the script keeps going — just without notifications.

---

## The One-Message Rule

The biggest improvement was switching to **one summary message per run**. That eliminated duplicates and turned Discord back into a tool instead of a firehose.

Example format:

```text
📅 Daily Minecraft Report

✅ Backup: SUCCESS
🗺 Web maps: success

👥 Joined (2): Alex, Sam
🚪 Left (2): Alex, Sam
🚫 Not whitelisted (1): Steve123
```

If I want the full detail, I can open the local CSV or text logs.

---

## Why This Matters

This was one of those small-but-huge ops lessons:

> If notifications are noisy, you stop reading them — and then you miss the important ones.

Once Discord became a high-signal summary channel, I could trust it again.

Next: how I built a backup script solid enough to support everything else.
