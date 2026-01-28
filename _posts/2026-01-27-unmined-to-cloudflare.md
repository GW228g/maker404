---
layout: post
title: "Rendering a 850MB Minecraft World Into a Web Map Every Night"
date: 2026-01-27
categories: [minecraft, cloudflare, automation]
tags: [unmined, cloudflare-pages, cli, static-sites, maps]
excerpt: "How a free CLI tool, Cloudflare's generous free tier, and some careful tile optimization turned nightly Minecraft backups into an interactive web map that 'just works'—and what I learned about static site deployment limits along the way."
---

At some point in this project, I stopped caring about the Minecraft server itself.

The thing I actually wanted to build was **this**:

A way for my kids to open their phones at breakfast and see—from space—what they built the night before.

No downloads. No installations. No "Dad, can you show me the map?"

Just a URL: [maps.maker404.com](https://maps.maker404.com/)

Here's how that works, why it took longer than expected, and what I learned about static site deployment at scale.

---

## The Problem That Wasn't Obvious at First

When you think about "Minecraft map," you probably imagine:
- Opening Minecraft
- Pressing F3 for coordinates
- Maybe looking at the in-game map item

That's fine for players. It's terrible for documentation.

I wanted something else entirely:
- **Accessible from anywhere** – Phone, tablet, laptop, whatever
- **No Minecraft required** – Just a web browser
- **Always current** – Updates automatically, no manual work
- **Shareable** – Send a link, not a file
- **Explorable** – Zoom, pan, search for specific builds

Essentially: **Google Maps, but for a Minecraft world.**

That's harder than it sounds.

---

## Why This Is Actually Difficult

A Minecraft world is not a static image. It's a **database of 3D blocks** that can be viewed from infinite angles.

To turn that into a web map, you need to:

1. **Read the world format** (Minecraft's region files are compressed NBT data)
2. **Render it from above** (orthographic projection, handling transparency, lighting)
3. **Generate zoom levels** (tiles at different scales, like Google Maps)
4. **Output web-friendly format** (thousands of small PNGs + HTML/CSS/JS viewer)
5. **Deploy it somewhere** (with enough bandwidth to actually serve it)

Oh, and do all of this **automatically every night** without manual intervention.

---

## Enter uNmINeD

[uNmINeD](https://unmined.net/) is one of those tools that makes you wonder why anyone would build it for free.

It's a Minecraft mapping tool that:
- Reads world files directly (no server required)
- Renders beautiful orthographic maps
- Generates interactive web viewers
- Has a CLI for automation
- Actually works reliably

The GUI version is great for one-off renders. But I needed the **CLI version** for nightly automation.

And that's where things got interesting.

---

## How uNmINeD CLI Actually Works

The basic command looks deceptively simple:

```bash
unmined-cli web render \
  --world "/path/to/world" \
  --output "/path/to/output"
```

That's it. Point it at a world, tell it where to put the files, done.

Except not really. Because that command has opinions:

### Opinion 1: Directory Structure Matters

uNmINeD expects a specific structure:

```
world_root/
├── world/           # Overworld
├── world_nether/    # Nether
└── world_the_end/   # End
```

**Not** three separate world folders.  
**Not** the dimensions inside a single world folder.  
Exactly this structure.

My backups were structured differently. Took me an hour to realize why renders were failing silently.

### Opinion 2: It Renders Everything

By default, uNmINeD renders:
- Every chunk that's ever been generated
- At multiple zoom levels (0-5 by default)
- With full transparency and lighting
- Into thousands of individual PNG tiles

My world is 850MB. The first render produced **47,000 files** totaling 2.3GB.

That's a problem when Cloudflare Pages has a **25,000 file limit**.

### Opinion 3: Configuration Is Everything

Here's the config file I eventually settled on:

```javascript
{
  "version": 1,
  "worlds": [{
    "name": "Minecraft World",
    "dimension": "overworld",
    "path": "world",
    "zoom": { "min": 0, "max": 4 },
    "render": {
      "background": "#2C3E50",
      "lighting": "smooth",
      "transparency": true,
      "updateOnly": false
    }
  }]
}
```

Let me break down what each setting actually does:

**`zoom.max: 4`** – Only render up to zoom level 4, not 5  
This alone cut file count by 40%. Most people don't zoom in that far anyway.

**`updateOnly: false`** – Always do a full render  
Incremental renders are faster but can produce artifacts. I have CPU time at 2 AM, I don't have time to debug weird tile glitches.

**`lighting: "smooth"`** – Better looking, but slower  
At 2 AM, I don't care. Make it pretty.

**`transparency: true`** – Glass shows what's beneath it  
Essential for greenhouses and underwater builds.

---

## The Cloudflare Pages Part

Once I had tiles generating reliably, I needed somewhere to host them.

**Requirements:**
- Static file hosting (these are just HTML/CSS/JS + PNG tiles)
- Fast global CDN (so the kids' friends can view it without lag)
- Generous bandwidth (people zoom around a lot)
- Reasonable file limits (see: 47,000 file problem)
- Preferably free (it's a family project, not a business)

Cloudflare Pages checks every box. The free tier gives you:
- Unlimited bandwidth (!)
- 500 builds/month
- 20,000 files per deployment
- Global CDN
- Automatic HTTPS

The 20,000 file limit was my only constraint. And I was at 47,000 files.

---

## The Optimization Battle

Getting under 20,000 files took several iterations:

### Attempt 1: Reduce Zoom Levels
**Before:** `zoom.max: 5` (47,000 files)  
**After:** `zoom.max: 4` (28,000 files)  

Still over the limit, but closer.

### Attempt 2: Optimize Tile Size
uNmINeD generates 256×256px tiles by default. Larger tiles = fewer files:

```javascript
"tileSize": 512
```

**Result:** 19,400 files

Finally under the limit! But...

### The Problem With Large Tiles

512×512px tiles load slower on mobile. Each tile is 4x the data of a 256px tile.

I ran tests loading the map on:
- iPhone (fast connection)
- iPad (medium connection)  
- Laptop (fast connection)

512px tiles were noticeably sluggish on the phone.

### The Compromise

Back to 256px tiles, but with aggressive zoom limits:

```javascript
{
  "zoom": { "min": 0, "max": 3 },
  "tileSize": 256
}
```

**Final count:** 18,200 files

Not perfect, but good enough. And the map is snappy on every device.

---

## The Deployment Script

The actual deployment happens in a second script that the backup system calls:

```bash
#!/bin/bash
set -euo pipefail

BACKUP_DIR="/path/to/minecraft/backups"
WORK_DIR="/tmp/minecraft_web_render"
OUTPUT_DIR="$WORK_DIR/output"
CLOUDFLARE_PROJECT="minecraft-maps"

# Find latest backup
LATEST_BACKUP=$(ls -t "$BACKUP_DIR"/world_backup_*.zip | head -1)

# Extract to temp location
rm -rf "$WORK_DIR"
mkdir -p "$WORK_DIR"
unzip -q "$LATEST_BACKUP" -d "$WORK_DIR"

# Render the map
unmined-cli web render \
  --config unmined-web.json \
  --world "$WORK_DIR" \
  --output "$OUTPUT_DIR"

# Deploy to Cloudflare Pages
wrangler pages deploy "$OUTPUT_DIR" \
  --project-name "$CLOUDFLARE_PROJECT" \
  --branch main

# Cleanup
rm -rf "$WORK_DIR"
```

Simple, right?

**Wrong.**

Let me show you what I learned the hard way.

---

## The Problems I Hit (And How I Solved Them)

### Problem 1: Extraction Takes Space

**The issue:** Extracting a 850MB backup creates ~2.5GB of uncompressed world data.

**The solution:** Use `/tmp` instead of the backup directory. On macOS, `/tmp` is usually on a separate partition with more space. Plus it auto-cleans.

### Problem 2: Old Renders Interfere

**The issue:** uNmINeD can do incremental renders. But sometimes it gets confused by leftover files from previous renders.

**The solution:** `rm -rf "$WORK_DIR"` before every render. Start clean every time. Disk space is cheap, debugging weird artifacts is expensive.

### Problem 3: Uploads Are Slow

**The issue:** Uploading 18,200 files to Cloudflare takes ~4 minutes on my home connection.

**The solution:** There isn't one. 4 minutes is fine at 2 AM. I considered:
- Incremental uploads (complex, error-prone)
- Compression (Cloudflare does this automatically)
- Better internet (not happening)

So I just... let it take 4 minutes. Sometimes the solution is acceptance.

### Problem 4: Deployment Failures Are Silent

**The issue:** If the Cloudflare upload fails, the script exits with code 0 anyway.

**The solution:** Capture the exit code and log it:

```bash
set +e
wrangler pages deploy "$OUTPUT_DIR" \
  --project-name "$CLOUDFLARE_PROJECT" \
  --branch main
DEPLOY_RC=$?
set -e

if [[ $DEPLOY_RC -ne 0 ]]; then
  echo "Deployment failed with exit code $DEPLOY_RC"
  exit $DEPLOY_RC
fi
```

Now the backup script sees the failure and sends a Discord alert.

### Problem 5: Map Viewer URL Changes

**The issue:** Each Cloudflare Pages deployment gets a unique URL like `abc123.minecraft-maps.pages.dev`

**The solution:** Use a custom domain. Set up `maps.maker404.com` to point at the Cloudflare project. Now the URL never changes.

---

## What The Daily Workflow Looks Like

Every night at 2:00 AM:

1. **Backup script runs** (see previous blog post)
2. **Creates clean world backup**
3. **Calls the map deployment script**
4. **Script extracts the backup**
5. **uNmINeD renders 18,200 tiles** (~15 minutes)
6. **Wrangler uploads to Cloudflare** (~4 minutes)
7. **Cloudflare deploys to CDN** (~30 seconds)
8. **Discord notification sent** (success or failure)
9. **Temp files cleaned up**

Total time: **~20 minutes**

By 2:30 AM, the map is updated. By 7:00 AM, the kids are looking at it over breakfast.

---

## The Parts That Surprised Me

### Good Surprise: uNmINeD's Reliability

In four months, uNmINeD has never crashed. Never corrupted a render. Never produced weird artifacts (that weren't my configuration's fault).

It just works. Every night. Without complaint.

For a free tool maintained by one person, that's remarkable.

### Bad Surprise: Cloudflare's File Limit

I knew there was a limit. I didn't realize how easy it was to hit.

18,200 files sounds like a lot until you're generating zoom tiles for a large world.

I've seen other people hit this with:
- Photo galleries (each photo = multiple sizes)
- Documentation sites (lots of small files)
- Game asset catalogs

The limit is real. Plan for it.

### Weird Surprise: PNG Optimization Doesn't Help

I tried running the tiles through `optipng` and `pngcrush`.

Savings: ~8% on file size  
Time added: ~25 minutes  
Impact on load time: negligible

Not worth it. Modern PNG encoding is already pretty good.

---

## The Current State

The map updates successfully **every night**.

It shows:
- All three dimensions (overworld, nether, end)
- Zoom levels 0-3
- Smooth lighting
- Glass transparency
- Current world state

It costs:
- $0/month (Cloudflare free tier)
- ~20 minutes of compute time
- ~2.5GB of bandwidth per update

It serves:
- My family
- The kids' friends
- Occasional curious parents
- Me, when I want to see what they built

**And nobody thinks about it.**

That's success.

---

## Technical Stack (For The Curious)

**World Rendering:**
- uNmINeD CLI 0.20.5
- Custom JSON configuration
- macOS automation via launchd

**Hosting:**
- Cloudflare Pages (free tier)
- Custom domain (maps.maker404.com)
- Automatic HTTPS via Cloudflare

**Deployment:**
- Wrangler CLI 3.x
- Bash orchestration script
- Integrated with backup workflow

**Monitoring:**
- Discord webhooks for notifications
- Daily deployment logs
- Exit code checking + alerts

---

## What I'd Do Differently

### Use Progressive Web App Features

The map viewer could be a PWA. Add a service worker, make it installable, enable offline viewing of cached tiles.

I haven't done this because... nobody's asked for it. But it'd be cool.

### Add a "What's New" Overlay

Show areas that changed since the last render. Highlight new builds automatically.

uNmINeD doesn't support this natively, but I could compare tile hashes between renders.

### Generate Video Flyovers

String together map tiles into a video that flies over the world. Perfect for showing off builds.

This is technically possible but requires ffmpeg + manual camera path planning. Maybe someday.

### Cache Intermediate Renders

Right now, every render is from scratch. I could cache tiles that haven't changed.

But full renders are reliable, and I have CPU time at 2 AM. Complexity isn't worth the speedup.

---

## Lessons About Static Site Deployment

This project taught me more about static site hosting than I expected:

### Lesson 1: File Count Matters More Than File Size

**Total size:** 1.8GB  
**File count:** 18,200  

Cloudflare doesn't care about the 1.8GB. It cares about the 18,200 files.

Most static site generators don't hit this limit. But if you're deploying:
- Game assets
- Map tiles  
- Photo galleries
- Large documentation sites

You'll hit it eventually. Plan accordingly.

### Lesson 2: Incremental Deployments Are Hard

In theory, only upload changed files. In practice:
- File hashing adds complexity
- Partial deploys can leave orphaned files
- Full deploys are simple and reliable

Unless you're deploying huge sites multiple times per day, just redeploy everything.

### Lesson 3: CDN Propagation Is Fast

Cloudflare's CDN updates in ~30 seconds globally.

I've checked the map from:
- US East Coast (home)
- US West Coast (family)
- Europe (friends)

Everyone sees updates within a minute of deployment. That's impressive.

### Lesson 4: Free Tiers Are Generous (Until They're Not)

Cloudflare's free tier is absurdly good:
- Unlimited bandwidth
- Global CDN
- Automatic HTTPS
- 500 builds/month
- 20,000 files

But hit that 20,001st file and you're stuck. The paid tier jumps to $20/month.

Know your limits. Optimize to stay under them.

---

## The Real Win

This isn't about Minecraft maps.

It's about **removing friction**.

Before this system:
- "Dad, can you show me the map?" (interruption)
- "Let me boot Minecraft..." (5 minutes)
- "Okay, where did you build?" (coordination)
- "Hold on, loading..." (waiting)

After:
- Open browser
- Type `maps.maker404.com`
- Zoom to your build
- Done

**That's the infrastructure win.**

Not speed. Not scale. Not cost savings.

**Removing the need to ask.**

---

## For Anyone Building Something Similar

If you want to do this for your own Minecraft server:

### Start Here
1. Install uNmINeD CLI from [unmined.net](https://unmined.net/)
2. Sign up for Cloudflare Pages (free tier)
3. Point it at a world backup and render locally first
4. Optimize zoom levels to stay under 20k files
5. Automate when you're confident it works

### Key Config Settings
```javascript
{
  "version": 1,
  "worlds": [{
    "dimension": "overworld",
    "path": "world",
    "zoom": { "min": 0, "max": 3 },  // Adjust based on world size
    "render": {
      "lighting": "smooth",
      "transparency": true,
      "updateOnly": false
    }
  }]
}
```

### Watch Out For
- **Directory structure** – uNmINeD is picky about world folders
- **File count limits** – Count your tiles before deploying
- **Deployment time** – Factor in upload speed
- **Disk space** – Renders can be 2-3x the backup size

### Don't Overthink
- Full renders are fine (incremental is complex)
- 256px tiles work great (512px is overkill)
- Nightly updates are enough (hourly is excessive)
- Free hosting is sufficient (paid is unnecessary)

---

## The Quiet Success Metric

The map has been live for **four months**.

In that time, I've received:
- Zero bug reports
- Zero "it's not working" messages  
- Zero requests for features
- Multiple "this is cool!" comments

**Nobody complains because nothing breaks.**

That's the only metric that matters for infrastructure.

---

## Why This Matters (To Me)

I built this so my kids could:
- See their world from above
- Share builds with friends  
- Plan new projects together
- Feel like their world is real

And it works.

They send screenshots of the map to friends.  
They plan building projects over breakfast.  
They show me new areas they've explored.

**The map made the server feel permanent.**

Not because of the technology. Because of what the technology enabled.

---

## Final Thought

Someone asked me: "Why not just use a hosted service?"

There are services that do this. Paid ones. They're fine.

But I didn't build this because it was the easiest path.

I built it because I wanted to understand:
- How map rendering works
- How static sites deploy at scale  
- How CDNs propagate changes
- How automation handles failure

**The map is the artifact.**  
**The learning is the point.**

And now my kids have an interactive map of their world that updates every night, costs nothing, and just works.

That's worth 20 minutes of CPU time.

---

*The code for this is part of the larger [minecraft-automation](https://github.com/maker404/minecraft-automation) project. The unmined configuration, deployment scripts, and integration details are all there.*

*This is what Maker404 is about: building things that solve real problems, learning from the implementation, then letting them run quietly in the background.*
