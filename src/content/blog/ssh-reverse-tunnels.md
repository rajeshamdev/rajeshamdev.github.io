---
title: 'SSH Reverse Tunnels: Reaching Machines You Cannot Reach'
description: 'How reverse SSH tunnels work, when to use them, and the gotchas nobody mentions until it breaks.'
pubDate: 'Apr 12 2026'
---

A regular SSH tunnel forwards a port on your local machine to a port on a remote host. A **reverse** tunnel does the opposite: a remote host forwards one of *its* ports back to you, so you can reach services on machines that sit behind NAT, a firewall, or any other network you can't initiate connections into.

The canonical use case is a headless device in the field — a test rig, an embedded radio, an IoT sensor — that can dial out to the internet but can't accept inbound connections. The device opens an SSH connection to a jump host you control, and asks the jump host to listen on a port and forward anything it receives back down the tunnel.

## The minimum viable command

On the device:

```bash
ssh -N -R 2222:localhost:22 user@jump.example.com
```

That tells the jump host: "open port 2222, and whatever shows up there, pipe it through this SSH connection to my port 22." Now from anywhere you can reach the jump host:

```bash
ssh -p 2222 user@jump.example.com
```

…and you land on the device.

## What they don't tell you

- **`GatewayPorts` is off by default.** Without `GatewayPorts clientspecified` in `sshd_config`, the forwarded port on the jump host binds only to `127.0.0.1`. You can `ssh -p 2222` *on* the jump host but not from outside it.
- **The tunnel dies with the network.** A reverse tunnel is just a TCP connection. Any network blip drops it and it doesn't come back. Production setups pair it with `autossh` or a systemd unit with `Restart=always`.
- **`ServerAliveInterval` is required, not optional.** Without keepalives, a silent connection drop can leave the jump host thinking the tunnel is still alive and refusing to re-bind the port when the device reconnects.

## Where I'd use it

...
