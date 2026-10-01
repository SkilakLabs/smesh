# Mesh

Mesh is a local outbound security gateway for supported AI requests.
It inspects requests routed through it and can block or redact detected
secrets, personal data, and organization-defined terms before forwarding.
Personal data covers email, phone, SSN and ITIN, payment cards, CVV and
PIN, bank routing and account numbers, IBAN, date of birth, passport and
driver's licence numbers, street addresses, and health-plan IDs; labels
or checksums keep ordinary numbers out.
Provider responses are relayed without content inspection.

[Download](https://mesh.skilak.ai/download) ·
[Documentation](https://skilak.ai/mesh/docs) ·
[Limitations](LIMITATIONS.md)

## Availability

Install Mesh with one command from the
[download page](https://mesh.skilak.ai/download). Supported AI tools are Claude
Code, Codex CLI, and Codex in VS Code. Read [Limitations](LIMITATIONS.md) before
relying on it.

## Choose an install route

The first public release is planned around direct installer scripts. Choose
your computer on the download page when downloads open; no GitHub account is
needed. Homebrew is an additional channel with separate qualification.

| Platform | Prerequisites | Route after publication |
|---|---|---|
| macOS, Apple Silicon (arm64) | macOS 13 or newer | `install.sh`; Homebrew only after that channel is separately qualified |
| macOS, Intel (x86-64) | macOS 13 or newer | `install.sh`; Intel Homebrew remains on hold |
| Windows, x86-64 | Windows 11; normal logged-in desktop account | `install.ps1` |
| Linux, x86-64 | Ubuntu 24.04 / glibc 2.39; systemd user session; system-provided `/usr/bin/setpriv` from util-linux | `install.sh`; Homebrew only after that channel is separately qualified |

There is no qualified Linux ARM, Windows ARM, 32-bit, older macOS, or Windows
MCP package. Other Linux distributions are not qualified merely because an
archive starts. WinGet community availability requires separate publication
and acceptance; no WinGet install command is offered here. The macOS DMG and
Windows setup EXE are held and are not part of the first release route.

### macOS

After downloads open, choose macOS on the download page and download
`install.sh`. The script selects the native CLI archive for Apple Silicon or
Intel, verifies it before activation, and prints the setup command to run.
Use your usual account, not root.

Homebrew will be an additional route for Apple Silicon only after its separate
channel passes qualification. Intel Homebrew remains held. Neither script
route installs the held Mac app or DMG.

Keep Gatekeeper and quarantine checks enabled. If macOS rejects the package,
stop and obtain a corrected verified release. Confirm the reported service
state after setup; installation alone does not prove login or restart behavior.

### Windows

After downloads open, download `install.ps1` from the Windows route and follow
the reviewed PowerShell command on the download page from your usual desktop
account. The script verifies the native x86-64 ZIP before activation. The
setup EXE remains held.

If setup changes the per-user PATH, open a new PowerShell window. Confirm
`smesh version` works before starting setup. Keep Windows security checks
enabled; the first CLI archive is unsigned and must pass the installer checks.
The native package registers a per-user background task.

### Linux

After downloads open, download `install.sh` from the Linux route. Open a
terminal in the folder where you saved it and run:

```sh
sh install.sh
```

The installer downloads and checks the native package automatically, then
prints the setup command to run. Use your usual account, not root. It installs
a systemd user service for background operation.

If `smesh` is not on PATH, use the exact launcher path printed by the
installer. It does not edit shell startup files. A working systemd user session
and the root-owned system `setpriv` executable are required for automatic
startup; installation alone does not prove login/restart behavior.

## First run with a verified native package

```sh
smesh init
smesh status
smesh client list
```

Guided setup presents the terms and license, selects policy and file handling,
previews supported client changes, and starts and health-checks the background
service. Use `smesh start` only for later recovery if the service is stopped.
On the Mac DMG route, opening the installed app starts the same guided flow.

Choose only the clients you intend to route. The automatic adapters are Claude
Code CLI and the shared Codex configuration used by Codex CLI and supported
IDE integrations. **Fully quit and restart every affected client.** A new chat
does not reload saved settings.

```sh
smesh client protection-state
```

A configured client and healthy service do not establish protection. The
command distinguishes recent inspected traffic, an unavailable route, and
unknown/not-verified state. Client relay and restart/login behavior still need
qualification for the exact installed package and client version.

This release supports Claude Code and Codex only. Installing Mesh does not
cover ChatGPT or Claude desktop and web apps, browser apps, or other agents.

## Daily use and maintenance

```sh
smesh status
smesh doctor
smesh logs
```

Before maintenance, restore approved clients to their direct routes:

```sh
smesh client pause
```

Follow the restart instructions and resolve any reported route conflict before
the gateway stops. Direct requests during a pause are outside Mesh. To start
the service and restore only the previously approved routes:

```sh
smesh client resume
```

### Upgrade

There is no working in-app upgrade or rollback command in the current beta.
`smesh upgrade check` reports unavailable metadata; `apply` and `rollback`
refuse. After a higher version is publicly released, use its verified installer
and the same package channel, then repeat setup/status and client checks.
Do not overwrite an installed version's files or substitute a development
build. Follow the release-specific migration instructions before changing
channels.

### Uninstall

Restore client routes while Mesh is still installed:

```sh
smesh client pause
smesh uninstall
```

For Homebrew, let the package manager remove its files after Mesh removes the
service and restores client routes:

```sh
smesh uninstall --keep-program-files
brew uninstall smesh
```

Windows also supports **Settings > Apps > Installed apps > Skilak Mesh >
Uninstall**. Wait for the delegated uninstaller and inspect its result.
A pending or retained item is not proof that removal completed.

By default, removal keeps user configuration, legal acceptance, and local
data. To deliberately remove that retained Mesh data, use
`smesh uninstall --purge-user-data` and review the confirmation. This does
not delete the AI clients or their provider accounts. Codex retains a marked
direct-provider entry for older conversations; `--purge-provider` removes
it only if you accept that those conversations may no longer resume.

Complete program removal, restart behavior, and a harmless repeated uninstall
remain release acceptance requirements. Report any retained program/service
item instead of deleting unfamiliar files manually.

## Troubleshooting

| Symptom | Next step |
|---|---|
| Download is unavailable | Try the download page again shortly. |
| `smesh` is not found | Use the launcher's printed path; on Windows open a new terminal after PATH changes; for the Mac app use **Command-Line Tools…**. |
| Service is unhealthy | Run `smesh doctor` and `smesh logs`; on Linux check the supported user session and setpriv prerequisite; on macOS check login-item approval. |
| Client is configured but not verified | Fully restart it, check `smesh client protection-state`, and verify the exact client/version using non-sensitive input. A saved route alone is insufficient. |
| `unknown_provider_alias` | Use the full `/p/openai/v1`, `/p/anthropic`, or reviewed `/p/gemini` path; a bare loopback origin is not a provider route. |
| Route conflict during pause or uninstall | Review the named setting. Use the documented disconnect conflict choice only after deciding whether to keep the newer route or restore the previous one. |
| MCP says the lightweight CLI cannot scan | Use a qualified native runtime containing the scanner on macOS or Linux. Windows MCP is unsupported. |
| Signature, policy, parser, or audit check fails | Stop and resolve the reported failure. Do not disable the check to obtain an allowed result. |

## Protection boundary

Only admitted requests that actually traverse Mesh are inspected. Direct
network requests, local file reads and shell commands need separate controls.
Managed device protection additionally requires independent prevention of
alternate egress. Provider responses remain uninspected.

Detection has false positives and false negatives. Unmarked confidential prose,
including some board and contract/pricing text, remains a known miss. Read
[Limitations](LIMITATIONS.md) before relying on a policy.

## License and terms

Mesh is proprietary software under the [Skilak Mesh License Agreement](LICENSE).
It is free for an individual's personal, non-commercial use, with no account or
activation. An organization may evaluate it for 30 days; any other company or
commercial use needs a written Company License from Skilak LLC. Skilak may grant
a no-fee Company License to an organization with fewer than 10 people, in
writing. The `LICENSE` file is authoritative.

- [Terms](TERMS.md)
- [Privacy](PRIVACY.md)
- [Commercial licensing](COMMERCIAL-LICENSE.md)
