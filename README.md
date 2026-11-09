# Back 4 Blood EAC fix for Xodus Nexus

Enables Back 4 Blood online play through Xodus Nexus on Linux by replacing
the EAC bootstrapper in the game's **temporary launch copy**.
Original files and other games are never touched.

**Confirmed working:** Back 4 Blood online co-op with EAC validated.

## The problem

MS Store games ship a **WinGDK** build of `EasyAntiCheat_EOS_Setup.exe`. The Proton EAC Runtime only recognizes the **Win64** (Steam) variant. Additionally, MS Store packages often lack `runtime.conf`, the signed file that authorizes EAC to run in Proton/Linux mode.

| Issue | Result |
|-------|--------|
| Missing `runtime.conf` | EAC never enters Proton mode → anti-cheat violation |
| GDK bootstrapper | "Unknown file version" → EAC validation fails → no online |

## How the fix works

1. **`runtime.conf` transplant** — Installs the EAC Proton authorization file (both store versions share the same deployment IDs)
2. **Scoped chain intercept** — Monitors `.xodus-chain-*` directories, validates B4B deployment IDs and executable hash, then atomically replaces the GDK bootstrapper with the Win64 variant

```
Xodus validates (GDK, 962560 bytes) ✓
    → creates chain → copies EAC exe
        → watcher validates IDs + hash → swaps GDK → Win64
            → game reads Win64 → Proton EAC Runtime validates ✓
                → online play works ✓
```

## Requirements

- Linux with Xodus Nexus and Back 4 Blood installed
- Python 3.8+

No pip packages or `inotify-tools` needed. The watcher uses Python's standard library and Linux inotify directly.

## Install

```sh
git clone https://github.com/hortelaa/xodus-b4b-fix.git
cd xodus-b4b-fix
python3 installer.py --game-dir '/path/to/Xodus Nexus/Back-4-Blood' --accept-bundled-files
```

The installer copies bundled fix files and creates a menu shortcut. It does not start the watcher or game. Do not run with sudo.

## Start and stop

1. Open **B4B EAC Watcher** from your application menu
2. Wait for **READY**, then launch Back 4 Blood through Xodus Nexus
3. Keep the watcher open while playing. Press **Ctrl+C** to stop

Or start from a terminal:

```sh
sh '/path/to/Xodus Nexus/b4b-eac-fix.sh'
```

## Remove

Stop the watcher and close the game, then delete:

- `b4b-eac-fix/` and `b4b-eac-fix.sh` beside your `Back-4-Blood` folder
- `~/.local/share/applications/b4b-eac-fix.desktop`

Remove `Back-4-Blood/EasyAntiCheat/Certificates/runtime.conf` only if this installer created it.

## Technical details

Both store versions share identical EAC deployment identifiers:

```
productid:    3bc138d96fb24d9597d4bbff23b9ecee
sandboxid:    57dfd95548214a138218e56cd9e5b9d8
deploymentid: e052f3c4fce741bab494ed0304eeaf62
```

The watcher validates all three IDs, the game executable marker, and a recognized input hash before replacing anything. Unknown versions and other games are left untouched.

- [Technical guide](TECHNICAL.md): safety checks, diagnostics, tests and limitations
- [Changes](CHANGES.md): what changed from the original implementation
- [Integration proposal](PR_PLAN.md): how this could be integrated into Xodus Nexus

## License

Code: [MIT](LICENSE). Bundled third-party files retain their own licensing terms.
