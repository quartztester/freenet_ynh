Freenet is a peer-to-peer platform for censorship-resistant publishing and
communication, developed by a nonprofit organization. Freenet nodes store and
route encrypted fragments of a shared data store on behalf of the network: the
operator neither selects nor can read the content a node holds.

This package runs an official Freenet node (the upstream `freenet-core`
container image) as a volunteer peer. The node's dashboard and client API are
served on a dedicated domain behind YunoHost's SSO; the API itself always
binds loopback and is never directly exposed.

The node participates in the opennet: its public IP is announced to other
peers, and it stores/relays encrypted data chosen by the network, not by you.
Review your ISP's terms of service before running a public peer — see the
README for details.
