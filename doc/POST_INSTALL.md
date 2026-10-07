# After installation

The Freenet node is installed and running. This page summarizes what to check first.

## Access the dashboard

**https://`__domain__`/** — gated behind YunoHost's SSO, so only users of your
portal can reach it. No second login is required when arriving through the portal.

## Required next step: open the peer port

Forward port **`__port_peer__` (UDP)** on your internet box/router to this server.
With it, peers connect to your node directly; without it, connectivity relies
entirely on hole-punching and the node contributes less to the network.

Check the dashboard's **network status** after forwarding: you should see inbound
peer connections, not only outbound ones.

If your DNS for this host goes through a proxy/CDN (e.g. Cloudflare orange-cloud),
the record must be DNS-only — proxied records drop raw UDP traffic.

## Updates are automatic

Freenet enforces a minimum compatible version between peers, so the node must stay
current. The package handles this: when the node exits wanting an update, its
supervisor applies the official release and restarts it — **no action needed from
you**. See the admin doc for the details and how to pin a version via
`yunohost app upgrade freenet`.

## Do not

- Hand-edit the node's configuration files — the schema is strict and unknown keys
  can prevent startup. Ports and pinned settings are managed by the package.
- The dashboard API is loopback-only by design; it is reachable exclusively through
  the SSO-gated proxy. Keep it that way.

Further administration and troubleshooting: see the app's
**Documentation** tab (admin doc) in the YunoHost admin.
