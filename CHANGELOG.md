# Changelog

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
