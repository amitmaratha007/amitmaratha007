# Codex Work Order — IDBI MIS "Work Command Center"

**Audience:** the coding agent (Codex). **Author:** planning pass over the existing code.
**Goal:** turn this single-owner demo into a real, multi-user, official IDBI Bank tool.

This app is an **offline, vanilla-JS single-page app** (no framework, no build step, no
internet/CDNs) served on the office LAN by a tiny Node server (`server.js`) with a Python
fallback (`server.py`). Do **not** introduce a framework, a bundler, npm packages, or any
external/CDN dependency. Keep it plain HTML/CSS/JS that runs by double-clicking
`Start-Dashboard.bat`.

---

## 0. Ground rules (read first)

1. **No new runtime dependencies.** Pure stdlib server, pure browser JS. Offline forever.
2. **Never lose user data.** Every change must include a migration path (see Task 6). The
   app currently stores personal data in browser `localStorage` under key
   `idbi.amit.dash.data.v1`. That data must survive the upgrade.
3. **Two servers stay in parity.** Any API change in `server.js` must be mirrored in
   `server.py` (same routes, same JSON shapes, same atomic temp-file→rename write).
4. **Keep both servers' write pattern atomic:** write to `*.tmp`, then rename over the real
   file (as `/api/save` already does — `server.js:40-42`, `server.py:48-51`).
5. **Bump cache-busting.** After editing assets, change every `?v=10` to `?v=11` in
   `index.html` (lines 7, 102, 103) so browsers on the LAN pick up the new code.
6. **Match the existing code style:** the codebase uses terse, single-line helpers,
   `data-action` attributes dispatched through one delegated `onClick` handler
   (`assets/app.js:717-752`), and `$ / $$` query helpers. Follow that style; don't refactor
   into modules.
7. After each task, smoke-test by launching `Start-Dashboard.bat` and hard-refreshing
   (Ctrl+Shift+R).

### Locked product decisions (do not re-litigate these)
- **Workspace storage = hybrid ("both"):** the **server `data.json` is the source of truth**;
  the browser keeps a **local mirror/cache** for offline resilience. On load: try server,
  fall back to cache. On change: write server **and** update the cache.
- **Identity = admin maps IP → person (keep the current model).** A person's PC is mapped to
  their `people[]` id in **⚙ Settings → Map IP → person**; that mapping decides whose
  workspace loads. The host PC (`127.0.0.1`) is Superadmin automatically. **Do not** add a
  self-select "pick your name" login.
- **Links & Tech Contacts = shared + personal (both).** There is one **shared/team** list
  *and* each person has their **own private** list. Both are editable **without** the Edit
  toggle and **without** special permission (see Task 4).

---

## 1. Current architecture (map, with exact references)

**Files**
| File | Role |
|---|---|
| `index.html` | Shell: sidebar, topbar, nav, content host, overlays. |
| `assets/app.js` (784 lines) | All logic. IIFE, no exports. |
| `assets/style.css` | All styling (glass / "liquid" theme, 5 themes). |
| `assets/holidays.js` | Maharashtra 2026 holiday list. |
| `data.json` | **Shared, server-side** team data (canonical). |
| `server.js` / `server.py` | Offline LAN web servers (pick whichever runtime exists). |
| `Start-Dashboard.bat` | Launcher. |

**Two data stores today**
- **TEAM (shared, on server):** `data.json` → `meta`, `access`, `workTags`, `people`,
  `workItems`, `links`, `vendors`. Loaded by `loadTeam()` (`app.js:141-144`); saved by
  `saveTeam()` → `POST /api/save` which **overwrites the whole file** (`app.js:145-149`,
  `server.js:34-51`).
- **PERSONAL (per-browser, single owner):** `localStorage["idbi.amit.dash.data.v1"]` →
  `projects`, `reqTypes`, `requests`, `trackedProjects`, `tasks`, `trackers`. Defined by
  `PERS_KEY` + `PERS_DEFAULTS` (`app.js:56-105`); `loadPersonal/savePersonal`
  (`app.js:107-124`). **Hardcoded to Amit** via `MY_ID = "p-amit"` (`app.js:39`) and
  `meta.owner = "Amit Dethe"`.

**Identity / access (`app.js:130-164`)**
- `MY_IP` from `GET /api/whoami`; `currentUserId = ipMap[MY_IP]`; loopback ⇒ Superadmin
  (`boot()` `app.js:774-777`).
- Superadmin "view-as" impersonation: `viewAs`, `realUserId`, `realSuper`,
  `applyViewAs/enterViewAs/exitViewAs` (`app.js:134-139`). View-as is a **read-only preview**.
- `viewScope()` = self + descendants (`app.js:156-158`); `canEdit(personId)` (`app.js:159`).

**The three bugs you're fixing live here**
- Logo (fabricated, 3 places): favicon `index.html:8`; sidebar SVG `index.html:15-21`;
  welcome screen `app.js:694` + `style.css:470`.
- "Saved" indicator: markup `index.html:86` (`#savedInd`), CSS `style.css:148-149,297-299`,
  updater `setSaved()` `app.js:179`.
- Workspace is single-owner: title hardcoded `"Amit Dethe · …"` `app.js:286`; toast
  `app.js:677`; owner resolution `workspaceOwnerId()` `app.js:162`; `MY_ID` `app.js:39`.
- Links/Contacts gated behind Edit mode + permission: `viewLinks()` `app.js:433-440`,
  `viewVendors()` `app.js:442-448`, actions `app.js:738-739`, `guardTeam()` `app.js:680`.

---

## 2. THE DATA-MODEL CHANGE (do this first — everything else depends on it)

Add a **per-person workspace map** to the canonical server file and split links/contacts into
shared + personal.

### 2.1 New shape of `data.json`
Keep all existing top-level keys. **Add** one key, `workspaces`, and treat the existing
top-level `links` / `vendors` as the **shared** lists:

```jsonc
{
  "meta": { ... },              // unchanged
  "access": { ... },            // unchanged (ipMap still drives identity)
  "workTags": [ ... ],          // unchanged (team work-item tags)
  "people": [ ... ],            // unchanged
  "workItems": [ ... ],         // unchanged (team pendency)
  "links":   [ ... ],           // SHARED team links   (existing)
  "vendors": [ ... ],           // SHARED tech contacts (existing)

  "workspaces": {               // NEW — one entry per person id
    "p-amit": {
      "meta":      { "owner": "Amit Dethe", "version": 5 },
      "projects":  [ /* the 4 default project tabs, per-workspace & renameable */ ],
      "reqTypes":  [ /* the 4 default request types, per-workspace */ ],
      "requests":  [ ... ],
      "trackedProjects": [ ... ],
      "tasks":     [ ... ],
      "trackers":  [ ... ],
      "links":     [ ... ],     // THIS PERSON'S private links
      "vendors":   [ ... ]      // THIS PERSON'S private tech contacts
    },
    "p-shivesh": { ...empty-but-seeded... },
    ...
  }
}
```

- `projects`, `reqTypes`, `requests`, `trackedProjects`, `tasks`, `trackers` carry exactly the
  fields they have today in `PERS_DEFAULTS` (`app.js:57-105`). Do not change those row shapes.
- **`projects` and `reqTypes` become per-workspace and editable** (today they're hardcoded and
  re-cloned on every load — `app.js:113`). This is what lets each user "create their own
  options" (their own project tabs / request types). Add modal editors for them (reuse the
  existing modal pattern, `openModal()` `app.js:500-503`).

### 2.2 New-workspace seed (important for Task 3 / point 4 & 5)
When a person opens a workspace that doesn't exist yet, seed it with:
- The **4 default project tabs** and **4 default request types** (copied from
  `PERS_DEFAULTS.projects` / `PERS_DEFAULTS.reqTypes`) so the UI isn't empty/confusing,
- **zero** rows in `requests`, `trackedProjects`, `tasks`, `trackers`, `links`, `vendors`,
- `meta.owner` = that person's name (from `people[]`).

So when **Shivesh** opens his workspace he sees his **own empty** Oracle-GL/ASP-GSP/… tabs to
fill — **never Amit's data**. Amit's existing rows belong only to `workspaces["p-amit"]`.

### 2.3 Client state
- Replace the global single `PD` with **`PD = the current user's workspace object`**, resolved
  from `data.json.workspaces[ workspaceOwnerId() ]` (seed if missing).
- `workspaceOwnerId()` (`app.js:162`) ⇒ return **`currentUserId`** (which already equals the
  viewed-as person under view-as, the mapped person otherwise). Remove the `|| MY_ID` fallback
  and delete/retire `MY_ID` (`app.js:39`).
- When `currentUserId` is `null` (an unmapped guest who is not Superadmin), **do not** load
  anyone's workspace. Show the personal area as a friendly empty state:
  *"Ask your admin to map this PC to your name (⚙ Settings) to get your own workspace."*
  **Never show one person's personal data to another/unidentified user.**

---

## 3. Task 1 — Use the **real IDBI Bank** logo (point 1)

The current logo is **hand-drawn / fictional** (green box + orange swoosh + "IDBI" text). This
is an official-use app, so it must show the **actual IDBI Bank logo from an asset file** — do
**not** redraw or invent a mark.

**Steps**
1. Expect an official logo asset in `assets/`. Use these filenames:
   - `assets/idbi-logo.svg` (preferred vector), and/or `assets/idbi-logo.png`,
   - `assets/favicon.png` (or `.ico`) for the browser tab.
   > These files are supplied by the bank (download from the official IDBI Bank brand/site).
   > If they are **not present** when you build, render a clean **text wordmark** "IDBI BANK"
   > as an obvious **temporary placeholder**, and leave a `TODO: drop official logo into
   > assets/idbi-logo.svg`. Do **not** fabricate an emblem.
2. Replace the three render spots with an `<img>` (with `alt="IDBI Bank"` and width/height):
   - **Favicon** `index.html:8` → point `rel="icon"` at `assets/favicon.png`.
   - **Sidebar brand** `index.html:15-21` → swap the inline `<svg class="logo">…</svg>` for
     `<img class="logo" src="assets/idbi-logo.svg" alt="IDBI Bank" width="48" height="48">`.
   - **Welcome card** `app.js:694` → replace `<div class="w-logo">IDBI</div>` with the `<img>`;
     adjust `.w-logo` CSS (`style.css:470`) to frame an image cleanly (drop the text/gradient
     fill, keep a subtle tile/shadow if it looks good).
3. Keep sizing crisp on HiDPI; constrain with CSS, don't stretch. Preserve the logo's aspect
   ratio (no distortion).

**Acceptance:** real IDBI logo appears in the tab icon, sidebar, and welcome screen; no
hand-drawn SVG remains; nothing is stretched; placeholder path documented if asset missing.

---

## 4. Task 2 — Fix the "Saved" indicator (the "saved blur thing", point 2)

**Symptom:** the `#savedInd` "● Saved" pill (`index.html:86`) sits flush against the **＋ Add**
button in the glass topbar (`.topbar` is `.glass` with `backdrop-filter`, `style.css:80-81`),
uses low-contrast muted text (`.saved { color: var(--muted) }`, `style.css:297`), and on wide
gradient backgrounds it smears/overlaps the button and bleeds toward the window edge.

**Fix (goal: always crisp, never overlapping, never clipped):**
1. Make `#savedInd` a **self-contained solid chip** that doesn't depend on the blurred parent:
   give it a **solid** background (use the theme's `--surface` token — it's defined per theme
   at `style.css:435-440` precisely for "solid, non-blurred" surfaces), a 1px border
   (`--brd`), rounded corners, and `padding: 4px 10px`.
2. **Separate it from ＋ Add:** add `margin-left: 8px` (the topbar `gap` already gives 12px;
   ensure the chip never visually touches the button). Keep `flex: 0 0 auto` (`style.css:149`).
3. **Raise contrast:** text should be readable — normal text color when idle, the success
   green (`--ok`) dot when "Saved", amber (`--warn`) when "Saving…", red (`--bad`) when
   "Save failed". Keep the existing states from `setSaved()` (`app.js:179`).
4. **Stop the clipping at the right edge:** ensure the topbar never pushes the chip off-screen
   — let the search input shrink (`.search-wrap` already has `min-width:0`, `style.css:152`)
   and confirm the chip stays fully inside the viewport at common widths (1280–1920). Keep the
   existing rule that hides it under 1050px (`style.css:481`).
5. Optional polish (allowed): animate the dot subtly and let the chip fade from "Saving…" to a
   brief "Saved ✓" then a calm idle dot. Don't make it noisy.

**Acceptance:** at full width the "Saved/Saving…" chip is sharp, has clear breathing room from
＋ Add, never overlaps or clips, and its color reflects state.

---

## 5. Task 3 — Per-user workspaces (points 4 & 5 — the core fix)

**Problem today:** every user who opens "their workspace" actually loads **Amit's**
localStorage data, and the header literally says *"Amit Dethe · …"* (`app.js:286`). There is
only one workspace.

**Required behaviour**
1. The personal workspace shown is **`workspaces[currentUserId]`** (resolved via
   `workspaceOwnerId()`), seeded empty if new (§2.2). Under Superadmin **view-as**, it's the
   viewed person's real workspace (now possible because storage is central).
2. **Replace every "Amit" hardcode with the current owner's name:**
   - Title `app.js:286`: `"Amit Dethe · " + …` → `ownerName + " · " + …` where `ownerName =
     (person(workspaceOwnerId())||{}).name`.
   - `openWorkspace()` toast `app.js:677`: *"Opened {ownerName}'s workspace"*.
   - Sidebar `#ownerName` (`index.html:22`, set in `boot()` `app.js:771`) and welcome
     name already use the person — verify they track `currentUserId`.
3. **Each user creates & tracks their own** projects, request types, requests, tracked
   projects, tasks, and custom trackers — identical UI to today, but reading/writing **their**
   workspace object. The existing personal views/modals all read the global `PD`
   (`viewProject` `app.js:453`, `viewProjects` `:463`, `viewTasks` `:471`, `viewTracker`
   `:488`, `editRequest` `:526`, `editProj` `:532`, `editTask` `:537`, `editTracker(Row)`
   `:544-565`). Once `PD` points at the current user's workspace, these keep working — just
   make sure `savePersonal()` writes to the **server** (Task 5) and the local cache.
4. **No Edit-toggle / no permission needed for your own workspace.** A user always edits their
   own workspace freely (it's theirs). Keep **view-as** read-only (it's a preview) — block
   personal edits while `viewAs` is set, consistent with `app.js:711`'s guard.
5. **Make `projects` and `reqTypes` editable** (add "＋ Project tab" and a "Manage request
   types" modal, or an inline editor) so each person can "create their own options". Persist
   them in their workspace. Renaming/removing a project tab must not crash existing rows that
   reference it (guard lookups like `pProj()` `app.js:125`).

**Team Work Pendency note (already mostly correct):** the shared `workItems` list in
*Work Pendency* is filtered by `viewScope()` to *self + reports*, which is why view-as Shivesh
correctly shows only Shivesh's items. **Keep that.** Additionally, to honor "track their own
without special permission," let a user add/edit/delete Work-Pendency items **assigned to
themselves** without flipping the global ✎ Edit toggle (show the inline ✎/🗑 and an "＋ Add my
item" affordance for their own rows). Leave management of *other people's* items and the
people/org structure behind the existing Superadmin/Edit gating.

**Acceptance:**
- Map a 2nd PC's IP to Shivesh → from that PC, opening the workspace shows **"Shivesh Kumar ·
  …"** with **empty** seeded tabs, and items he adds persist and are visible from any PC.
- Superadmin *view-as Shivesh* shows Shivesh's **real** workspace (read-only).
- Amit's existing requests/tasks/trackers still appear in **Amit's** workspace only.

---

## 6. Task 4 — Open Links & Tech Contacts: shared + personal, no permission (point 3)

Two lists each, both **add/edit/delete without** the ✎ Edit toggle and **without** `canEdit`:

1. **Shared/team** Links & Tech Contacts = `data.json.links` / `data.json.vendors` (existing).
   Editable by **any** user on the LAN (mapped or not). Saved with the team save path.
2. **Personal** Links & Tech Contacts = `workspaces[uid].links` / `workspaces[uid].vendors`.
   Editable by that user only; saved with the workspace save path.

**UI:** In the **Links** tab (`viewLinks()` `app.js:433-440`) and **Tech Contacts** tab
(`viewVendors()` `app.js:442-448`), render **two clearly-labelled sections**:
*"Shared — team"* and *"Mine — {ownerName}"*, each with its own "＋ Add" and per-card
Edit/Delete. (If the user is an unmapped guest, show only the Shared section + a note that a
personal list appears once their PC is mapped.)

**Remove the gating for these four flows:**
- Delete the `guardTeam()` wrapper from `add/edit/del-link` and `add/edit/del-vendor`
  (`app.js:738-739`) and the `teamEdit && canEdit(null)` conditions inside `viewLinks` (line
  434, 439) and `viewVendors` (446, 447). Replace with: always show the controls; only block
  when `viewAs` is active (keep the read-only preview honest).
- `editLink()` / `editVendor()` (`app.js:586-595`) need a target argument so they write to
  **shared** vs **personal** (e.g. `editLink(id, scope)` where `scope ∈ {"shared","mine"}`),
  and save via the matching path (team save for shared, workspace save for personal).
- Keep `setSaved()` feedback on every save.

**Acceptance:** any user can add/edit/delete a **shared** link/contact with no Edit toggle and
no permission prompt; the same for their **personal** ones; shared edits are visible to
everyone, personal ones only to the owner; view-as stays read-only.

---

## 7. Task 5 — Server endpoints + concurrency safety (supports §2, §5, §6)

The current `POST /api/save` **replaces the entire `data.json`** with the client's TEAM object.
With many people now writing (their own workspaces, shared lists), a blind full overwrite can
**clobber** another user's concurrent change. Fix with **server-side read-modify-write merge**.

Implement in **both** `server.js` and `server.py` (mirror exactly; atomic temp→rename):

1. **`POST /api/workspace`** — body `{ "userId": "p-shivesh", "data": { ...workspace } }`.
   Server: read `data.json`, set `json.workspaces[userId] = data`, write atomically. Returns
   `{ ok:true }`. Used by `savePersonal()`.
2. **`POST /api/shared`** — body `{ "links": [...], "vendors": [...] }` (either/both).
   Server: read `data.json`, replace only the provided keys, write atomically. Used when
   editing shared Links/Contacts.
3. **Keep `POST /api/save`** for team structure (people, workItems, workTags, access, meta) but
   make it **merge, not clobber**: read the on-disk file first and **preserve** any keys the
   client didn't send — at minimum never drop `workspaces`, `links`, `vendors` that exist on
   disk. (Simplest: shallow-merge incoming top-level keys onto the on-disk object.)
4. Client wiring:
   - `savePersonal()` (`app.js:124`) → `POST /api/workspace` for `currentUserId`, **and** mirror
     to `localStorage` cache (key e.g. `idbi.dash.cache.<userId>`). Keep the debounced
     "Saving…→Saved" via `setSaved()`.
   - `saveTeam()` (`app.js:145-149`) → still `POST /api/save` (now merge-safe).
   - Shared link/contact edits → `POST /api/shared`.
   - `loadTeam()` (`app.js:141-144`) already pulls the whole `data.json` (so it gets
     `workspaces` too). On fetch failure, hydrate from the local cache.

**Concurrency invariant (must hold):** *No user's save may erase another user's workspace or
the shared lists.* If you prefer one generic endpoint over three, that's fine — but it must do
read-modify-write and satisfy this invariant.

**Acceptance:** two browsers editing different workspaces simultaneously both persist; neither
wipes the other; shared-list edits from one PC appear on the other after reload.

---

## 8. Task 6 — Migration & backups (no data loss)

1. **One-time migration on first load of the new build:**
   - If `localStorage["idbi.amit.dash.data.v1"]` exists **and** `data.json.workspaces["p-amit"]`
     is missing/empty, copy that localStorage object into `workspaces["p-amit"]` (mapping its
     `projects/reqTypes/requests/trackedProjects/tasks/trackers` across), set
     `meta.owner = "Amit Dethe"`, and save via `/api/workspace`. Then mark migration done
     (e.g. a `localStorage["idbi.migrated.v5"]="1"` flag) so it runs once.
   - Existing `data.json.links` / `data.json.vendors` **become the Shared lists** as-is.
2. **Backups must include everything.** `exportJSON()` (`app.js:661`) and `backupToDisk()`
   (`app.js:663-670`) already serialize `{ personal: PD, team: TEAM }`. Since `TEAM` now
   contains `workspaces`, all users' data is captured — **verify** the round-trip and that
   `importJSON()` (`app.js:662`) restores `workspaces` too. Update those functions if needed so
   restore repopulates per-user workspaces and both shared lists.
3. Write a fresh timestamped file into `backups/` (server already supports this) as part of
   testing, and confirm restore rebuilds the full multi-user state.

**Acceptance:** upgrading an existing install keeps Amit's data (now under his workspace);
Backup → wipe → Restore reproduces all workspaces + shared lists exactly.

---

## 9. Task 7 — Beautification & innovation (your latitude)

You may improve the look/feel freely **within the no-dependency, offline constraints**. Good
targets:
- A cleaner, more "official bank" topbar and section headers; consistent spacing/typography.
- Nicer empty states for new/empty workspaces (the "No links yet", "No tasks yet" cards).
- Subtle, tasteful motion (the existing glass/orb welcome is the vibe — don't overdo it).
- A small **"Switch workspace"** affordance for Superadmin to jump between people's workspaces
  (built on the existing view-as machinery, `app.js:137-139`, `247-249`).
- Keyboard niceties, better mobile/narrow layout (`@media` blocks start `style.css:481`).
- Accessibility: real `alt` text, focus styles, adequate contrast in all 5 themes
  (`THEMES`/`THEME_ACCENT` `app.js:50-51`).

**Don'ts:** no frameworks/CDNs/fonts-from-internet, no telemetry, no breaking the
`data-action` delegation pattern, no renaming data keys without a migration.

---

## 10. Final acceptance checklist (run all)

- [ ] Real IDBI logo in tab icon, sidebar, welcome; no fabricated SVG; no distortion.
- [ ] "Saved" chip is crisp, solid, spaced from ＋ Add, never overlaps/clips; states colored.
- [ ] Mapping a PC to Shivesh gives **Shivesh** an own, empty-seeded workspace; his data
      persists server-side and shows from any PC; **never** shows Amit's data.
- [ ] Amit's pre-existing requests/tasks/trackers migrated into **his** workspace only.
- [ ] Superadmin view-as shows each person's **real** workspace (read-only).
- [ ] Anyone can add/edit/delete **shared** Links & Tech Contacts with no Edit toggle / no
      permission; each person manages their **own** personal Links & Tech Contacts too.
- [ ] `projects`/`reqTypes` are editable per workspace ("create your own options").
- [ ] Concurrent edits from two browsers don't clobber each other (read-modify-write).
- [ ] `server.js` and `server.py` expose identical new endpoints with atomic writes.
- [ ] Backup → Restore reproduces all workspaces + both shared lists.
- [ ] `?v=10` → `?v=11`; app still launches 100% offline via `Start-Dashboard.bat`.

## 11. Quick file/line index (jump points)
- `MY_ID` / personal store: `assets/app.js:39`, `56-124`
- Owner/title hardcode: `assets/app.js:162`, `286`, `677`; `index.html:22`
- Identity/access/view-as: `assets/app.js:130-164`, `768-781`
- Links/Vendors views + modals + actions: `assets/app.js:433-448`, `586-595`, `738-739`
- Edit gating: `assets/app.js:275-278`, `679-680`, `711-712`
- Saved indicator: `index.html:86`; `assets/style.css:148-149,297-299`; `assets/app.js:179`
- Logo: `index.html:8,15-21`; `assets/app.js:694`; `assets/style.css:470`
- Servers: `server.js:23-110`, `server.py:27-95` (save at `server.js:34-51`, `server.py:46-54`)
- Cache-bust: `index.html:7,102,103`
