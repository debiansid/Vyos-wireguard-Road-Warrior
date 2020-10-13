# VyOS - Wireguard (Road Warrior)

## Assumptions

1. Generated a public/private keypair for Wireguard.
2. [Zone policy](https://vyos.readthedocs.io/en/latest/appendix/examples/zone-policy.html) zones have been implemented (_zones_: `WAN`, `LAN`, & `LOCAL`).
3. Wireguard has the subnet `192.168.50.0/24`, and all peers have an unique IP within that subnet.
4. Wireguard is listening on port `51820`.
5. Server's public IP is `15.15.15.15` and internal DNS servers are `192.168.1.253` & `192.168.1.254`.
6. All rules in this example are named 10, for no real reason.

## VyOS configuration

### Firewall
```
set firewall name WAN-LAN rule 10 description "allow wireguard traffic"
set firewall name WAN-LAN rule 10 action accept
set firewall name WAN-LAN rule 10 log enable
set firewall name WAN-LAN rule 10 destination port 51820
set firewall name WAN-LAN rule 10 protocol udp
set firewall name WAN-LAN rule 10 state new enable

```

### Interfaces

Each peer has it's own unique IP, as shown by the `/32` netmask.

```
# wg0 interface
set interfaces wireguard wg0 address 192.168.50.1/24
set interfaces wireguard wg0 port 51820

# wg0 peer #1
set interfaces wireguard wg0 peer user1-phone allowed-ips 192.168.50.2/32
set interfaces wireguard wg0 peer user1-phone pubkey ABC...XYZ
# wg0 peer #2
set interfaces wireguard wg0 peer user1-laptop allowed-ips 192.168.50.3/32
set interfaces wireguard wg0 peer user1-laptop pubkey DEF...TUV
# wg0 peer #3
set interfaces wireguard wg0 peer user2-phone allowed-ips 192.168.50.4/32
set interfaces wireguard wg0 peer user2-phone pubkey GHI...QRS

# optional
set interfaces wireguard wg0 peer $PEER-NAME persistent-keepalive 15
```

### NAT

```
set nat source rule 10 outbound-interface eth0
set nat source rule 10 source address 192.168.50.0/24
set nat source rule 10 translation address masquerade
```

### Protocols

```
set protocols static interface-route 192.168.50.0/24 next-hop-interface wg0
```

### Zone Policy

```
set zone-policy zone LAN interface wg0
```

## Peer Configuration

Let's assume that this is the setup for peer #1 - user1-phone. Other peers will have similar settings.

```
[Interface]
PrivateKey = 000...000 # peer's private key
Address = 192.168.50.2/24
DNS = 192.168.1.253,192.168.1.254 # use internal DNS

[Peer]
PublicKey = 000...000 # server's public key
AllowedIPs = 0.0.0.0/0, ::/0 # force all traffic through the tunnel
Endpoint = 15.15.15.15:51820 # server's public IP
PersistentKeepalive = 25 # optional