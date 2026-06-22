# Changelog

## 2026.06.21

### What Changed
- Added a **Plasma / KDE branch** to `Functions.py::_get_logout()`. Plasma was unhandled, so logout fell
  through to the generic `pkill <name>` fallback and ran **`pkill plasma`** — which only kills
  `plasmashell`. On Plasma 6 Wayland the compositor (`kwin_wayland`) and the rest of
  `graphical-session.target` kept running, holding **DRM-master**, so the next login could not drive the
  GPU and landed on a **black screen** (recoverable only by reboot).

### Technical Details
- New branch matches `plasma` / `plasmawayland` / `plasmax11` / `kde` (and the session-file paths) and
  returns `qdbus6 org.kde.Shutdown /Shutdown org.kde.Shutdown.logout` — Plasma 6's native logout, which
  works on both X11 and Wayland and stops `graphical-session.target` cleanly so `kwin_wayland` releases
  DRM-master.
- Diagnosed live on a Plasma box: after `pkill plasma`, `kwin_wayland` still held `/dev/dri/card1` and
  the new session logged `atomic commit failed: Permission denied`. The Kiro `systemd-logind` drop-in was
  ruled out (failure reproduced without it).
- flake8 + ruff clean (max line length 120).

## 2026.06.16

### What Changed
- Ported the ArchLinux Tweak Tool startup UTF-8 guard into **both** binaries — `archlinux-logout` and
  `archlinux-betterlockscreen` — so neither crashes on a non-UTF-8 system locale (latin-1 `fr_BE`,
  etc.). On a UTF-8 locale (incl. `fr_FR.UTF-8`/`it_IT.UTF-8`/`es_ES.UTF-8`) the guard is inert; the
  apps were already robust there. Part of the ecosystem-wide UTF-8 robustness audit of all Kiro GTK4
  apps.

### Technical Details
- `archlinux-logout.py` + `archlinux-betterlockscreen.py`: two blocks at the top of each entry point.
  (1) Re-exec with `-X utf8` + `PYTHONUTF8=1` only when `codecs.lookup(sys.getfilesystemencoding()).name
  != "utf-8"` — forces UTF-8 for stdout, `text=True` subprocess decoding and `open()` regardless of
  `LANG`; loop-safe. (2) When the current locale is not UTF-8, fall back to `C.UTF-8` so spawned child
  output stays readable. `codecs`/`os`/`sys` imports deduplicated into the guard; later imports carry
  `# noqa: E402`.
- The fullscreen-overlay (`archlinux-logout`) was given extra scrutiny since re-exec interacts with its
  monitor grab — re-exec happens before any GTK work, so the relaunched process builds the overlay
  cleanly. ruff + `py_compile` clean on both; re-exec verified under `nl_BE.iso88591`.

### Files Modified
- usr/share/archlinux-logout/archlinux-logout.py
- usr/share/archlinux-betterlockscreen/archlinux-betterlockscreen.py
- CHANGELOG.md

## 2026.05.31

### What Changed
- Added a standalone **settings mode** so ArchLinux Logout can be configured from the XFCE Settings Manager without launching the fullscreen power overlay. New `archlinux-logout --settings` opens just the configuration panel in a small decorated window; ships a new `.desktop` entry ("ArchLinux Logout Settings") with XFCE Settings categories.
- Added an **Exit** button to the standalone settings window (not shown in the overlay popover, which already has Cancel).
- Fixed a latent infinite-recursion bug in the save path (`_after_save` calling itself) that a global replace had introduced — would have broken Save in the overlay popover too.
- **archlinux-betterlockscreen**: replaced the `panel.png` image-as-background behind the in-app notification ("Lockscreen set successfully") with a CSS-styled bar, matching the fix already shipped in ArchLinux Tweak Tool. The notification no longer renders a stretched bitmap behind the text.

### Technical Details
- `GUI.py`: extracted the settings widget tree out of the overlay popover into a shared `SettingsPanel(self, Gtk, fn)` builder, used by both the popover (overlay mode) and the new settings window.
- `archlinux-logout.py`: `TransparentWindow.__init__` takes `settings_only`; in that mode it builds a small decorated window via `_build_settings_window()` and returns early — skipping fullscreen, monitor detection, background CSS, and async pixbuf loading. The power keybinds are **not** wired in settings mode so pressing "S"/"R"/etc. can't trigger a real action.
- The `--settings` branch sits at the `__main__` lock-file guard and bypasses the singleton entirely (never creates `/tmp/archlinux-logout.lock`), so it opens even while the overlay is up or after a stale-lock crash. Flag is parsed from `sys.argv` directly (not via GApplication) and the app runs with `app.run(None)`.
- `on_save_clicked` now calls `_after_save()`, which pops down the popover in overlay mode or closes the window in settings mode.
- Launcher `usr/bin/archlinux-logout` now forwards `"$@"`.

### Files Modified
- usr/share/archlinux-logout/GUI.py
- usr/share/archlinux-logout/archlinux-logout.py
- usr/bin/archlinux-logout
- usr/share/applications/archlinux-logout-settings.desktop (created)
- usr/share/archlinux-betterlockscreen/GUI.py
- CHANGELOG.md

## 2026.05.21

### What Changed
- Added the four other required markdown scaffold files (created stubs for whichever of `IDEAS.md` / `TODO.md` / `CLAUDE.md` were missing) per the new ecosystem MD-scaffold rule codified in [Kiro-HQ/CLAUDE.md](/home/erik/Insync/Kiro/Kiro-HQ/CLAUDE.md#required-markdown-scaffold-every-repo). README was already substantial; left untouched.

### Files Modified
- CHANGELOG.md
- IDEAS.md (created where missing)
- TODO.md (created where missing)
- CLAUDE.md (created where missing)

## 2026.05.03

**What Changed**
Full code quality and GTK4 correctness pass across both apps. Added session-agnostic pkill fallback, fixed GTK3 deprecated API usage, corrected signal handler signatures, fixed GLib timeout double-remove bug, and enforced project coding rules throughout.

**Technical Details**
- `_get_logout()` pkill fallback: if no named DE matches, extracts bare name from `DESKTOP_SESSION`/`XDG_CURRENT_DESKTOP` and returns `"pkill <name>"` — makes logout work on any unrecognised WM
- `__exec_cmd` now spawns `subprocess.Popen` in a daemon thread instead of blocking `os.system()` call
- `subprocess.call()` in `archlinux-betterlockscreen.py::set_lockscreen` replaced with `Popen().wait()`
- All `Gtk.Image` + `set_from_pixbuf()` (GTK3 deprecated) replaced with `Gtk.Picture` + `set_paintable(Gdk.Texture.new_for_pixbuf())` across betterlockscreen GUI.py, Support.py, Splash.py
- `btnsearch`/`btndefault` `.connect("clicked", handler, self.fb)` fixed — GTK4 `clicked` only emits the widget; extra `self.fb` arg was silently dropped at runtime causing `fb` param to always be `None`
- Dead `fb` parameter removed from `on_load_clicked` and `on_default_clicked`
- `timeOut()` now returns `False` to signal one-shot; `close_in_app_notification()` no longer calls `GLib.source_remove()` (double-remove caused GLib warning)
- Unused `subprocess`, `threading` imports removed from betterlockscreen `Functions.py`
- Unused `os` import removed from betterlockscreen `GUI.py`
- All unused callback `widget` params renamed to `_widget` across both apps per GTK4 convention
- `Gtk.AlertDialog` (GTK 4.10+) confirmed safe — Arch Linux ships GTK 4.16+, no version guard needed

**Files Modified**
- `usr/share/archlinux-logout/Functions.py`
- `usr/share/archlinux-logout/archlinux-logout.py`
- `usr/share/archlinux-betterlockscreen/archlinux-betterlockscreen.py`
- `usr/share/archlinux-betterlockscreen/GUI.py`
- `usr/share/archlinux-betterlockscreen/Support.py`
- `usr/share/archlinux-betterlockscreen/Splash.py`
- `usr/share/archlinux-betterlockscreen/Functions.py`

---

## 2026.04.16

**What Changed**
README fully rewritten with comprehensive documentation: installation, dependencies, features, theming, keyboard shortcuts, session support (X11/Wayland), troubleshooting, and links to video tutorials.

**Technical Details**
Content-only change; no code modified.

**Files Modified**
- `README.md`

---

## 2026.04.14

**What Changed**
Added the `neocandy` theme.

**Technical Details**
Full set of 14 SVGs (7 actions × normal + blur variant) plus `theme.css` added under `usr/share/archlinux-logout-themes/themes/neocandy/`.

**Files Modified**
- `usr/share/archlinux-logout-themes/themes/neocandy/` (15 new files)

---

## 2026.04.13

**What Changed**
Settings popover gained per-button visibility checkboxes. Removed monitor-geometry-based auto-scaling of icon sizes in favour of fixed defaults.

**Technical Details**
- Added a `Gtk.Grid` of `Gtk.CheckButton` widgets (one per action: cancel, shutdown, restart, suspend, hibernate, lock, logout) to the settings popover.
- Save action now serialises the checkbox state back to `buttons=` in the config file.
- Removed `_set_scaled_icon_sizes()` which computed sizes from monitor resolution; defaults set to 80px main / 32px aux.

**Files Modified**
- `usr/share/archlinux-logout/GUI.py`
- `usr/share/archlinux-logout/archlinux-logout.py`

---

## 2026.04.09

**What Changed**
Minor config default update and session-detection fix in the main window.

**Technical Details**
Config default value adjusted; 7 lines changed in the main window (likely Wayland/X11 session detection path).

**Files Modified**
- `etc/archlinux-logout.conf`
- `usr/share/archlinux-logout/archlinux-logout.py`

---

## 2026.04.08

**What Changed**
Refinements to DE detection logic and monitor handling.

**Technical Details**
- `Functions.py`: cleaned up redundant DE detection code.
- `archlinux-logout.py`: improved multi-monitor detection path.
- Minor GUI/main fixes.

**Files Modified**
- `usr/share/archlinux-logout/Functions.py`
- `usr/share/archlinux-logout/GUI.py`
- `usr/share/archlinux-logout/archlinux-logout.py`

---

## 2026.04.06

**What Changed**
Initial project commit and extensive iterative development in a single day. Established the full application architecture: GTK4 logout overlay + betterlockscreen companion app.

**Technical Details**
- `_get_logout()` split into `_detect_desktop()` (env var + process-based detection) and a command-lookup function; added `pgrep`-based fallback for ohmychadwm/chadwm.
- GUI refactored around a `make_card()` factory: each button is a `Gtk.Box` card that owns its own `GestureClick` and `EventControllerMotion` controllers, replacing an earlier approach where gesture targets were separate from the visual widget.
- Icon loading moved to a background thread (`threading.Thread`) with `GLib.idle_add(_build_gui)` so the dark overlay appears immediately while SVGs load asynchronously.
- `set_widget_pixbuf()` helper added to handle GTK4 `Gdk.Texture`/pixbuf API differences.
- `GUI.py` reduced to a pure UI builder (no logic); all state and signal handling stays in `TransparentWindow`.
- betterlockscreen companion app restructured with the same three-file split.

**Files Modified**
- `usr/share/archlinux-logout/archlinux-logout.py`
- `usr/share/archlinux-logout/GUI.py`
- `usr/share/archlinux-logout/Functions.py`
- `usr/share/archlinux-betterlockscreen/archlinux-betterlockscreen.py`
- `usr/share/archlinux-betterlockscreen/Functions.py`
- `usr/share/archlinux-betterlockscreen/GUI.py`
- `usr/share/archlinux-betterlockscreen/Splash.py`
- `usr/share/archlinux-betterlockscreen/Support.py`
- `etc/archlinux-logout.conf`
- `README.md`
