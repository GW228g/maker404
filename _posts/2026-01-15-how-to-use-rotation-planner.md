---
layout: post
title: How to Use the 4v4 Rotation Planner Tool
date: 2026-01-15
---

Managing rotations during a youth basketball game can be stressful. Between coaching, keeping track of time, and managing substitutions, it's easy to lose track of who's played and who's sitting. That's why we built the 4v4 Rotation Planner, a free, mobile-friendly tool that takes the math out of managing fair rotations.

This guide will walk you through everything you need to know to use the tool effectively.

The same scheduling logic is also useful in school settings for rotating students through stations, labs, and collaborative roles.

## What Is the 4v4 Rotation Planner?

The 4v4 Rotation Planner is a web-based application designed specifically for youth basketball games played 4-on-4 across 8 periods. It automatically generates fair rotations ensuring every player gets equal (or nearly equal) playing time, while giving you the flexibility to mark your best ball handlers and adjust for late arrivals or injuries.

**Key Features:**
- Automatic rotation generation with multiple modes
- Fair playing time distribution
- Top player coverage (keeps ball handlers on court)
- Offline functionality (works without internet)
- Mobile-optimized for use during games
- Print and share options
- Saved rosters for future games

## Getting Started: Setting Up Your Team

### Step 1: Access the Tool

Navigate to [rotations.maker404.com](https://rotations.maker404.com) on any device. The tool works on phones, tablets, and computers. For the best game-day experience, we recommend using it on your phone so you can easily check it during timeouts.

**Pro Tip:** On mobile, you can "Add to Home Screen" to create an app-like experience that works offline.

### Step 2: Import Your Team

Instead of manually entering player names one by one, use the **Import List** feature:

1. Click the **"Import List"** button in the Players section
2. Paste your team roster with one name per line
3. Click **"Import"**

Your entire roster is now loaded and ready. You can always add individual players later by importing additional names.

### Step 3: Mark Your Top Players

The tool allows you to designate up to 2 players as "Top" players. These are typically your best ball handlers who can dribble under pressure and run your offense.

To mark a player as Top:
1. Find the player in your roster
2. Check the **"Top"** checkbox next to their name

When you mark players as Top, the tool's algorithms will ensure at least one of them is on the court during each period (when using "Fair time optimized" or "Sliding rotation (adaptive)" modes). This keeps your offense functional while maintaining fair overall playing time.

### Step 4: Organize Your Roster (Optional)

You can drag and drop players to reorder your roster:
1. Touch and hold the **☰** handle on the left side of any player
2. Drag the player up or down
3. Release to drop them in the new position

This is particularly useful for the "Sliding rotation (fixed)" mode, which uses roster order to determine rotation patterns.

## Generating Your Rotation

Once your roster is set up, it's time to generate the rotation schedule.

### Choosing a Rotation Mode

The tool offers four different rotation modes. Select the one that fits your coaching philosophy:

**Fair Time Optimized (Recommended)**
- Builds rotations period-by-period
- Ensures equal playing time as the top priority
- Automatically prioritizes Top players when possible
- Considers streak avoidance if enabled
- Best for: Most coaches who want fairness with smart ball handler coverage

**Sliding Rotation (Fixed)**
- Uses predetermined rotation patterns based on roster size
- Follows your roster order exactly as arranged
- Predictable and repeatable from game to game
- Ignores Top player designations
- Best for: Coaches who want consistent, predictable patterns

**Sliding Rotation (Adaptive)**
- Uses sliding patterns but automatically assigns Top players to positions with most playing time
- Still maintains overall fairness
- Combines predictability with smart prioritization
- Best for: Coaches who like patterns but want Top players getting maximum time

**True Random (Fair Time)**
- Completely random lineup selection each period
- Maintains strict fairness
- Maximum variety and unpredictability
- Ignores Top player designations
- Best for: Maximum lineup diversity while staying fair

### Optional Settings

Before generating, you can toggle these options:

**Top-2 Coverage**: When enabled, ensures at least one Top player is on court each period (only works with Fair Time Optimized and Sliding Adaptive modes)

**Avoid Long Streaks**: Penalizes rotations that make players sit for 3+ consecutive periods

**Auto Rebuild**: Automatically regenerates the rotation whenever you change your roster

### Generate the Rotation

Click **"Rebuild from current"** to generate your rotation schedule. Within seconds, you'll see:
- Complete lineups for all 8 periods
- Who's on court and who's sitting for each period
- Total playing time for each player

## Using the Tool During Games

### Tracking the Current Period

Use the **"Current period"** dropdown to indicate which period you're in. The tool will highlight that period in the Lineups section so you can easily see who should be on the court.

### Managing Late Arrivals and Injuries

Players don't always arrive on time, and injuries happen. Here's how to handle these situations:

**Late Arrival:**
1. Find the player in your roster
2. Uncheck **"Avail"** (Available) at the start of the game
3. When they arrive, check **"Avail"** again
4. Click **"Rebuild from current"** to regenerate remaining periods

**Injury During Game:**
1. Check the **"Out"** box for the injured player
2. Click **"Rebuild from current"**
3. The tool redistributes playing time among available players

### Locking Periods

Once a period is completed, you can lock it to prevent accidental changes:
1. Make sure you're on the completed period (use Current period dropdown)
2. Click **"Lock current period"**

Locked periods won't be changed when you rebuild rotations. This is useful if you need to adjust for the second half but want to preserve the first half exactly as played.

### Making Manual Adjustments

While the tool generates optimized rotations, you might occasionally want to make manual swaps. After rebuilding, you can always note changes on paper or mentally adjust. For future versions, we're considering adding drag-and-drop lineup editing.

## Sharing and Printing

### Copy to Clipboard

Click **"📋 Copy Text"** to copy the entire rotation schedule including playing time summary. You can then:
- Text it to your assistant coach
- Email it to parents
- Paste it into team communication apps

The copied format looks like:
```
🏀 Rotation Schedule

Period 1: Grace, Faith, Olivia, Emily
Period 2: Genieve, Grace, Faith, Olivia
...

📊 Playing Time:
Grace (TOP): 4 periods
Faith (TOP): 4 periods
...
```

### Print for Paper Backup

Click **"🖨 Print View"** to generate a printer-friendly version. This compact format fits on a single page and includes:
- Active and unavailable players
- All 8 period lineups
- Playing time totals

Having a printed backup is useful in case your phone dies or you want to post it in your team area.

## Advanced Tips

### Saving for Future Games

The tool automatically saves your roster using your browser's local storage. When you return to the site, your last roster will still be there. This means:
- No need to re-enter players each game
- Your Top player designations are remembered
- Quick setup for game day

### Starting Mid-Game

If you start using the tool partway through a game:
1. Set the "Current period" to where you are now
2. Click "Rebuild from current"
3. The tool generates rotations for remaining periods based on current playing time

### Resetting

**Reset Game**: Clears the schedule and locked periods but keeps your roster
**Reset Everything**: Completely clears all data including your roster (use this to start fresh with a new team)

## Troubleshooting Common Issues

**"Need 4 active players" message**: Make sure at least 4 players have "Avail" checked

**Unfair playing time**: 
- Check that you've marked the correct players as Available/Out
- Try clicking "Rebuild from current" again
- Ensure you're using an appropriate mode for your needs

**Can't mark more than 2 Top players**: This is intentional—the tool is designed for 2 primary ball handlers. If you need different logic, try "True Random" mode which ignores Top designations.

**Changes not saving**: Make sure your browser allows local storage and isn't in private/incognito mode

## Getting the Most Out of the Tool

The 4v4 Rotation Planner is designed to make game day easier, but it works best when combined with good coaching practices:

1. **Review your rotation before the game** - Don't wait until tip-off
2. **Communicate with your team** - Let players know the rotation plan
3. **Stay flexible** - The tool helps you plan, but you can always adjust
4. **Track throughout the game** - Update Current period and player status as needed
5. **Be consistent** - Using the same approach game-to-game helps players know what to expect

## Conclusion

The 4v4 Rotation Planner takes the stress out of managing youth basketball rotations. By automating the math and tracking playing time, you can focus on what matters most—coaching your players and helping them develop their skills.

Whether you prefer predictable patterns or optimized fairness, there's a rotation mode that fits your style. Give it a try at your next game and experience how much easier rotation management can be.

---

**Ready to try it?** Head to [rotations.maker404.com](https://rotations.maker404.com) and set up your team roster in under 2 minutes. See you on the court!
