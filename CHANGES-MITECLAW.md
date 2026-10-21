# MiteClaw changes to OfficeCLI

This repository is a fork of [iOfficeAI/OfficeCLI](https://github.com/iOfficeAI/OfficeCLI)
(Apache-2.0). MiteClaw bundles the OfficeCLI binary in its desktop installer. The branch
`miteclaw/main` starts at the upstream tag `v1.0.147`
(`b94f3906fd52d450c64f8e40370e376b9e15079e`) and carries the changes below. `LICENSE`,
`NOTICE` and `THIRD-PARTY-NOTICES.txt` are unchanged; each modified file says so in its
header.

## 1. No self-update, no bare-invocation self-install (2026-10-21)

Upstream OfficeCLI updates itself: on every run (`Program.cs` → `UpdateChecker.CheckInBackground`,
and every hour under `officecli mcp`) it swaps in a staged `officecli(.exe).update`, refreshes
the officecli skill files in other agents' folders when its version changed, and once a day
starts `officecli __update-check__`, which asks `https://d.officecli.ai` (then GitHub) for the
latest release, downloads it and stages it next to the binary. A bare `officecli` with no
arguments also copies itself to `%LOCALAPPDATA%\OfficeCli` (`~/.local/bin` elsewhere) and
installs skills and MCP entries for other tools.

MiteClaw ships one pinned binary whose SHA256 its installer checks, so the binary must not
replace itself or install itself elsewhere.

| File | Change |
| --- | --- |
| `src/officecli/officecli.csproj` | Defines `MITECLAW_NO_SELF_UPDATE`; `InformationalVersion` is `<version>-miteclaw.1`, so `officecli --version` prints `1.0.147-miteclaw.1`. |
| `src/officecli/Core/UpdateChecker.cs` | With `MITECLAW_NO_SELF_UPDATE`, `CheckInBackground` and `RunRefresh` return at once: no network request, no staged-update swap, no skill refresh, no `~/.officecli/config.json` write. |
| `src/officecli/Core/Installer.cs` | With `MITECLAW_NO_SELF_UPDATE`, `MaybeAutoInstall` returns at once. The explicit `officecli install` command is unchanged. |
| `global.json` | New: pins the .NET SDK used for MiteClaw builds (10.0.402, later patches allowed). |

Everything else (commands, output formats, `config`, `install`, `mcp`, `skills`, plugins,
`watch`) is upstream code. Generated HTML previews still reference KaTeX and three.js on
`d.officecli.ai` / jsDelivr, and diagram rendering can fetch mermaid.js from them; MiteClaw's
preview page blocks those loads with its Content-Security-Policy.

## Build

```sh
dotnet publish src/officecli/officecli.csproj -c Release -r win-x64 -o publish --nologo
```

(the upstream release command from `.github/workflows/build.yml`; other runtime ids as there).
