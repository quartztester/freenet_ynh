# Freenet YunoHost package

Run a [Freenet](https://freenet.org) node as a first-class YunoHost app:
managed by systemd, backed up by YunoHost, with the node's dashboard served on
an SSO-gated subdomain.

Uses the **official container image** [`ghcr.io/freenet/freenet-core`](https://freenet.org/quickstart/docker/).
The node keeps itself up to date over the network (peers refuse stale nodes);
the package's `docker pull` on upgrade only refreshes the base image.

## What you get

- A volunteer Freenet peer (opennet) with its identity/chunk store in a named
  Docker volume that survives upgrades and is included in YunoHost backups.
- The node dashboard + client API proxied at `https://<your-domain>/`, gated by
  YunoHost SSO. The API itself always binds loopback only — never exposed.
- Fixed UDP transport port, opened in the YunoHost firewall.

## Prerequisites

- YunoHost ≥ 12 with `docker.io` and `docker-compose-v2` installed
  (`apt install docker.io docker-compose-v2`).
- A (sub)domain you can point at the server (an existing wildcard DNS record is
  fine for HTTPS-served apps).
- The chosen domain must already exist on your YunoHost instance, with a valid
  certificate (YunoHost → Domains).

## Install

```bash
yunohost app install https://github.com/quartztester/freenet_ynh -a "domain=freenet.example.com&path=/"
```

Or in the web admin: **Settings → Applications → Install a custom app**,
paste the URL above.

Install questions:

| Question | Default | Meaning |
|---|---|---|
| Peer port (UDP) | `31337` | The UDP port other Freenet peers reach you on. Also opened in the YunoHost firewall. |

### Router port-forward (do this!)

Forward **UDP 31337** (or your chosen port) on your router to this server's LAN
address. Freenet's transport is UDP-only — there is no TCP component. Without
the forward the node still runs and relays, but direct inbound peer connections
are limited.

## Dashboard

`https://<your-domain>/` — behind YunoHost SSO. It serves the node dashboard
and any Freenet apps the node exposes.

The client API binds loopback (`http://127.0.0.1:7509`) and is deliberately not
reachable from the network except through the SSO-gated proxy. Treat it like a
database socket: it can read and modify node state and key material.

## Admin notes

- Node logs: `docker logs <app>` and `/data/logs` inside the volume.
- Version: `docker exec <app> freenet --version`
- Health: `docker inspect --format '{{.State.Health.Status}}' <app>`
- Disk: the shared chunk store grows inside the `freenet-data` Docker volume;
  watch it with `docker system df`. The package does not impose a store cap
  (none is configurable via documented env vars); size the disk you give the
  node by controlling free space.
- Upgrades: `yunohost app upgrade freenet` (recreates the container; identity
  volume preserved).

## Risks / responsibilities (read before running a public peer)

Freenet runs on the open internet by default (opennet): **your node's public IP
is announced to other peers**, and the network stores and relays content you do
not control or choose. Check your ISP's terms of service and local laws before
running a public node from a residential connection, and consider a bandwidth
and disk budget accordingly.

## License

This packaging is GPLv3+ (like YunoHost packages generally). Freenet itself is
GPLv2+; see [freenet/freenet-core](https://github.com/freenet/freenet-core).
