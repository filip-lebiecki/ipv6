# ipv6

apt remove --purge netplan.io
apt remove --purge network-manager

ip -br l

cd /etc/systemd/network/

vi eth0.network

[Match]
Name=eth0 

[Network]

Address=192.168.10.3/24
Gateway=192.168.10.200
DNS=1.1.1.1
IPForward=yes

systemctl restart systemd-networkd
systemctl status systemd-networkd
ip -br a


apt install systemd-resolved
resolvectl status
rm /etc/resolv.conf
ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
systemctl restart systemd-resolved
resolvectl status
cat /etc/resolv.conf
resolvectl query google.com
resolvectl query google.com


fping -e 118.91.187.67
curl ifconfig.me
cd /etc/systemd/network/
vi wg0.key

vi wg0.netdev

[NetDev]
Name=wg0
Kind=wireguard

[WireGuard]
PrivateKeyFile=/etc/systemd/network/wg0.key
ListenPort=51820

[WireGuardPeer]
PublicKey=<enter public key here>
AllowedIPs=::/1,8000::/1
Endpoint=118.91.187.67:44754
PersistentKeepalive=30

vi wg0.network
[Match]
Name=wg0

[Network]
Address=2a11:6c7:f04:c4::2/64
BindCarrier=eth0
IPForward=yes

[Route]
Destination=::/0
Gateway=2a11:6c7:f04:c4::1

systemctl restart systemd-networkd
ip -br a
ip -6 r
ping 2a11:6c7:f04:c4::1
ping google.com
wg 
curl -6 ifconfig.me
ip -br a
ping 2a11:6c7:f04:c4::2
tracepath 2a11:6c7:f04:c4::2
ssh 2a11:6c7:f04:c4::2

ip -br a
sipcalc 2a11:6c7:1200:c200::/56
vi /etc/systemd/network/eth1.network

[Match]
Name=eth1

[Network]
Address=2a11:06c7:1200:c2ff::1/64
IPForward=yes

systemctl restart systemd-networkd
ip -br a

apt install radvd


vi /etc/radvd.conf
interface eth1 {
    AdvSendAdvert on;
    MaxRtrAdvInterval 600;
    MinRtrAdvInterval 200;
    prefix 2a11:6c7:1200:c2ff::1/64 {
      AdvOnLink on;
      AdvAutonomous on;
    };
    RDNSS 2606:4700:4700::1111 {
        AdvRDNSSLifetime 1200;
    };
};

systemctl restart radvd
systemctl status radvd

systemctl restart systemd-networkd
systemctl restart radvd
systemctl status radvd
sysctl -a | grep forward

ip -br a s dev eth1
ip -6 r
ping fe80::215:5dff:fe0a:2caa%eth1
resolvectl status
rdisc6 eth1
ping google.com
mtr google.com
ip -br a s dev eth1
mtr 2a11:6c7:1200:c2ff:215:5dff:fe0a:2ca8
ping amazon.com
host amazon.com
