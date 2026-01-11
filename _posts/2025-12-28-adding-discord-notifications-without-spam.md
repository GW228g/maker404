---
layout: post
title: "Adding Discord Notifications Without Spam"
date: 2025-12-28
categories: [minecraft, automation]
tags: [discord, notifications, webhooks, scripting]
---

# Adding Discord Notifications Without Creating Noise

The first version of my server automation worked — but it was silent.

Silence is ambiguous:
- Did the backup run?
- Did the server restart?
- Did anything fail?

So I added Discord notifications.

Then I added too many.

This post is about how I moved from “post everything” to a setup that’s actually useful: **one high-signal summary**, with details saved locally when needed.

---

## The Trap of Over-Notification

Once you have a webhook, it’s tempting to post:
- chat previews
- command logs
- screenshots and map images
- debug output

I did that. It worked briefly… and then it became noise.

> Notifications should summarize, not narrate.

---

## What Actually Matters Daily

After observing the server for a while, I narrowed it down to:

1. Did the backup succeed?
2. Did the server restart cleanly?
3. Who joined and left recently?
4. Were there whitelist blocks?
5. Did the maps publish successfully?

---

## One Clean Daily Summary

```text
📅 Daily Minecraft Report

✅ Backup: SUCCESS
🗺 Web maps: success

👥 Joined (2): PlayerA, PlayerB
🚫 Not whitelisted (1): PlayerX
```

No spam. No scrolling. Just signal.
