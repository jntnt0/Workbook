# WireGuard VPN — Server and Client Configuration

## Overview

Reference for standing up a WireGuard VPN server on Linux, generating keys, and configuring a client. Covers initial setup, key generation, config file structure, and verification.

---

## Prerequisites

- WireGuard installed on the server (`sudo apt install wireguard`)
- A public-facing server with a known IP or hostname
- `iptables` available for NAT/forwarding rules

---

## Step 1 — Generate Keys

Run on the server. Generate separate key pairs for the server and each client.

```bash
# Server keys
SERVER_PRIVATE_KEY=$(wg genkey)
SERVER_PUBLIC_KEY=$(echo "$SERVER_PRIVATE_KEY" | wg pubkey)

# Client keys
CLIENT_PRIVATE_KEY=$(wg genkey)
CLIENT_PUBLIC_KEY=$(echo "$CLIENT_PRIVATE_KEY" | wg pubkey)

# Confirm values
echo "Server private: $SERVER_PRIVATE_KEY"
echo "Server public:  $SERVER_PUBLIC_KEY"
echo "Client private: $CLIENT_PRIVATE_KEY"
echo "Client public:  $CLIENT_PUBLIC_KEY"
```

Optionally save keys to files for reference:

```bash
echo "$SERVER_PRIVATE_KEY" | sudo tee /etc/wireguard/server_private.key > /dev/null
echo "$SERVER_PUBLIC_KEY"  | sudo tee /etc/wireguard/server_public.key  > /dev/null
sudo chmod 600 /etc/wireguard/server_private.key
```

---

## Step 2 — Server Configuration

Create `/etc/wireguard/wg0.conf`:

```ini
[Interface]
Address     = 192.168.6.1/24
ListenPort  = 51820
PrivateKey  = <server-private-key>
DNS         = 8.8.8.8

# NAT and forwarding — replace <egress-interface> with your outbound interface (e.g. eth0)
PostUp   = iptables -A FORWARD -i wg0 -j ACCEPT; \
           iptables -A FORWARD -o wg0 -j ACCEPT; \
           iptables -t nat -A POSTROUTING -o <egress-interface> -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; \
           iptables -D FORWARD -o wg0 -j ACCEPT; \
           iptables -t nat -D POSTROUTING -o <egress-interface> -j MASQUERADE

[Peer]
PublicKey  = <client-public-key>
AllowedIPs = 192.168.6.2/32
```

Set correct permissions:

```bash
sudo chmod 600 /etc/wireguard/wg0.conf
```

---

## Step 3 — Enable IP Forwarding

Required for the server to route traffic between the VPN tunnel and the internet.

```bash
# Apply immediately
sudo sysctl -w net.ipv4.ip_forward=1

# Persist across reboots
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

---

## Step 4 — Client Configuration

Create `wg0-client.conf` on the client device:

```ini
[Interface]
PrivateKey = <client-private-key>
Address    = 192.168.6.2/32
DNS        = 8.8.8.8

[Peer]
PublicKey           = <server-public-key>
AllowedIPs          = 0.0.0.0/0, ::/0
Endpoint            = <server-ip-or-hostname>:51820
PersistentKeepalive = 25
```

> Set `AllowedIPs = 192.168.6.0/24` instead of `0.0.0.0/0` if you only want split-tunnel access to the VPN subnet rather than routing all traffic through the server.

---

## Step 5 — Start and Enable WireGuard

```bash
# Start the interface
sudo systemctl start wg-quick@wg0

# Enable on boot
sudo systemctl enable wg-quick@wg0

# Verify
sudo systemctl status wg-quick@wg0
```

---

## Step 6 — Verify the Connection

**On the server — confirm peer is connected:**

```bash
sudo wg
```

Expected output shows the peer with a recent handshake timestamp and data transfer counters incrementing.

**From the client — ping the server tunnel address:**

```bash
ping 192.168.6.1
```

**Verify traffic is routing through the VPN:**

```bash
curl https://ifconfig.me
```

The returned IP should match the server's public IP, not the client's local IP.

---

## Resetting Keys and Configs

To regenerate everything from scratch:

```bash
# Backup existing configs
sudo cp /etc/wireguard/wg0.conf /etc/wireguard/wg0.conf.bak

# Remove existing config
sudo rm /etc/wireguard/wg0.conf

# Stop the interface before reconfiguring
sudo systemctl stop wg-quick@wg0
```

Then repeat Steps 1–5 with newly generated keys.

---

## Troubleshooting

|Symptom|Likely Cause|Fix|
|---|---|---|
|No handshake on `sudo wg`|Client config has wrong server public key or endpoint|Verify keys and endpoint match|
|Traffic not routing through VPN|IP forwarding not enabled|Run `sysctl -w net.ipv4.ip_forward=1`|
|VPN drops on server reboot|Service not enabled|Run `systemctl enable wg-quick@wg0`|
|Client can reach VPN subnet but not internet|NAT PostUp rule missing or wrong interface|Confirm `<egress-interface>` in PostUp matches actual outbound interface|
|`curl ifconfig.me` returns local IP|`AllowedIPs` not set to `0.0.0.0/0` on client|Update client config and restart interface|