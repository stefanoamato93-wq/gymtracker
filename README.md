# Gym

> **Maintenance rule (read first): EVERY change the user asks for MUST be
> documented in this README.** No exception. Any time you modify `index.html`
> (a bug fix, a new feature, a layout tweak, a colour/style change, a copy
> tweak, a data-model change, anything at all the user requests), update the
> relevant sections of this README in the same change so the docs never drift
> from the code. Treat "update the README" as the mandatory final step of every
> task, not optional, even for one-line or cosmetic changes. If a request
> touches behaviour or appearance, the matching README section is updated before
> the task is considered done.

A single-file, mobile-first web app to run and log workouts at the gym: pick a
**program** (scheda), then for each exercise record the **weight (kg)**, the
**reps for each set**, and a **note**. Built to be used live during a session,
spin a roll to set the weight, and it prefills what you did last time so you just
update the numbers.

No backend code and no build step: everything (HTML, CSS, JS and the default
programs) lives in `index.html`. State syncs to a Firebase Realtime Database and
is cached in `localStorage`. To deploy: upload `index.html` to GitHub Pages.

## Deploy to GitHub Pages

1. Create a repo (e.g. `gym`) and upload `index.html`.
2. **Settings > Pages**, source "Deploy from a branch", `main`, root `/`.
3. Live at `https://<you>.github.io/gym/`.
4. On iPhone open in Safari and **Share > Add to Home Screen** for an app-like icon.

## Firebase: publish the security rules (one-time, required)

Sync reuses the existing RTDB project `groceries-d6616` (the same one the
calendar and grocery apps use) under a separate top-level node `gymTracker`, so
it never touches their data. The database has **per-node** rules, so until the
`gymTracker` node is granted access every read/write returns **HTTP 401** and
nothing syncs.

Open the Firebase console > project `groceries-d6616` > **Realtime Database** >
**Rules**, make sure the rules include the `gymTracker` node, then **Publish**:

```json
{
  "rules": {
    "lists": { "casa": { ".read": true, ".write": true } },
    "travelCalendar": { ".read": true, ".write": true },
    "gymTracker": { ".read": true, ".write": true }
  }
}
```

(Keep the `lists` and `travelCalendar` lines so the grocery and calendar apps
keep working; just add the `gymTracker` line.) No per-user auth: anyone with the
DB URL can read/write. Fine for personal workout data, do not store anything
sensitive.

Verify it worked (PowerShell):

```powershell
$base = "https://groceries-d6616-default-rtdb.europe-west1.firebasedatabase.app"
Invoke-RestMethod "$base/gymTracker.json"   # null = empty/ready (not 401)
```

`401` = rules not published yet. `null` = node empty and ready (the app will
seed it on first run). JSON = data is syncing.

## Using it

The header has three tabs: **History** (default), **Load**, and **Programs**.

### History (default view)
A reverse-chronological list of every workout, grouped by month. Each card is
kept minimal: the program badge (**A = red-tinted**, **B = blue**, matching the
consistency calendar), the date (day), and the exercise count.
A 📝 note line shows underneath if the session has one. **Tap a card to open and
edit** that workout. (The old per-exercise recap line was removed to keep the
list clean.)

At the top of History a **Training consistency** card shows how regularly you
train: a continuous week grid (columns = weeks, rows = Mon–Sun) that **starts
from the week of your earliest logged workout** (not a fixed window) and runs to
the current week. Month labels sit on top, and to make month boundaries precise
**each day is an outer box tinted by its month**, with the shade alternating
month to month (light grey / lighter grey), so the change of month reads per-day
even mid-week. A **smaller inner square is filled only on days you trained**,
coloured by program (**red = A**, **blue = B**, **green = other/none**).
**Tapping a trained day** shows that session's date as a toast and opens the
workout. Below the grid three pills summarise **trainings in the last 30 days**,
**total sessions**, and the **current week streak** (consecutive weeks with at
least one session; an empty in-progress current week does not break the streak).
It is computed inline from your workout data, nothing external.

### Load
The **Load** tab is an overall recap of training load, where load is defined as
the sum of **weight × reps** across every completed set of every exercise. At the
top a card shows the **total load** for the current selection (in `kg·reps`) plus
three pills: **sessions**, **avg / session**, and **best session**.

Below is a **stacked bar trend chart**: one bar per session in chronological order
(oldest left, newest right, horizontally scrollable), the bar height is the
session's total load, and it is **split into coloured segments per exercise** so
you see which movements drive the load. A legend maps each colour to an exercise,
and the value above each bar is its total. Tap a bar to open that session.

A **program filter** (All / A / B, shown when more than one program is in your
history) scopes the whole page to one program so you can read the trend within a
scheda. Bodyweight moves (0 kg) contribute nothing to load, so the metric tracks
weighted volume; a bodyweight-only session shows an empty (near-zero) bar.

### New workout
Tap the floating **+ New workout** button. Pick a program (or "Empty workout").
The editor opens pre-filled with that program's exercises, dated today, and each
exercise shows two hints: a **`last: ...`** hint with the weight and reps from
the most recent time you did it (prefilled into the inputs) so you only adjust
what changed, and a **`🏆 best: ...`** hint right next to it showing your best
set ever for that exercise, scored by **weight × reps** (the heaviest single set
by volume). For bodyweight moves (kg 0) the best falls back to the most reps in
one set. Both hints refresh live when you change the exercise in the dropdown.
For each exercise:

- **Reorder** — drag the **⠿** grip on the left of any exercise to move it up or
  down (works with touch). The order is saved with the workout.
- **Exercise name** — a **dropdown** of exercises you've done before (plus the
  built-in library). No free typing in the row. To use a brand-new movement pick
  **➕ Add new…** at the bottom of the list and type its name once; it's then
  remembered and selectable everywhere.
- **kg** — a horizontal **roll / wheel**: swipe left or right to dial the weight
  (0.5 kg steps, snaps to the centred value highlighted in blue).
- **kg / reps per set** — one row per set (1, 2, 3, ...) with **+/−** steppers.
  Each set has a small red **×** on the right to **delete that specific set**
  (the last remaining set can't be deleted). The **+** below the sets adds a new
  one. The +/− stepper buttons no longer trigger iOS double-tap zoom when tapped
  quickly (all buttons use `touch-action: manipulation`).
- **Note** — optional, per exercise (e.g. "Fastidio spalla dx", "Machine").
- **×** removes the exercise from this session.
- **+ Add exercise** adds another (dropdown selection, same as above).
- **Workout note** at the bottom for the whole session.

Tap **Save workout** (or **Save** in the header). **Delete** removes the session.

### Programs
Switch to the **Programs** tab to manage your schede. Tap a program to rename it,
add/remove/reorder exercises (▲▼), and set the default number of sets per
exercise. **+ New program** creates one. Deleting a program keeps your past
workouts. The built-in defaults **A** and **B** match the current sheet.

### Backup / Sync
The dot next to the title shows the connection (**green = online**, **grey =
offline**); edits made offline are queued and pushed on reconnect. Tap
**Backup** in the header to **Copy** or **Download** a JSON snapshot, **Import** a
JSON (becomes the shared truth, pushed to all devices), or **Reset** everything
to the built-in defaults.

## Default programs (from the sheet)

- **A**: Dead Hang, Mucca Gatto, Glute Bridge, Hip thrust, Elastici, Panca
  manubri, Lat Machine, Dead Bug, 90 degrees legs, Pelvic stretch.
- **B**: Dead Hang, Mucca Gatto, Glute Bridge, Bulgarian, Row chest supported,
  Elastici, Lateral raises seated, Pallof press, Biceps, 90 degrees legs,
  Pelvic stretch.

## Seeded history (from the sheet screenshot)

On a device's **first run** (no local data yet) the app preloads four sessions
transcribed from the tracking sheet, so History is not empty and the `last:`
hints work straight away: **23 May (A)**, **25 May (B)**, **28 May (A)** and
**1 Jun (B)**. The 28 May Panca manubri keeps the "Fastidio spalla dx" note.
These are pushed to the cloud on the first sync. After that the cloud is the
source of truth; edit or delete them like any other workout.

---

# Developer handoff

## What this is

A single-file vanilla-JS app (`index.html`, no framework, no build). Default
programs are baked into a compact `SEED_PROGRAMS` string and an initial history
is baked into `SEED_WORKOUTS` (transcribed from the sheet). All data (workouts,
programs, custom exercise names) syncs to a **Firebase Realtime Database** and is
cached in `localStorage`. Bump `VERSION` (top of the CONFIG `<script>`) on
functional changes.

The file has 5 `<script>` blocks: (1) a banner comment, (2) CONFIG + SEED +
DATA MODEL, (3) RENDERING + WORKOUT EDITOR, (4) PROGRAM PICKER/EDITOR + BACKUP +
CLOUD SYNC GLUE + INIT, and (5) a `<script type="module">` at the very end that
is the Firebase transport. Classic scripts share one global scope; the module
talks to them only through `window.*` hooks.

## Data model

```js
// Program (scheda) — a template
{ id:'p_a', name:'A', exercises:[ { name:'Hip thrust', sets:3 }, ... ] }

// Workout (session), keyed by id under WORKOUTS
{
  id:'w_xxx', date:'2026-06-15', program:'A', note:'',
  exercises:[
    { name:'Hip thrust', kg:20, sets:[12,12,12], note:'' },
    { name:'Dead Hang',  kg:0,  sets:[45,45],    note:'' }
  ]
}
```

`sets` is an array of reps per set (numbers; `''` for a blank set, also used for
seconds on holds like Dead Hang). `kg` is a number (0.5 steps, decimals allowed).

## Firebase (Realtime Database)

- Config is inline in the final module. It **reuses the grocery/calendar project**
  `groceries-d6616`, under a separate top-level node `gymTracker`.
- Node layout: `gymTracker/workouts/<id>`, `gymTracker/programs` (array),
  `gymTracker/exLib` (array of custom exercise names).
- **Security rules required** — see the deploy section above. If reads/writes
  401, the `gymTracker` rule is missing.
- The module is a thin transport exposing `window.fb`:
  `setWorkout(id,val)`, `removeWorkout(id)`, `setWorkouts(obj)`,
  `setPrograms(arr)`, `setExlib(arr)`, `removeAll()`.
- It calls back via `window.on*` hooks: `onRemoteWorkouts`, `onRemotePrograms`,
  `onRemoteExlib`, `onSyncStatus`. Migration helpers `localWorkouts` /
  `localPrograms` / `localExlib` push this device's data up on the first
  snapshot when the cloud node is `null`.
- **`SEED_WORKOUTS`** is loaded only on the very first run (when `localStorage`
  has no workouts key); ids are deterministic (`w_seed_YYYYMMDD_<prog>`) so
  re-seeding never duplicates. Programs fall back to `DEFAULT_PROGRAMS` when the
  cloud `programs` node is empty/null.

## Debugging sync

```powershell
$base = "https://groceries-d6616-default-rtdb.europe-west1.firebasedatabase.app"
Invoke-RestMethod "$base/gymTracker.json"            # whole node (null if empty)
Invoke-RestMethod "$base/gymTracker/workouts.json"   # just the workouts
Invoke-RestMethod "$base/gymTracker/programs.json"   # just the programs
```

- HTTP **401** = security rules don't allow the node (fix the rules).
- `null` = node empty (no successful write yet; open the app, dot should go green).
- Returns JSON = sync is working.

## Common edits

- **Change the default programs**: edit `SEED_PROGRAMS` (format
  `Name | Exercise*sets ; ...`). Only used when the cloud `programs` node is empty.
- **Change the seeded history**: edit the `SEED_WORKOUTS` array (each exercise is
  `[name, kg, [reps...], note?]`). Only applied on a fresh device/first run.
- **Add exercises to the dropdown**: append to `DEFAULT_EXLIB`. At runtime the
  workout-editor name field is a `<select>` built by `fullExLib()`, which unions
  exercises done in past workouts + program exercises + `DEFAULT_EXLIB` + custom
  `EXLIB`. Picking **➕ Add new…** prompts for a name and remembers it in `EXLIB`.
- **kg roll (wheel) picker**: the `<select>`-free weight control. Ticks are
  generated once into `KG_TICKS` (0..`KG_MAX` in `KG_STEP` increments) and reused
  per card. `positionRoll()` centres the roll on the current value (call after the
  modal is visible via `initKgRolls()`), and the scroll handler writes the centred
  tick's value into the hidden `.kg-in` input that `collectWorkoutFromEditor` reads.
- **Drag-to-reorder**: `wireDragHandle()` runs a pointer-based drag from the
  `.exc-grip` handle (touch-friendly, uses `setPointerCapture` + a placeholder).
  Exercise order is taken from DOM order at save time.
- **`lastEntryFor(name)`**: finds the most recent prior workout containing that
  exercise; powers both the `last:` hints and the prefill on new workouts.
- **`bestEverFor(name)`**: scans the whole history and returns the best single
  set `{ score, kg, reps, date }`, scored by `kg*reps` (falls back to max reps
  when `kg` is 0). **`exHintHtml(name)`** builds the combined `last: ... · 🏆
  best: ...` line shown under each exercise (used by `exerciseCardHtml` and
  `updateLastHint`, the latter called when the name dropdown changes).
- **Load recap page**: `viewMode` can be `history` | `load` | `programs`
  (persisted in `localStorage` under `gymView`; the header `#viewSeg` has a
  `data-view="load"` button). `render()` routes `load` to `renderLoad()`. Load
  helpers: `exerciseLoad(ex)` = Σ `kg × reps` over an exercise's completed sets
  (reuses `completedSetDetails`) and `sessionLoad(w)` sums that over the session.
  `renderLoad()` shows a total/avg/best summary card, an optional program filter
  (`#loadFilter` seg control, state in module-level `loadProgFilter`, rebuilds on
  click), and a **stacked bar chart** built with flexbox divs: `.load-chart` is
  `align-items:flex-end`; each `.load-col` holds a value label, a `.load-stack`
  (`flex-direction:column-reverse`, fixed height = `load/maxLoad*CHART_H`) whose
  `.load-seg` children are per-exercise heights, and an x-label. Exercise colours
  come from `EX_PALETTE` mapped by first-appearance order (`colorOf`). It
  re-renders on remote workout updates (`onRemoteWorkouts` refreshes for both
  `history` and `load`). `fmtInt` formats values with thousands separators;
  `.load-*` / `.chart-*` CSS styles it. To change the palette edit `EX_PALETTE`;
  to change chart height edit the `CHART_H` const.
- **Training-consistency heatmap**: `renderConsistency(list)` builds the week
  grid + stat pills at the top of History; `renderHistory` prepends its output.
  `ymdLocal(date)` formats a local `Date` to `YYYY-MM-DD`. The window is
  **dynamic**: `start` is the Monday of the earliest workout's week (falls back
  to ~8 weeks when there's no data), and `WEEKS` runs to the current week
  (capped at 104). Each cell is an **outer box** with class `m0`/`m1` by
  `month % 2` (the alternating month shade) plus `on`/`pa`/`pb` when trained; the
  inner `<span class="hm-in">` is the coloured training square. Trained cells get
  `clk` + `data-id`/`data-date`; `renderHistory` wires `.hm-cell.clk` clicks to
  `toast(date)` + `openWorkout(id)`. Month labels are emitted at the column that
  contains the 1st of a month. Cell/label sizing and the month shades
  (`.hm-cell.m0` / `.m1`) live in the `.hm-*` CSS. Streak is computed by walking
  weeks back with `weekHas(mondayDate)` and skips an empty in-progress current
  week so it does not reset mid-week.
- **`.row2` layout**: the Date/Program editor row is `display:flex`; its
  children use `flex:1 1 0; min-width:0`. The native **`<input type="date">`**
  also gets `-webkit-appearance:none; appearance:none` (plus a fixed `height`)
  in CSS, because iOS otherwise renders it at a fixed intrinsic size, ignores
  `width:100%`, and paints over the Program `<select>`. Keep both the
  `min-width:0` and the `appearance:none` on the date input if you touch this row.
