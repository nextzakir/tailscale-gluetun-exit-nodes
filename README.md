# Tailscale VPN Exit Nodes via Gluetun

Run multiple country-based Tailscale exit nodes, each tunneled through a VPN provider via Gluetun, with a shared DNS resolver handling DNS for all nodes. The example uses ProtonVPN with NextDNS, but you can replace those with any provider/resolver supported by Gluetun.

## Architecture

![Architecture diagram](architecture.svg)

Each country gets its own Gluetun instance with a dedicated WireGuard configuration. The Tailscale exit node container shares Gluetun's network namespace, so all its traffic goes through the VPN tunnel provided by Gluetun. The diagram shows two example exit nodes (sg / ca) that route through country-specific ProtonVPN WireGuard tunnels and use NextDNS as a shared resolver.

## Prerequisites

- Docker + Docker Compose
- [Tailscale](https://tailscale.com) account
- A VPN provider supported by [Gluetun](https://github.com/qdm12/gluetun-wiki/tree/main/setup/providers) (example uses ProtonVPN with WireGuard)
- A DNS resolver (example uses [NextDNS](https://nextdns.io))

## Setup

### 1. Create the external network

```bash
docker network create --subnet=172.20.0.0/16 vpnlink
```

### 2. Create external volumes

```bash
docker volume create sg-exit-node-data
docker volume create ca-exit-node-data
```

### 3. Fill in the placeholders

| Placeholder | Description |
|---|---|
| `<nextdns_profile_id>` | Your NextDNS profile ID (if using NextDNS) |
| `<wireguard_private_key>` | WireGuard private key from your VPN provider (if using ProtonVPN, find it under Downloads → WireGuard configuration) |
| `<wireguard_addresses>` | WireGuard IP assigned by your VPN provider (if using ProtonVPN, e.g. `10.2.0.2/32`) |
| `<tailscale_auth_key>` | Tailscale auth key |
| `<timezone>` | Your timezone (e.g. `America/New_York`) |

Generate WireGuard credentials from your VPN provider's dashboard. For ProtonVPN, find them under **Downloads → WireGuard configuration**.

### 4. Approve exit nodes

After bringing the stack up, approve each exit node in the [Tailscale admin panel](https://login.tailscale.com/admin/machines) under the machine's settings.

### 5. Bring it up

Start all nodes:

```bash
docker compose up -d
```

Or start specific nodes only:

```bash
docker compose up -d nextdns sg-gluetun sg-exit-node
```

## Adding more countries

Copy any `*-gluetun` + `*-exit-node` service block, update the country name, assign a new static IP (helps Tailscale establishing direct connections) in the `vpnlink` subnet, create a volume for it, and update placeholders.

## Notes

- Each Gluetun instance uses a separate WireGuard tunnel, so they are fully isolated from each other
- NextDNS is used as the shared DNS resolver in this example — replace it with any DNS resolver you prefer
- Tailscale containers do not need `NET_ADMIN` capability since they inherit the network namespace from Gluetun
