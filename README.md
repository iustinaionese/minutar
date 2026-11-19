# Minutar

[Română](README.ro.md)

**An automatic time tracker for Windows. 100% local, no cloud.**

Minutar records on its own where your time goes (apps, websites, projects), shows
everything in a local dashboard and nudges you politely when you drift into
distractions. Your data never leaves your computer: see [PRIVACY.md](PRIVACY.md).

> The interface is in Romanian for now. An English UI is planned.

## What it does

- **Automatic tracking**: the active window, app and title, with AFK detection (breaks
  are not counted). Videos playing with sound count as active time even when you don't
  move the mouse.
- **Time per project**: by keywords in window titles, domains, dedicated apps, browser
  profiles (each Chrome or Edge profile can be a different client) and **Claude Code
  sessions per project** (two metrics: "attention" and "claude-work").
- **Productive / neutral / unproductive classification**, with rules you edit in the
  dashboard, plus exceptions for educational YouTube (title keywords or allowlisted
  channels).
- **Anti-distraction popup**: it shows up when you spend too long on something
  unproductive. You can snooze it, confirm you know what you're doing, or close it. The
  message rotates and is written with warm humour, not as a scolding.
- **Phone time**: iPhone Screen Time, imported by hand. You take screenshots, give them to
  your own LLM (any account, free ones included) and paste the JSON into the "Phone" page.
  The app talks to no LLM and needs no API keys. Each app is classified once and appears
  in the report next to your PC time, in a separate block.
- **Local dashboard** ([http://localhost:5601](http://localhost:5601)): day, week and month
  views, timeline, hourly heatmap, breakdowns by class, project, app and domain, a daily
  journal, focus score, streaks, and minute-level reallocation of time between projects.
- **Focus mode**: strict work windows that block unproductive sites.
- **100% local**: SQLite storage in `%LOCALAPPDATA%\TimeTracker`, automatic nightly
  backups, and a server that listens only on localhost. The optional integrations (a
  Telegram briefing and Google Calendar) are off by default.

---

## Install (recommended, for everyone)

1. Open the **[Releases](https://github.com/iustinaionese/minutar/releases/latest)** page
   and download **`Minutar-Setup-x.y.z.exe`**.
2. Double-click it.
   - Windows may show a blue **"Windows protected your PC"** screen (the app is new and not
     signed yet). Click **More info → Run anyway**.
   - The installer also installs the **.NET Desktop Runtime** if it is missing (once).
3. Done. The **Minutar** icon appears next to the clock (system tray), the dashboard opens
   at **[http://localhost:5601](http://localhost:5601)**, and the app starts on its own at
   every sign-in.

Requirement: **Windows 10 (22H2) or Windows 11**. Nothing else to install.

### Configuration (projects, rules)

Open the dashboard and go to the **Settings** tab: add your projects, the classification
rules (what is productive or unproductive) and the thresholds. Everything is saved locally
in `%LOCALAPPDATA%\TimeTracker\tracker.toml`.

### Browser extension (Chrome / Edge), recommended

The extension reports the active site and tab, so the app knows which project you are
working on and applies the video rule correctly. The installer already puts it on disk;
load it like this:

1. Open `chrome://extensions` (or `edge://extensions`).
2. Turn on **Developer mode** (the switch at the top right).
3. **Load unpacked** and pick the folder `%LOCALAPPDATA%\time-tracker\extension`
   (paste the path into the Explorer address bar to find it quickly).
4. Click the extension icon, then **Options**, and type a name for the profile (for example
   "work" or "personal"). Put the same name under **browser_profiles** in the matching
   project on the Settings page, so everything you do in that profile counts toward it.
5. Repeat in every browser profile you use.

> The extension is coming to the Chrome Web Store and Edge Add-ons. It will then install
> in one click, without the steps above.

### Uninstall

From **Windows Settings → Apps → Minutar → Uninstall** (your data is kept). To delete your
data too, run the uninstaller with `-RemoveData` (see below).

---

## Build from source (for developers)

Requirements: Windows 10 (22H2) or 11,
[.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0),
[Node.js LTS](https://nodejs.org).

```powershell
git clone https://github.com/iustinaionese/minutar.git
cd minutar

# build the UI (once per version)
cd src/dashboard && npm ci && npm run build && cd ../..
cd src/extension && npm ci && npm run build && cd ../..

# install and start (PowerShell AS ADMINISTRATOR)
powershell -ExecutionPolicy Bypass -File .\scripts\install.ps1
```

When built from source, the extension loads from `src\extension\dist`.

Claude Code hooks (optional, for the per-project metrics):
`node hooks\install-claude-hooks.mjs`.

Uninstall from source:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\uninstall.ps1             # keeps your data
powershell -ExecutionPolicy Bypass -File .\scripts\uninstall.ps1 -RemoveData # deletes everything
```

## Architecture (short)

| Component | Role |
|---|---|
| `Tracker.Watcher` (.NET 10) | active window + AFK → REST heartbeats |
| `Tracker.Daemon` (.NET 10) | own SQLite store, rules engine, WPF popup, coach, API + dashboard |
| `Tracker.Supervisor` (.NET 10) | tray, watchdog with restart, autostart |
| `src/dashboard` (React + Vite) | the dashboard, served by the daemon on :5601 |
| `src/extension` (MV3, TypeScript) | URL, title, audio and YouTube channel per profile → daemon |

## Status

Working MVP, used daily. Per-user installer ready (no SDK needed). Next: the extension in
the Chrome Web Store and Edge Add-ons, code signing (removes the SmartScreen warning), and
an English UI.

## License

[MIT](LICENSE) © 2026 Iustin Aionese
