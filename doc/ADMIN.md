The node is administered from its own dashboard (YunoHost's SSO guards the door). Notes:

- **Updates**: Freenet enforces a minimum compatible version at the peer handshake, so a stale node gets cut off. The systemd unit mirrors upstream's own contract: when the node exits wanting an update, systemd runs `freenet update` and restarts it — updates apply themselves. `yunohost app upgrade freenet` installs the release pinned by the package instead (useful if the self-updater ever rolls back).
- **Ports**: transport is UDP `__port_peer__` (forward it on your router), dashboard API is loopback-only on port `__PORT__`, reachable exclusively through the SSO-gated nginx proxy.
- **Identity**: node keys and stored data live in the app data dir and are included in YunoHost app backups; restoring keeps the same node identity.
- A crash-looping node stops itself after 5 rapid restarts (upstream's rate limiter); `systemctl reset-failed __APP__ && systemctl start __APP__` recovers, and the self-heal updater runs on each stop.
