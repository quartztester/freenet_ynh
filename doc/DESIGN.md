# freenet — packaging design

Volunteer peer node for the Freenet p2p network: a single self-updating static
binary whose on-disk version can legitimately drift ahead of the packaged pin.

Anything not mentioned below is standard YunoHost boilerplate identical in most packages.

## Lifecycle
- **Install:** path/multi-instance guards → `ynh_setup_source` (one bare binary) →
  systemd unit + `yunohost service add` → start, poll `/v1/version` up to 300 s → nginx + portal tile.
- **Upgrade:** re-extract sources only when the on-disk binary is older than the pin; re-render unit+nginx; node state untouched.
- **Backup/restore:** archive data_dir, binary, unit; restore re-adds the service, re-renders nginx, `systemctl enable` before restart.

## Decisions
- **Source:** official GitHub release assets, sha256 pins, per-arch, `in_subdir = false` — the tarball holds a single bare `freenet` file ([manifest.toml L64-L74](../manifest.toml#L64-L74)); `autoupdate.strategy = latest_github_release` with per-arch asset regexes ([L72-L74](../manifest.toml#L72-L74)).
- **Runtime:** static musl binary; no interpreter or build chain ([manifest.toml L20-L21](../manifest.toml#L20-L21)).
- **Services:** one unit, plus two upstream-contract hooks: `ExecStartPre` kills orphaned nodes holding the port ([conf/systemd.service L23](../conf/systemd.service#L23)); `ExecStopPost` runs `freenet update` on crash-like exits ([L36](../conf/systemd.service#L36)).
- **Storage:** no DB; node identity + store in `$data_dir`; unit forces `HOME` into data_dir because updater/rollback state has no flag to move ([L18](../conf/systemd.service#L18)).
- **Auth:** `ldap/sso = not_relevant` — the node has no accounts; SSO gates the dashboard URL with `main.auth_header = false` so ssowat never injects Basic creds into the same-origin API ([manifest.toml L24-L25](../manifest.toml#L24-L25), [L51-L54](../manifest.toml#L51-L54)).
- **Email:** no mail.
- **Path:** root-only, enforced by `ynh_die` in install ([scripts/install L16-L22](../scripts/install#L16-L22)) and change_url ([scripts/change_url L10-L16](../scripts/change_url#L10-L16)); `[install.path]` question defaults to `/`.
- **Ports/firewall:** loopback dashboard 7509; UDP 31337 fixed+exposed ([manifest.toml L56-L62](../manifest.toml#L56-L62)); `--needs_exposed_ports` registered in install/upgrade/restore so the ports diagnosis keeps reminding ([scripts/install L46](../scripts/install#L46)).
- **Scheduler:** none — self-update rides the unit's ExecStopPost.
- **Migration:** upgrade compares on-disk vs pinned version and skips downgrade of a self-updated node ([scripts/upgrade L18-L26](../scripts/upgrade#L18-L26)); no secrets exist to preserve.
- **Sandboxing:** example_ynh baseline; deviation `ReadWritePaths=$install_dir $data_dir` because the self-updater replaces the binary in place ([conf/systemd.service L38-L50](../conf/systemd.service#L38-L50)).
- **Backup:** `$data_dir` + binary + unit archived; restore regenerates nginx (not in archive) and tolerates pre-binary-backup installs ([scripts/restore L27-L30](../scripts/restore#L27-L30)).
- **Admin-surface:** no config panel; all knobs live in the app's own dashboard (doc/ADMIN.md).

## Don't break
- `exclude = ["install.subdir"]` is not sloppiness — the dashboard emits absolute URLs ([tests.toml L6-L8](../tests.toml#L6-L8)).
- Restore must `systemctl enable` before restart: the CI post-step `systemctl isolate` kills restored-but-disabled units ([scripts/restore L44-L48](../scripts/restore#L44-L48)).
- `yunohost user permission update "$app" --logo` takes the bare app name; `$app.main` silently no-ops ([scripts/install L80](../scripts/install#L80)).
- The tarball ships the binary without the exec bit — `chmod 0755` after every `ynh_setup_source` ([scripts/install L30-L31](../scripts/install#L30-L31)).
- `SuccessExitStatus=42 43` + `RestartPreventExitStatus=43`: 42 means "update me", 43 means "port held" — both are upstream protocol, not errors to sanitize ([conf/systemd.service L32-L34](../conf/systemd.service#L32-L34)).
- change_url re-renders the systemd unit, not just nginx: `--allowed-host` lives in ExecStart flags ([scripts/change_url L41-L47](../scripts/change_url#L41-L47)).
- Multi-instance guard in install is deliberate: YNH 12.1 does not enforce `multi_instance = false` on its own ([scripts/install L10-L14](../scripts/install#L10-L14)).
- README is readme_generator output edited by hand (badges, appstore link) — version badge must be re-synced on each bump ([README.md L14](../README.md#L14)).
