# Supply-Chain Hardening — Developer Workstation

3-day install-age gate + disabled lifecycle scripts across npm, pnpm, Bun, uv, pip. Apply once per workstation.

## File paths per OS

| Tool | Linux | macOS | Windows |
|---|---|---|---|
| npm | `~/.npmrc` | `~/.npmrc` | `%USERPROFILE%\.npmrc` |
| pnpm | `~/.config/pnpm/config.yaml` | `~/Library/Preferences/pnpm/config.yaml` | `%LOCALAPPDATA%\pnpm\config\config.yaml` |
| Bun | `~/.bunfig.toml` | `~/.bunfig.toml` | `%USERPROFILE%\.bunfig.toml` |
| uv | `~/.config/uv/uv.toml` | `~/.config/uv/uv.toml` | `%APPDATA%\uv\uv.toml` |

pip uses an environment variable — see below.

## Contents

### npm — `.npmrc`
```ini
min-release-age=3
ignore-scripts=true
audit-level=high
fund=false
```

### pnpm — `config.yaml`
**pnpm 11 ignores `.npmrc` for non-auth settings.** Must use YAML.
```yaml
minimumReleaseAge: 4320       # 3 days, in minutes
ignoreScripts: true
blockExoticSubdeps: true
```

### Bun — `.bunfig.toml`
```toml
[install]
exact = true
auto = "fallback"
ignoreScripts = true
minimumReleaseAge = 259200    # 3 days, in seconds

[install.lockfile]
save = true
```

### uv — `uv.toml`
```toml
exclude-newer = "3 days"
```

### pip — environment variable

| OS | How |
|---|---|
| Linux / macOS | Add `export PIP_UPLOADED_PRIOR_TO=P3D` to `~/.bashrc` and `~/.zshrc` |
| Windows | `setx PIP_UPLOADED_PRIOR_TO P3D` (PowerShell, persists across sessions) |

Requires pip ≥ 26.1.

## When `ignoreScripts=true` blocks a real package

Native modules (`sharp`, `bcrypt`, `esbuild`, `prisma`, `puppeteer`, etc.) need their install scripts. Allowlist per project:

- **pnpm 11**: `allowBuilds: { esbuild: true, sharp: true }` in project `pnpm-workspace.yaml`
- **npm**: `npm rebuild <pkg> --foreground-scripts` after install
- **Bun**: add `"trustedDependencies": ["sharp"]` to `package.json`

## Notes on units (different per tool)

| Tool | Key | Unit |
|---|---|---|
| npm | `min-release-age` | days |
| pnpm | `minimumReleaseAge` | minutes |
| Bun | `minimumReleaseAge` | seconds |
| uv | `exclude-newer` | duration string (`"3 days"`) |
| pip | `PIP_UPLOADED_PRIOR_TO` | ISO 8601 (`P3D`) |
