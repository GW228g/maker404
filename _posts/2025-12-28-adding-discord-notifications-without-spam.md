---
layout: post
title: "Adding Discord Notifications Without Spam"
date: 2025-12-28
categories: [minecraft, automation]
tags: [discord, notifications, webhooks, scripting]
---

# Adding Discord Notifications Without Spam

When I first automated my Minecraft server, everything *worked* — but it worked silently.

At first, that felt great. No pop-ups. No alerts. No noise.

Then the doubts crept in.

Did the backup actually run?  
Did the server restart cleanly?  
Would I notice if something failed?

That uncertainty was what pushed me to integrate Discord notifications. Unfortunately, my *first* attempt went too far in the opposite direction.

---

## The First Mistake: Posting Everything

Once you have a Discord webhook, it’s dangerously easy to post everything the script touches:

- every chat line  
- every command  
- every file generated  
- every map image  
- every debug message  

Technically impressive — completely unusable.

After a few days, the Discord channel became something everyone muted. That was the moment I realized an important rule:

> **If notifications are noisy, they get ignored.**

---

## Deciding What Actually Matters

I stepped back and asked a simpler question:

> “If I only read *one message per day*, what would I want it to tell me?”

The answer turned out to be very small:

1. Did the backup succeed?
2. Did the server restart properly?
3. Did anyone join or leave?
4. Were there whitelist blocks?
5. Did the maps publish successfully?

Everything else could live quietly on disk.

---

## Designing a High-Signal Summary

Instead of streaming logs into Discord, the script now **builds one summary at the end**.

Example:

```text
📅 Daily Minecraft Report

✅ Backup: SUCCESS
🗺 Web maps: success

👥 Joined (2): PlayerA, PlayerB
🚫 Not whitelisted (1): PlayerX
```

One message. Once per day. No scrolling required.

---

## Making Notifications Resilient

Another important lesson: **notifications should never be allowed to break the backup**.

The script:
- checks for network availability
- gracefully skips Discord if offline
- treats webhook failures as non-fatal

That way, the server stays healthy even if Discord doesn’t.

---

## Why This Matters More Than It Sounds

This change made the entire system feel *trustworthy*.

I didn’t need to babysit the server.
I didn’t need to scroll logs.
I didn’t need to guess.

Discord became a confidence check — not a firehose.

With notifications under control, it was finally safe to expand the system further.
