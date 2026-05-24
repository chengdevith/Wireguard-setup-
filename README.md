# WireGuard Setup for Cross-Account K8s Cluster

## Overview

This guide documents how to set up a WireGuard mesh network between 5 DigitalOcean
droplets across 2 accounts to enable Kubernetes cluster scaling.

```
Account A (3 nodes)              Account B (2 nodes)
┌────────────────────┐          ┌────────────────────┐
│ node1 (CP)         │          │ node4 (worker)     │
│ node2 (CP)         │◄────────►│ node5 (worker)     │
│ node3 (CP)         │          └────────────────────┘
└────────────────────┘
     WireGuard Mesh (10.200.0.0/24)
```

## Node IP Reference

| Node  | Account | Public IP        | Internal IP  | WireGuard IP |
|-------|---------|------------------|--------------|--------------|
| node1 | A       | 170.64.214.50    | 10.126.0.4   | 10.200.0.1   |
| node2 | A       | 209.38.29.170    | 10.126.0.3   | 10.200.0.2   |
| node3 | A       | 134.199.154.163  | 10.126.0.2   | 10.200.0.3   |
| node4 | B       | 170.64.133.124   | -            | 10.200.0.4   |
| node5 | B       | 170.64.200.248   | -            | 10.200.0.5   |

> **Important:** node1, node2, node3 are in the same DO VPC so they use
> **internal IPs** as WireGuard endpoints. node4 and node5 use **public IPs**.

---

## Step 1 — Install WireGuard (All 5 Nodes)

```bash
sudo su -
apt update && apt install -y wireguard wireguard-tools
mkdir -p /etc/wireguard
```

## Step 2 — Generate Keys (All 5 Nodes)

Run on each node and save the public key output:

```bash
wg genkey | tee /etc/wireguard/privatekey | wg pubkey > /etc/wireguard/publickey
chmod 600 /etc/wireguard/privatekey
cat /etc/wireguard/publickey   # save this
cat /etc/wireguard/privatekey  # use this in wg0.conf
```

## Step 3 — Create WireGuard Config

Create `/etc/wireguard/wg0.conf` on each node. Replace `<THIS_NODE_PRIVATE_KEY>`
with the output of `cat /etc/wireguard/privatekey` on that node.

### node1 — `10.200.0.1`

```ini
[Interface]
Address = 10.200.0.1/24
ListenPort = 51820
PrivateKey = <NODE1_PRIVATE_KEY>
MTU = 1420

[Peer]
# node2 — same VPC, use internal IP
PublicKey = ix+j8k6c8hXtaQJjtm+iuhXHJXCRxLqA3GdoIIKlTWw=
Endpoint = 10.126.0.3:51820
AllowedIPs = 10.200.0.2/32
PersistentKeepalive = 10

[Peer]
# node3 — same VPC, use internal IP
PublicKey = sZoNQd6dVf5571sU8EZYQ+wzrMQgha6blv1HIR4NzBU=
Endpoint = 10.126.0.2:51820
AllowedIPs = 10.200.0.3/32
PersistentKeepalive = 10

[Peer]
# node4 — different account, use public IP
PublicKey = hwdTUSp5wmwqRiCYNMLdGeCuOjkB6g/hv63SXllTBEE=
Endpoint = 170.64.133.124:51820
AllowedIPs = 10.200.0.4/32
PersistentKeepalive = 10

[Peer]
# node5 — different account, use public IP
PublicKey = tXe8HdhlSSFnNhPrsZUvbpdCdaLkHB+2CyNVAD7BnSA=
Endpoint = 170.64.200.248:51820
AllowedIPs = 10.200.0.5/32
PersistentKeepalive = 10
```

### node2 — `10.200.0.2`

```ini
[Interface]
Address = 10.200.0.2/24
ListenPort = 51820
PrivateKey = <NODE2_PRIVATE_KEY>
MTU = 1420

[Peer]
# node1 — same VPC, use internal IP
PublicKey = 0PVpB2UfigYF8hEJ0Vlr1eyS/QnO3+Gkp9/ODFLuhWk=
Endpoint = 10.126.0.4:51820
AllowedIPs = 10.200.0.1/32
PersistentKeepalive = 10

[Peer]
# node3 — same VPC, use internal IP
PublicKey = sZoNQd6dVf5571sU8EZYQ+wzrMQgha6blv1HIR4NzBU=
Endpoint = 10.126.0.2:51820
AllowedIPs = 10.200.0.3/32
PersistentKeepalive = 10

[Peer]
# node4 — different account, use public IP
PublicKey = hwdTUSp5wmwqRiCYNMLdGeCuOjkB6g/hv63SXllTBEE=
Endpoint = 170.64.133.124:51820
AllowedIPs = 10.200.0.4/32
PersistentKeepalive = 10

[Peer]
# node5 — different account, use public IP
PublicKey = tXe8HdhlSSFnNhPrsZUvbpdCdaLkHB+2CyNVAD7BnSA=
Endpoint = 170.64.200.248:51820
AllowedIPs = 10.200.0.5/32
PersistentKeepalive = 10
```

### node3 — `10.200.0.3`

```ini
[Interface]
Address = 10.200.0.3/24
ListenPort = 51820
PrivateKey = <NODE3_PRIVATE_KEY>
MTU = 1420

[Peer]
# node1 — same VPC, use internal IP
PublicKey = 0PVpB2UfigYF8hEJ0Vlr1eyS/QnO3+Gkp9/ODFLuhWk=
Endpoint = 10.126.0.4:51820
AllowedIPs = 10.200.0.1/32
PersistentKeepalive = 10

[Peer]
# node2 — same VPC, use internal IP
PublicKey = ix+j8k6c8hXtaQJjtm+iuhXHJXCRxLqA3GdoIIKlTWw=
Endpoint = 10.126.0.3:51820
AllowedIPs = 10.200.0.2/32
PersistentKeepalive = 10

[Peer]
# node4 — different account, use public IP
PublicKey = hwdTUSp5wmwqRiCYNMLdGeCuOjkB6g/hv63SXllTBEE=
Endpoint = 170.64.133.124:51820
AllowedIPs = 10.200.0.4/32
PersistentKeepalive = 10

[Peer]
# node5 — different account, use public IP
PublicKey = tXe8HdhlSSFnNhPrsZUvbpdCdaLkHB+2CyNVAD7BnSA=
Endpoint = 170.64.200.248:51820
AllowedIPs = 10.200.0.5/32
PersistentKeepalive = 10
```

### node4 — `10.200.0.4`

```ini
[Interface]
Address = 10.200.0.4/24
ListenPort = 51820
PrivateKey = <NODE4_PRIVATE_KEY>
MTU = 1420

[Peer]
# node1 — use public IP (different account)
PublicKey = 0PVpB2UfigYF8hEJ0Vlr1eyS/QnO3+Gkp9/ODFLuhWk=
Endpoint = 170.64.214.50:51820
AllowedIPs = 10.200.0.1/32
PersistentKeepalive = 10

[Peer]
# node2 — use public IP (different account)
PublicKey = ix+j8k6c8hXtaQJjtm+iuhXHJXCRxLqA3GdoIIKlTWw=
Endpoint = 209.38.29.170:51820
AllowedIPs = 10.200.0.2/32
PersistentKeepalive = 10

[Peer]
# node3 — use public IP (different account)
PublicKey = sZoNQd6dVf5571sU8EZYQ+wzrMQgha6blv1HIR4NzBU=
Endpoint = 134.199.154.163:51820
AllowedIPs = 10.200.0.3/32
PersistentKeepalive = 10

[Peer]
# node5 — same account B
PublicKey = tXe8HdhlSSFnNhPrsZUvbpdCdaLkHB+2CyNVAD7BnSA=
Endpoint = 170.64.200.248:51820
AllowedIPs = 10.200.0.5/32
PersistentKeepalive = 10
```

### node5 — `10.200.0.5`

```ini
[Interface]
Address = 10.200.0.5/24
ListenPort = 51820
PrivateKey = <NODE5_PRIVATE_KEY>
MTU = 1420

[Peer]
# node1 — use public IP (different account)
PublicKey = 0PVpB2UfigYF8hEJ0Vlr1eyS/QnO3+Gkp9/ODFLuhWk=
Endpoint = 170.64.214.50:51820
AllowedIPs = 10.200.0.1/32
PersistentKeepalive = 10

[Peer]
# node2 — use public IP (different account)
PublicKey = ix+j8k6c8hXtaQJjtm+iuhXHJXCRxLqA3GdoIIKlTWw=
Endpoint = 209.38.29.170:51820
AllowedIPs = 10.200.0.2/32
PersistentKeepalive = 10

[Peer]
# node3 — use public IP (different account)
PublicKey = sZoNQd6dVf5571sU8EZYQ+wzrMQgha6blv1HIR4NzBU=
Endpoint = 134.199.154.163:51820
AllowedIPs = 10.200.0.3/32
PersistentKeepalive = 10

[Peer]
# node4 — same account B
PublicKey = hwdTUSp5wmwqRiCYNMLdGeCuOjkB6g/hv63SXllTBEE=
Endpoint = 170.64.133.124:51820
AllowedIPs = 10.200.0.4/32
PersistentKeepalive = 10
```

---

## Step 4 — Start WireGuard (All 5 Nodes)

```bash
wg-quick up wg0
systemctl enable wg-quick@wg0

# Verify
wg show
```

---

## Step 5 — Test Connectivity (From node1)

```bash
ping -c 3 10.200.0.2   # node2
ping -c 3 10.200.0.3   # node3
ping -c 3 10.200.0.4   # node4
ping -c 3 10.200.0.5   # node5
```

Expected latency: `< 5ms` for all nodes (same DO region).

---

## Step 6 — Open Firewall Ports (Both DO Accounts)

In DigitalOcean Console → Networking → Firewalls, add inbound rule:

```
Type:     Custom
Protocol: UDP
Port:     51820
Sources:  All IPv4, All IPv6
```

Also open these ports for Kubernetes:

```
TCP 6443        # K8s API server
TCP 10250       # kubelet
TCP 9500-9503   # Longhorn
UDP 51820       # WireGuard
```

---

## Key Lessons Learned

**Same-account nodes (Account A):**
- DO blocks public IP traffic between droplets in the same VPC
- Must use **internal IPs** (`10.126.0.x`) as WireGuard endpoints

**Cross-account nodes (Account A ↔ Account B):**
- Must use **public IPs** as WireGuard endpoints
- DO routes traffic efficiently between regions — expect ~1-3ms latency

**etcd peer communication:**
- etcd TLS certs are issued for specific IPs
- Do NOT change etcd peer URLs to internal IPs unless you regenerate certs
- etcd between same-VPC nodes works fine over public IPs via DO backbone

---

## Uninstall WireGuard (Safe — No Data Loss)

```bash
# 1. Bring down interface
wg-quick down wg0

# 2. Disable autostart
systemctl disable wg-quick@wg0

# 3. Remove package
apt remove -y wireguard wireguard-tools

# 4. Clean up configs
rm -f /etc/wireguard/wg0.conf
rm -f /etc/wireguard/privatekey
rm -f /etc/wireguard/publickey
```

> Longhorn data, K8s state, and etcd data are completely unaffected by
> removing WireGuard.