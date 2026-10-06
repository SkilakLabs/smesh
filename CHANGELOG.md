# Skilak Mesh changelog

What changed in each public release of Skilak Mesh. Install or upgrade from the
[download page](https://mesh.skilak.ai/download), or run `smesh upgrade check`.

## 0.0.12

### Fixed

- **Windows:** `smesh restart`, `smesh stop` and `smesh start` no longer fail
  with "Windows task instance query failed" when Task Scheduler briefly reports
  the gateway as queued or starting, or when the first start on a new computer
  is slow. Mesh now waits up to a minute for the gateway to start, and checks
  its status faster.
- **Windows:** installation no longer fails its protection check when Windows
  briefly holds Mesh's local log file open. Lasting logging failures still fail
  the check.
- Conversation history sent again after an update is checked with the current
  detectors, so detection fixes also protect resent history. The first request
  after updating may not reuse your AI provider's prompt cache.

### Changed

- `smesh status` now shows the reminder when your 30-day evaluation has ended,
  as the home screen already does. Protection keeps running.
- `smesh uninstall --purge-user-data` on a computer where setup never ran now
  says "Nothing to purge: Mesh was never set up here." and succeeds. Data Mesh
  can't recognize as its own is still left untouched.
- Support bundles (`smesh support-bundle`) no longer include hostnames,
  home-folder paths or anything that looks like a credential, and they say when
  a bundle is safe to attach to an issue.

## 0.0.11

### New

- `smesh upgrade check` tells you whether a newer version is available, and
  `smesh upgrade apply` installs it. The installer's checksum and size are
  verified first, and your settings and connected tools are kept.
- `smesh terms show terms`, `privacy`, `license` or `company` prints the full
  text of that document, offline, without accepting anything.

### Changed

- Running the installer over an existing installation now tells you which
  version it replaced and whether Mesh restarted.

### Known issues

- **Windows:** `smesh restart` can occasionally fail with "Windows task
  instance query failed". Run it again. Fixed in 0.0.12.

## 0.0.10

### New

- **Simpler setup.** `smesh init` uses arrow-key menus with the recommended
  choice already selected, and connects your AI tools in one step. It asks how
  you'll use Mesh; company use points you to hello@skilak.ai.
- **Ready to use after install.** On macOS and Linux, the installer adds Mesh
  to your PATH, removed again on uninstall (opt out with `--no-modify-path`),
  and finishes by running `smesh init`.
- **Report a problem easily.** A failing `smesh doctor`, a crash, or
  `smesh --help` shows where to report it, and the GitHub issue forms ask for
  what we need.
- When your 30-day evaluation ends, Mesh reminds you on the home screen, in
  `smesh license status` and during setup. Protection never stops.

### Fixed

- **Windows:** `smesh dashboard status` could report the dashboard as stopped
  while it was running.
- The support bundle now lists exactly what it collects.

## 0.0.9

First public release.

- Mesh is a local security gateway for AI coding tools. It inspects the
  requests your tools send and can block or redact secrets, personal data and
  terms your organization defines before they leave your computer.
- Works with Claude Code, Codex CLI and Codex in VS Code.
- Runs on macOS 13 or newer (Apple Silicon and Intel), Windows 11 (x86-64) and
  Ubuntu 24.04 (x86-64).
- Installs with one command from the download page; no account is needed. Read
  [Limitations](LIMITATIONS.md) before relying on it.
