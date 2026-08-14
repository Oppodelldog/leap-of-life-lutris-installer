# Leap of Life — Lutris installer

An unofficial [Lutris](https://lutris.net) install script for
**[Leap of Life](https://oppodelldog.itch.io/leap-of-life)** by Oppodelldog — a cozy stealth
game about protecting the next generation of frogs, made for the
[Summer Frog Jam 2026](https://itch.io/jam/summer-frog-jam-2026).

The game ships as a Windows-only Unreal Engine 5 build. This script sets up a 64-bit Wine
prefix, unpacks the game into it and registers it with Lutris.

## Install

1. Download `leap-of-life-windows.zip` (393 MB) from the
   [itch.io page](https://oppodelldog.itch.io/leap-of-life).
2. Run:

   ```bash
   lutris -i https://raw.githubusercontent.com/horroko-dev/leap-of-life-lutris-installer/main/leap-of-life.yaml
   ```

3. When the installer asks for a file, point it at the zip from step 1.

Lutris downloads and runs the script directly from the URL — no lutris.net account and no
installer moderation involved.

## Tested

Full install and a play session on 2026-08-14:

| | |
| --- | --- |
| Lutris | 0.5.22 |
| Wine | system Wine 11.13 (no version pinned in the script) |
| GPU / driver | AMD Radeon 780M, amdgpu 26.1.5, Xwayland |
| Renderer | DX12 through vkd3d-proton — no `-dx11`/`-vulkan` fallback needed |
| Engine | Unreal Engine 5.8 |
| Install size | 939 MB in the game directory (prefix included) |

The game runs and exits cleanly. Lutris tracks it correctly: `FrogSniper.exe` is only a
launcher stub, and both it and the `FrogSniper-Win64-Shipping.exe` it spawns stay children of
`lutris-wrapper`, so Lutris notices when the game quits.

Harmless noise in the log on first start: `Failed to map read-only cache: vkd3d-proton.cache`
(the shader cache does not exist yet) and `readMonitorEdidFromKey` / `DXGI: Failed to parse
display metadata` under Xwayland.

### Why you have to download the zip yourself

itch.io serves downloads only behind a CSRF-protected `POST`, so there is no stable direct URL
that a Lutris script could fetch. The script therefore uses Lutris' `N/A:` form, which asks you
to select the file. If a stable mirror URL under the author's control ever exists, the change is
a single line:

```yaml
  files:
  - installer: https://example.org/leap-of-life-windows.zip
```

## What the script does

| Step | Detail |
| --- | --- |
| `create_prefix` | 64-bit Wine prefix in the game directory |
| `extract` | The zip has no wrapper directory, so its contents go straight into `$GAMEDIR` |
| `wineexec` | Runs the bundled `vc_redist.x64.exe` with `/q /norestart` |
| launch | `$GAMEDIR/FrogSniper.exe` — `FrogSniper` is the game's internal project name |

No pinned Wine version, no Winetricks, no launch flags: DXVK, esync and fsync stay on Lutris'
defaults. The shipped build contains `D3D11RHI`, `D3D12RHI` and `VulkanRHI`, so `-dx11` or
`-vulkan` are available as fallbacks if DX12 misbehaves on your driver.

## Testing a local change

```bash
lutris -i ./leap-of-life.yaml
```

Logs land in `~/.cache/lutris/lutris.log`; the installed configuration is written to
`~/.local/share/lutris/games/leap-of-life-summer-frog-game-jam-v1-<timestamp>.yml`.

To avoid clicking through the file chooser on every test run, do **not** put an absolute path in
`files:`. Lutris 0.5.22 rewrites such a path to `file://…` and then hands it to `requests`, which
has no `file://` adapter and fails with `InvalidSchema`. Serve the zip locally instead:

```bash
python3 -m http.server 8931 --bind 127.0.0.1 --directory ~/Downloads &
sed 's|"N/A:.*"|http://127.0.0.1:8931/leap-of-life-windows.zip|' leap-of-life.yaml > test-install.yaml
lutris -i ./test-install.yaml
```

## Game details

- **Author:** [Oppodelldog](https://oppodelldog.itch.io/) (aware of and fine with this script)
- **Released:** 2026-08-13, build `summer-frog-game-jam-v1`, free
- **Made with:** Unreal Engine, Blender, Krita, FL Studio
- **Controls:** WASD move, Space jump, Shift sprint, right mouse aim, left mouse shoot, E lay eggs

> Leap of Life is a cozy stealth game about protecting the next generation of frogs. Eat insects
> to fill up your life bar, when life bar gets filled more, breeding bar fills up. When the frog
> is ready for breeding, find a breeding site and lay eggs. But with more animals in the pond,
> more predators will occur in the biotope. Be careful and claim all breeding site and frogs
> will survive!

## License

The install script is released under [CC0 1.0](LICENSE) — public domain. It carries no rights to
the game itself; Leap of Life belongs to its author.
