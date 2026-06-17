# EggShell (fork) — Python 3 port

> Fork of [neoneggplant/eggshell](https://github.com/neoneggplant/eggshell) by Lucas Jackson.
> The upstream README (original author's, untouched) is in [README.md](README.md). This file documents *this fork*.

## What the project is

EggShell is a post-exploitation / remote-surveillance tool written in Python. The
operator runs a listener on their own machine; a payload executed on a target
device (jailbroken iOS, macOS, or Linux) calls back over an SSL socket and opens
a remote command session. From that session you get a shell plus device-specific
commands — file upload/download, camera, mic, location, SMS/contacts/notes dumps,
passcode retrieval, persistence, and so on (full list in the upstream README).

Original author wrote it for Python 2.7.

## The backstory

I cloned this in 8th grade. My school pushed a locked-down MDM **configuration
profile** onto the school iPad — the kind that restricts apps, blocks settings,
and can't be removed from the normal UI. The iPad was jailbroken, so the goal was
to get a real shell on the device and reach the filesystem where those profiles
live, instead of being stuck behind the locked Settings screen. EggShell was the
tool that gave that on-device session.

## What I actually did to it

The repo shipped as Python 2.7. I ported the whole thing to **Python 3** with a
bulk find-and-replace pass rather than editing ~100 module files by hand:

- [`recipe-277753-1.py`](recipe-277753-1.py) — the ActiveState "search/replace
  across many files" recipe, run from the CLI.
- [`replace.py`](replace.py) — the same logic wrapped as a callable
  `searchreplace()` for scripting the pass over the `modules/` tree.

The transformations that pass applied, visible in the (now-removed) `.bak`
backups it left behind:

| Python 2 | Python 3 |
|----------|----------|
| `print X` | `print(X)` |
| `raw_input(...)` | `input(...)` |
| `import helper` / `from multihandler import …` | `from . import helper` / `from .multihandler import …` |

So the work was: **a mechanical-but-real Py2→Py3 migration of a ~100-file
codebase, driven by a reusable replace script**, in service of getting the tool
running on a current Python to free a school iPad from its MDM profile.

## Cleanup done in this pass (2026)

- Removed 56 leftover `.bak` files the find/replace tool created (167 → 111
  tracked files). They were pure backups, no longer needed once the port landed.

## Layout

- `eggshell.py` — entry point / main menu.
- `modules/` — server, session, multihandler, payload generators, and per-command
  modules grouped `iOS/`, `macOS/`, `universal/`, `local/`.
- `resources/` — prebuilt iOS/macOS payload binaries and the `espl` bootstrap.
- `src/` — Objective-C / Theos source for the iOS (`esplios`, `espro`) and macOS
  (`esplmacos`) native payload components.

## Notes / caveats

- Proof-of-concept security tooling. Upstream license is GPL-2.0; intended for
  devices you own — see the disclaimer in [README.md](README.md).
- Two stray duplicate command files (`setvol.py`, `getfacebook.py`) sit next to
  their `_ios`/`_macos` counterparts. Left in place — the loader keys off the
  `_ios`/`_macos`/`_universal` suffixes, so the bare ones are inert dupes, not
  worth the risk of pruning blind.
