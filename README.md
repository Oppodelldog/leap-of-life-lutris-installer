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
installer moderation involved. Tested with Lutris 0.5.22.

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
`~/.local/share/lutris/games/leap-of-life-<timestamp>.yml`.

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
