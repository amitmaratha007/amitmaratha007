# IDBI MIS — Work Command Center

A personal, **macOS "Liquid Glass"** style work dashboard for Amit Dethe.
Runs on **Node.js**, opens in **Chrome**, and works **100% offline** (no internet, no CDNs).

## ▶️ How to run
**Double-click `Start-Dashboard.bat`.**
- It starts a tiny local server (Node, or Python as fallback) and opens the dashboard in Chrome.
- Keep that black window open while you use the dashboard; close it to stop.
- If Windows asks about the firewall the first time, click **Allow access**.

Open it at **`http://localhost:8080`** (this PC) — always open it the **same way** so your saved data is there.

## 🧊 What's inside
- **Left glass panel** — live **analog clock**, today's date, and a **calendar pre-loaded with Maharashtra 2026 holidays** (dots on holidays, "next holiday in N days", click a holiday to see its name).
- **Overview** — KPIs (open / overdue / due ≤7 days / active projects / open tasks), open-requests-by-project bars, upcoming deadlines, project status, tasks due soon.
- **4 project tabs** — **Oracle GL · ASP-GSP (GST) · GST Litigation Tool · Miscellaneous Work**, each with **4 sub-tabs**: **Firewall CRF · i-Smart Request · Service Request · Change Request**. Every entry tracks Raised date, **Deadline (date picker)**, status, owner/approver, **days pending** and a **deadline countdown** (Due in Nd / Due today / Overdue Nd).
- **Projects** tab — track **UAT date, Go-Live date, current status** and progress %.
- **Tasks** tab — quick to-dos with priority, project tag and due date (check to complete).
- **Team** — your MIS contacts (read from `data.json`).
- **Search** (Ctrl + K), light/dark theme, accent colour picker.

## 💾 Your data is safe
- Everything you add is **saved automatically in the browser (localStorage)**.
- **Re-generating / updating the code does NOT erase your data** — the app only seeds samples on the very first run and keeps all your records afterwards.
- Extra safety nets (bottom-left of the sidebar):
  - **⬇ Backup** — download a `.json` backup file.
  - **⬆ Restore** — load a backup file back in.
  - **💾** — save a timestamped copy to the `backups/` folder on disk (needs the server running).

> Tip: data is tied to the address you use. To move data to another PC, use **Backup** here and **Restore** there.

## ✏️ Customise
- **Holidays:** edit `assets/holidays.js` (please **verify festival dates** against the official RBI / Maharashtra 2026 list — lunar dates can shift a day).
- **Starting sample data / project names:** edit the `DEFAULTS` object in `assets/app.js`.
- **Port:** change `set "PORT=8080"` in `Start-Dashboard.bat`.

## Files
| File | Purpose |
|------|---------|
| `Start-Dashboard.bat` | Click to run everything (Node → Python fallback, opens Chrome) |
| `index.html`, `assets/style.css`, `assets/app.js` | The dashboard (glass UI + logic) |
| `assets/holidays.js` | Maharashtra 2026 holiday list (editable) |
| `data.json` | Team contacts for the Team tab |
| `server.js` / `server.py` | Offline web server + `/api/backup` |
| `backups/` | Timestamped disk backups (created on demand) |
