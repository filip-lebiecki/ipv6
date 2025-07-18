# IPv6 only network [Youtube video](https://youtu.be/F2t9qBfSY08?si=y-ti4BcSWdU4mIF3)
# IPv6 only network part 2 [Youtube video](https://youtu.be/XrBuoTAhEbE?si=R9AGQNxfoaNQ3vOB)

# Router external interface configuration

```bash
apt remove --purge netplan.io
apt remove --purge network-manager
ip -br l
cd /etc/systemd/network/
vi eth0.network
# [Match]
# Name=eth0
# [Network]
# Address=192.168.10.3/24
# Gateway=192.168.10.200
# DNS=1.1.1.1
systemctl restart systemd-networkd
systemctl status systemd-networkd
ip -br a
```

# DNS stub resolver
```bash
apt install systemd-resolved
resolvectl status
rm /etc/resolv.conf
ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
systemctl restart systemd-resolved
resolvectl status
cat /etc/resolv.conf
resolvectl query google.com
```

# IPv6 Tunnel Setup
```bash
fping -e 118.91.187.67
curl ifconfig.me
cd /etc/systemd/network/
vi wg0.key # Paste WireGuard Private Key
vi wg0.netdev
# [NetDev]
# Name=wg0
# Kind=wireguard
# [WireGuard]
# PrivateKeyFile=/etc/systemd/network/wg0.key
# ListenPort=51820
# [WireGuardPeer]
# PublicKey=vinh5d0qq7DkXAkldLl/aeDtM8P5L0QAmuZ9SmWnVSY= (Route64 Peer Public Key)
# AllowedIPs=::/1,8000::/1
# Endpoint=118.91.187.67:44754 (Route64 PoP Endpoint)
# PersistentKeepalive=30
vi wg0.network
# [Match]
# Name=wg0
# [Network]
# Address=2a11:6c7:f04:c4::2/64 (Your Tunnel IPv6 Address)
# BindCarrier=eth0
# [Route]
# Destination=::/0
# Gateway=2a11:6c7:f04:c4::1 (Route64 Tunnel Gateway)
systemctl restart systemd-networkd
```

# IPv6 connectivity test
```bash
ip -br a
ip -6 r
ping 2a11:6c7:f04:c4::1
ping google.com
wg
curl -6 ifconfig.me (on external client)
ip -br a (on external client)
ping 2a11:6c7:f04:c4::2 (from external client to router)
tracepath 2a11:6c7:f04:c4::2 (from external client to router)
ssh 2a11:6c7:f04:c4::2 (from external client to router)
```

# Router internal interface configuration
```bash
ip -br a
sipcalc 2a11:6c7:1200:c200::/56 (Your /56 prefix)
vi /etc/systemd/network/eth1.network
# [Match]
# Name=eth1
# [Network]
# Address=2a11:06c7:1200:c2ff::1/64 (Your chosen /64 subnet for internal interface)
systemctl restart systemd-networkd
ip -br a
apt install radvd
vi /etc/radvd.conf
# interface eth1 {
#     AdvSendAdvert on;
#     MaxRtrAdvInterval 600;
#     MinRtrAdvInterval 200;
#     prefix 2a11:6c7:1200:c2ff::1/64 {
#         AdvOnLink on;
#         AdvAutonomous on;
#     };
#     RDNSS 2606:4700:4700::1111 {
#         AdvRDNSSLifetime 1200;
#     };
# };
systemctl restart radvd
systemctl status radvd
```

# Enable IP forwarding
```bash
cd /etc/systemd/network/
vi eth0.network # Add IPForward=yes
vi eth1.network # Add IPForward=yes
vi wg0.network # Add IPForward=yes
systemctl restart systemd-networkd
systemctl restart radvd
systemctl status radvd
sysctl -a | grep forward
```

# Client connectivity test
```bash
ip -br a s dev eth1 (client)
ip -6 r (client)
ping fe80::215:5dff:fe0a:2caa%eth1 (client to router link-local)
router: ip -br a s dev eth1
resolvectl status (client)
rdisc6 eth1 (client)
ping google.com (client)
mtr google.com (client)
ip -br a s dev eth1 (client)
mtr 2a11:6c7:1200:c2ff:215:5dff:fe0a:2ca8 (from external IPv6 host to client)
ping amazon.com (client)
host amazon.com (client)
```

# Public DNS64/NAT64 Gateway

On client

```bash
host ovh.com
http ovh.com
vi /etc/systemd/network/eth1.network
UseDNS=no
DNS=2a01:4f9:c010:3f02::1

systemctl restart systemd-networkd
resolvectl status
host ovh.com
http ovh.com
```

# Self-Hosted NAT64 (Tayga) with External DNS64 (Google Public DNS64)

On server

```bash
apt install tayga
vi /etc/tayga.conf` 
ipv6-addr 2001:db8:1::2
prefix 64:ff9b::/96

systemctl restart tayga
vi /etc/radvd.conf
RDNSS 2001:4860:4860::6464

systemctl restart radvd
vi /etc/systemd/network/eth1.network
removing custom DNS

systemctl restart systemd-networkd
host ovh.com
ping ovh.com
```

# Fully Self-Hosted DNS64 (Unbound) & NAT64 (Tayga)

On server

```bash
apt install unbound
cp server.conf /etc/unbound/unbound.conf.d/
cat server.conf
systemctl restart unbound
ip -6 -br a
vi /etc/radvd.conf
RDNSS 2a11:6c7:1100:caff::1

systemctl restart radvd
resolvectl status
host ovh.com
ping ovh.com
```
