# Administering Freenet

The node is managed from its own web dashboard. YunoHost's SSO guards the door —
arriving through the portal, the dashboard needs no second login.

## First steps after install

1. Open the dashboard and check the **network status**.
   - Direct inbound peer connections → you're done.
   - Only hole-punched connections → the peer port needs forwarding (see Ports).
2. Optionally review bandwidth limits in the dashboard's configuration.

## Ports

| Purpose | Port | Notes |
|---|---|---|
| Peer transport | `__port_peer__` (UDP) | Forward on your router so peers connect to you directly instead of relying only on hole-punching. |
| Dashboard / node API | loopback only, port `__PORT__` | Never directly exposed; reachable exclusively through the SSO-gated nginx proxy. |

## Updates (automatic by design)

Freenet enforces a minimum compatible version at the peer handshake — a stale node
gets cut off from the network. The systemd unit therefore mirrors upstream's own
contract: when the node exits wanting an update, systemd runs `freenet update` and
restarts it, so **updates apply themselves**.

`yunohost app upgrade freenet` instead installs the exact release pinned by this
package — use it if the self-updater ever misbehaves or you want a known version.

## Data & identity

Node keys and stored data live in the app data dir and are included in YunoHost app
backups. Restoring a backup preserves the same node identity.

## Troubleshooting

| Symptom | Cause & fix |
|---|---|
| Dashboard unreachable through SSO | Confirm `systemctl is-active __APP__` reports `active`; the API port is loopback-only by design, so only the nginx proxy can serve it. |
| Node stopped and won't restart | A crash-looping node stops itself after 5 rapid restarts (upstream's rate limiter). Recover: `systemctl reset-failed __APP__ && systemctl start __APP__`. The self-heal update runs on every stop. |
| Few or no peers | UDP `__port_peer__` not forwarded, or the domain's DNS record is Cloudflare-proxied (must be DNS-only for raw UDP). |
| Port or config oddities | Ports and pinned settings are managed via package resources and ExecStart flags; avoid hand-editing node config — the strict schema can reject unknown keys. |
| Checking why it restarted | `journalctl -u __APP__ -e` — exits with code 42/43 are the updater handshake and count as clean. |
