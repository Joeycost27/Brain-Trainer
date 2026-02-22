# FocusForge — Full System Scope & Architecture
**Version 1.0 — Build Blueprint**

---

## 1. What This Is

FocusForge is a browser-based brain training platform with two distinct but connected experiences:

- **Daily Flashcard Mission** — off-screen activities drawn from the 99-item reference bank, designed to be done in the real world with parental encouragement
- **Online Training Game** — interactive digital challenges with timers, multiple choice, XP, and scoring that can be played directly on the device

Both share the same hub page, the same scoring system, the same badge set, and the same progress/report engine. The Teaching Assistant is available throughout.

**No server required.** Everything runs from a local folder or a static web host. All data lives in the browser's `localStorage` and is exportable as TSV on demand.

---

## 2. File Structure

```
focusforge/
│
├── index.html              ← Hub / home page — entry point for everything
├── daily.html              ← Daily Flashcard Mission
├── game.html               ← Online Interactive Training Game
│
├── question_bank.tsv       ← 99 off-screen activity reference bank (READ ONLY)
├── game_bank.tsv           ← Interactive game questions with MC options (READ ONLY)
│
└── assets/                 ← (optional) sounds, icons if added later
```

**Data that lives in localStorage (not files):**
- Player profile (name, age group, avatar)
- XP, level, streak count, last session date
- Completed card IDs (for cycle tracking)
- Badge unlock status and dates
- Session history (raw data for report generation)

**Data that gets downloaded on demand:**
- `progress_report.tsv` — generated from localStorage, downloadable any time
- `session_log.tsv` — full session-by-session log

---

## 3. Page Breakdown

### 3.1 `index.html` — The Hub

The home screen. Every session starts here.

**Sections:**
- Header: FocusForge logo, Teaching Assistant button, Player name + avatar
- Stats strip: Level, XP to next level, current streak, total sessions
- Two big entry buttons: **Daily Mission** → `daily.html` | **Training Game** → `game.html`
- Badge showcase: earned badges displayed, locked ones greyed out
- Recent activity feed: last 5 sessions with date, score, XP earned
- Footer: Parent Link (subtle, bottom right) | Download Report | Wipe Data

**Parent Link behaviour:**
- Small unobtrusive text link bottom-right: "Parent View"
- Tapping it opens a slide-up panel (no PIN in v1 — kept simple)
- Panel shows: session history table, total XP, badge progress, option to download TSV report, option to reset all data
- Parent can also set how many days of data to retain (7 / 14 / 30 / keep all)

---

### 3.2 `daily.html` — Daily Flashcard Mission

**Session flow:**

1. **Mission Briefing screen** — "Today's Mission: 3 Challenges" — shows category icons for today's cards (no titles yet, just icons — anticipation builds)
2. **Card reveal** — one card at a time, full screen
   - Front: Title, category badge, time suggestion, prompt/hook sentence
   - Flip/tap: Full instructions, success criteria, materials needed
   - Audience tag visible (youth/adult/both)
3. **"I Did It!" / "Do This Later"** buttons — no forced completion, no guilt
4. **Between cards**: XP celebration animation if marked done, brief encouragement message
5. **End of mission**: Summary screen — cards completed, XP earned, streak status, any badge unlocked
6. **Boss Round trigger**: if 5-day streak active, a 4th card appears flagged as BOSS (difficulty 3, higher XP)

**Randomisation logic:**
- Date is used as a daily seed
- Cards are drawn from the full 99, weighted by:
  - Not seen in last 25 sessions (prevents repeats)
  - Matches current age group filter
  - Balanced across categories (no two from same category in one day)
- Seed resets at midnight — same card set available all day for that date (so parent and child see same mission)

**Scoring:**
- Completing a card: base XP from `xp_value` column
- Boss card completion: 3× XP multiplier
- Full mission completion (all cards done): +50 XP bonus
- Streak bonus: +10 XP per consecutive day

---

### 3.3 `game.html` — Online Interactive Training Game

The digital skill-builder. Uses `game_bank.tsv` — questions designed to be answered on screen.

**Session flow:**

1. **Mode select**: Quick Play (5 questions random) | Focused (pick a category) | Boss Challenge (unlocked at level 5)
2. **Question card** — displayed one at a time:
   - Question text (randomised variables injected from pools)
   - Countdown timer (ring animation, seconds from `time_seconds` column)
   - 4 multiple choice options (1 correct + 3 distractors, shuffled)
   - Hint button (costs 5 XP — visible but costs something)
3. **Answer feedback**: Immediate — correct (green flash, XP gain) or wrong (red, correct answer revealed, brief explanation)
4. **Between questions**: XP tally running total, combo counter (consecutive correct answers)
5. **End of session**: Score breakdown, XP earned, category performance, personal best comparison
6. **Boss Round**: after completing any full category set (all questions in that category attempted), a Boss Round unlocks — 3 harder questions, double XP, timed more tightly

**Randomisation — variable injection:**
The game engine holds pools of random data injected at question-load time:

| Pool name | Contents |
|---|---|
| `WORDS_N` | Random word list, N items (drawn from 200-word vocabulary) |
| `NUMBERS_N` | Random number sequence, N digits |
| `SEQUENCE_N` | Number sequence with a pattern (arithmetic/simple) |
| `SCENARIO` | Random everyday situation (e.g. "You're getting ready for school") |
| `NAME` | Random first name |
| `EMOTION` | Random emotion word |
| `COLOUR` | Random colour |
| `OBJECT` | Random everyday object |
| `TASK` | Random simple task description |

Example question template in TSV:
> `"You were just told this list: {{WORDS_5}}. Which word came THIRD in the list?"`

Engine injects 5 random words, computes correct answer (word[2]), generates 3 distractors from the same list. Every play = genuinely different question.

---

## 4. TSV File Schemas

### 4.1 `question_bank.tsv` (existing — no changes needed)

Already built. 99 rows. Used by `daily.html`.

| Column | Purpose |
|---|---|
| id | Unique ID |
| category | Skill domain |
| subcategory | Specific skill |
| audience | youth / adult / both |
| difficulty | 1–3 |
| activity_type | streak / focus_drill / goal_setting / reflection |
| time_minutes | Suggested off-screen time |
| title | Card title |
| prompt | Hook sentence |
| instructions | Full activity instructions |
| success_criteria | What done looks like |
| materials_needed | What to have ready |
| sensory_friendly | yes / no |
| tags | Comma-separated |

---

### 4.2 `game_bank.tsv` (to be built — ~80 rows)

Used by `game.html`. Every question is a template, not static text.

| Column | Purpose |
|---|---|
| id | e.g. GB-001 |
| category | Matches category names from question_bank |
| subcategory | Specific skill tested |
| audience | youth / adult / both |
| difficulty | 1–3 |
| question_template | Text with `{{VAR}}` placeholders |
| variable_slots | Comma list of variable types needed e.g. `WORDS_5,POSITION` |
| correct_logic | How correct answer is derived e.g. `words[position-1]` |
| distractor_logic | How wrong answers are generated e.g. `other_words_from_list` |
| option_format | How options display e.g. `word / number / sentence` |
| xp_value | 10 / 25 / 50 |
| time_seconds | Countdown timer length |
| is_boss | true / false |
| hint | One hint sentence (costs XP to reveal) |
| explanation | Brief why the answer is correct (shown after answer) |
| tags | Comma-separated |

---

### 4.3 Generated Reports (downloaded as TSV)

**`progress_report.tsv`** — one row per session:

| date | mode | duration_mins | questions_attempted | questions_correct | pct_correct | xp_earned | categories_played | streak_day | level_at_session | boss_completed | badges_earned |

**`session_detail.tsv`** — one row per question across all sessions:

| date | session_id | question_id | category | difficulty | answer_correct | time_taken_sec | hint_used | xp_earned |

Both generated fresh from localStorage on demand. Parent can download either or both. No data ever leaves the device unless explicitly downloaded.

---

## 5. XP & Levelling System

| Level | Name | XP Required |
|---|---|---|
| 1 | Spark | 0 |
| 2 | Igniter | 150 |
| 3 | Focuser | 400 |
| 4 | Sharpener | 750 |
| 5 | Challenger | 1,200 |
| 6 | Forger | 1,800 |
| 7 | Master Forger | 2,600 |
| 8 | Brain Titan | 3,600 |

**XP Sources:**

| Action | XP |
|---|---|
| Daily card completed | 25 |
| Full daily mission (all cards) | +50 bonus |
| Game question correct | 10 |
| Game question correct (difficulty 2) | 25 |
| Game question correct (difficulty 3) | 50 |
| Boss round completed | 150 |
| 7-day streak | 200 bonus |
| First card in new category | 75 bonus |

---

## 6. Badge System

| Badge | ID | Unlock Condition |
|---|---|---|
| First Spark | badge_first | Complete first ever card |
| On a Roll | badge_3streak | 3-day streak |
| Week Warrior | badge_7streak | 7-day streak |
| Mind Miner | badge_ef | First Executive Function card done |
| Memory Keeper | badge_wm | First Working Memory card done |
| Laser Focus | badge_fs | First Focus Stamina card done |
| Pause Master | badge_im | First Impulse Management card done |
| Emotion Expert | badge_er | First Emotional Regulation card done |
| Life Forge | badge_rw | First Real-World card done |
| Blueprint Pro | badge_tp | First Technical Planning card done |
| Category Conqueror | badge_allcats | All 7 category badges earned |
| Boss Slayer | badge_boss | First Boss Round completed |
| Full Deck | badge_99 | All 99 reference cards completed |
| Speed Demon | badge_speed | 5 game questions answered correctly under 5 seconds each |
| Hint-Free | badge_nohint | Full game session with 0 hints used |

---

## 7. Teaching Assistant

Available on all three pages via a persistent button.

**Function:**
- Stores a project/assistant URL in localStorage
- On click: opens the URL in a new tab
- Optional context injection: passes current question ID and category as URL parameters so the TA can provide targeted help e.g. `https://your-ta-link.com?q=WM-003&cat=Working+Memory`

**Future-ready:** when an API link is provided, the TA button can be upgraded to an inline chat panel without rebuilding the app.

---

## 8. Data Lifecycle & Parent Controls

**Storage:** `localStorage` only. Nothing is sent anywhere.

**Parent panel (accessible from hub footer):**
- View session history table (last N days)
- Set retention: 7 / 14 / 30 days / keep all
- Download `progress_report.tsv`
- Download `session_detail.tsv`
- Wipe all data (with confirmation)
- Toggle age group: Youth / Adult / Both (filters which cards are served)

**Auto-pruning:** sessions older than the retention setting are automatically removed from localStorage when the app loads, keeping the data lean.

---

## 9. Build Order

| Phase | What gets built | Files |
|---|---|---|
| **Phase 1** | Game bank TSV — 80 interactive questions with variable templates | `game_bank.tsv` |
| **Phase 2** | Hub page — home screen with stats, badges, navigation | `index.html` (rebuild) |
| **Phase 3** | Daily Mission — flashcard engine with seed, flip, XP, streaks | `daily.html` |
| **Phase 4** | Online Game — MC engine, timer, variable injector, scoring | `game.html` |
| **Phase 5** | Report engine — session logging, TSV download, parent panel | wired into all pages |
| **Phase 6** | Polish — badge animations, level-up celebrations, Teaching Assistant context passing | all pages |

---

## 10. Technical Notes

- **No build tools required.** Vanilla HTML/CSS/JS. Runs from any static host or local folder via a simple HTTP server.
- **TSV parsing:** `fetch()` + `text.split('\n').map(l => l.split('\t'))`. Header row skipped. Robust to trailing whitespace.
- **Daily seed:** `new Date().toDateString()` hashed to an integer → used to seed a deterministic shuffle of the card pool. Same result for anyone opening the app on the same day.
- **localStorage keys:** all prefixed `ff_` to avoid collisions.
- **Variable injection:** regex replace on `{{TOKEN}}` patterns at question-render time, not stored. Each page load generates fresh values.
- **Report TSV generation:** `Blob` + `URL.createObjectURL` + programmatic `<a>` click. No server needed.
- **Offline capable:** once loaded, works with no internet (Google Fonts will gracefully degrade).

---

*Scope document v1.0 — ready for Phase 1 build.*
