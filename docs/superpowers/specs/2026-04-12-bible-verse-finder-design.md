# Bible Verse Finder — Design Spec

## Overview

A single-file HTML web app that drills Bible verse lookup speed. The user's 10-year-old daughter opens the file in her desktop browser, picks a round size, and races to find randomly generated verse references in her physical Bible. The app tracks every attempt in IndexedDB, shows improvement over time, and uses gamification (streaks, badges, levels) to keep her motivated.

## Target User

- Neha, 10-year-old girl
- Attends St Mary's Syriac Orthodox Church in Livermore, CA
- Preparing for a Sunday school Bible verse competition
- Using a standard Protestant Bible (66 books)
- Running the app in a desktop browser (no mobile needed)
- Honor system — no verse text verification

## Tech Stack

- **Single self-contained HTML file** — inline CSS and JS, no external dependencies
- **IndexedDB** for persistent data storage
- **No server required** — double-click to open in any browser

## Core Flow

### 1. Home Screen

The landing page she sees when she opens the app.

**Layout (desktop, full-width):**
- App title "Bible Verse Finder" top-left
- Streak counter prominently displayed (e.g., "5 day streak!")
- Level indicator with XP progress bar (e.g., "Level 4 · Explorer — 60% to Level 5")
- Quick stats row: Best Time | Average Time | Total Rounds
- Badge showcase: earned badges displayed as icons, unearned shown as locked/greyed out
- Three round-size buttons at the bottom: **5 Verses** | **10 Verses** | **20 Verses**

Clicking a round-size button immediately starts the round (transitions to the Verse Screen).

### 2. Verse Screen

Displayed during each verse in a round.

**Layout (desktop, full-width):**
- Top bar: app name left, running stats right (Avg so far, Best so far, verse count "3 / 10")
- Progress bar below top bar: segmented, filled segments for completed verses, pulsing segment for current, empty for remaining
- Center content (vertically and horizontally centered):
  - Verse reference in very large bold text (e.g., "Psalm 119:105")
  - Subtitle: "Find this verse in your Bible!"
  - Live timer counting up in large monospace-style text (e.g., "00:14"), updating every 100ms
  - Large "Found It!" button

**Behavior:**
- Timer starts immediately when the verse appears (no separate "Start" step)
- Clicking "Found It!" stops the timer and records the attempt
- Transitions to the Result Flash

### 3. Result Flash

Brief feedback shown after each verse find.

**Layout:**
- Large time display (e.g., "12 seconds!")
- Comparison text (e.g., "3s faster than your average!" or "New personal best!")
- Auto-advances to the next verse after 2 seconds
- If it was the last verse in the round, transitions to Round Summary instead

### 4. Round Summary

Celebration screen after completing a round.

**Layout:**
- Congratulatory message
- Round stats: average time, best time, worst time
- Comparison to previous rounds (e.g., "Your average improved by 4s since last round!")
- Any badges earned during this round
- Any personal bests broken
- "Play Again" button (returns to Home Screen)

## Verse Generation

Verses are generated randomly from the full Protestant Bible (66 books).

**Implementation:** A hardcoded data structure mapping each book to its chapter counts, and each chapter to its verse count. This ensures only valid verse references are generated (e.g., won't generate "Genesis 51:1" since Genesis has 50 chapters).

**Data source:** The verse count data for all 66 books will be embedded directly in the HTML file as a JavaScript object. This is roughly 1,200 entries (one per chapter across all 66 books).

**Random selection algorithm:**
1. Pick a random book (uniform distribution across all 66 books)
2. Pick a random chapter within that book
3. Pick a random verse within that chapter

No deduplication within a round (the odds of collision across 31,000+ verses are negligible).

## Data Model

### IndexedDB Database: `BibleVerseFinder`

**Object Store: `attempts`**
| Field | Type | Description |
|-------|------|-------------|
| id | auto-increment | Primary key |
| verse | string | Full reference, e.g., "Psalm 119:105" |
| book | string | Book name, e.g., "Psalms" |
| chapter | number | Chapter number |
| verseNum | number | Verse number |
| timeMs | number | Time to find in milliseconds |
| roundId | number | Foreign key to rounds store |
| timestamp | number | Date.now() when recorded |

**Object Store: `rounds`**
| Field | Type | Description |
|-------|------|-------------|
| id | auto-increment | Primary key |
| size | number | Round size (5, 10, or 20) |
| averageMs | number | Average time across all attempts in round |
| bestMs | number | Fastest attempt in round |
| worstMs | number | Slowest attempt in round |
| timestamp | number | Date.now() when round completed |

**Indexes:**
- `attempts`: index on `timestamp`, index on `book`, index on `roundId`
- `rounds`: index on `timestamp`

## Gamification

### Streaks

- A streak increments when she completes at least one round on a calendar day
- Consecutive days = streak count
- Missing a day resets the streak to 0
- Displayed on the home screen

### Levels

| Level | Name | Rounds Required |
|-------|------|----------------|
| 1 | Beginner | 0 |
| 2 | Learner | 5 |
| 3 | Reader | 15 |
| 4 | Explorer | 30 |
| 5 | Scholar | 50 |
| 6 | Expert | 80 |
| 7 | Master | 120 |
| 8 | Champion | 170 |
| 9 | Legend | 230 |
| 10 | Bible Navigator | 300 |

Progress bar shows percentage toward next level.

### Badges

| Badge | Condition |
|-------|-----------|
| First Steps | Complete first round |
| Bookworm | Complete 10 rounds |
| Bible Scholar | Complete 50 rounds |
| Speed Demon | Find a verse in under 10 seconds |
| Lightning Fast | Find a verse in under 5 seconds |
| Consistent | 7-day streak |
| Devoted | 30-day streak |
| Explorer | Find verses in 20 different books |
| Old Testament Pro | Find verses in all 39 OT books |
| New Testament Pro | Find verses in all 27 NT books |
| Master Navigator | Find verses in all 66 books |

Badges are checked after each round completes. Newly earned badges are highlighted on the Round Summary screen.

### Encouragement Messages

The app addresses Neha by name and shows encouraging messages at key moments:

**Result Flash (after each verse):**
- Rotating encouraging phrases like "Great job, Neha!", "You're getting faster!", "Keep it up, Neha!", "God is proud of you!"
- When she beats her average: "Amazing, Neha! That was faster than your average!"
- When she sets a personal best: "New personal best, Neha! You're on fire!"

**Round Summary:**
- Motivational closing messages like "Awesome round, Neha! You're going to do great at Sunday school!", "Practice makes perfect — keep going, Neha!", "You're becoming a Bible navigation expert!"
- When she improves over previous rounds: "You're improving, Neha! Your hard work is paying off!"

**Home Screen:**
- Welcome message: "Welcome back, Neha!" (or "Welcome, Neha!" on first run)
- Streak encouragement: "You've been practicing 5 days straight — amazing dedication!"

### Personal Bests

Tracked per Bible section:
- **Pentateuch** (Genesis–Deuteronomy)
- **History** (Joshua–Esther)
- **Poetry** (Job–Song of Solomon)
- **Major Prophets** (Isaiah–Daniel)
- **Minor Prophets** (Hosea–Malachi)
- **Gospels & Acts** (Matthew–Acts)
- **Epistles** (Romans–Jude)
- **Revelation** (Revelation)

Plus an overall personal best across all verses.

## Visual Design

**Style: Cool & Playful**
- Background: light blue gradient (`#E8F4FD` → `#D1ECFF`)
- Primary color: blue (`#3B82F6`, `#1A56DB`)
- Accent: purple (`#8B5CF6`)
- Success: green (`#10B981`)
- Text: dark blue for headings, muted blue for secondary text
- Cards/panels: white with subtle blue-tinted shadows
- Corners: rounded (12–16px)
- Typography: system font stack, bold weights for key info
- Layout: centered content, generous whitespace, desktop-optimized (not mobile)

## File Structure

Single file: `index.html`

Internal organization (within the file):
1. `<style>` block — all CSS
2. HTML markup — screen containers (home, verse, result-flash, round-summary)
3. `<script>` block — all JavaScript
   - Bible data (book/chapter/verse counts)
   - IndexedDB helpers (init, save attempt, save round, query stats)
   - Game logic (verse generation, timer, round management)
   - Gamification (streak calc, badge checks, level calc)
   - UI rendering (screen transitions, stat displays, animations)

## Edge Cases

- **First run:** No data in IndexedDB. Home screen shows "No rounds yet — start your first one!" instead of stats. Streak is 0. Level is 1 (Beginner).
- **Browser data cleared:** Same as first run. No recovery mechanism (acceptable for this use case).
- **Mid-round close:** Round is not saved. Attempts from the incomplete round are discarded (kept in memory only, written to IndexedDB on round completion).
- **Same verse twice in a round:** Allowed. Statistically unlikely (31,000+ verses) and not worth the complexity to prevent.
