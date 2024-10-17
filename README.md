# MAP-E-tunnel
**How to setup MAP-E tunnel**



                     DUT1  eth5================MAP E tunnel==================eth5 DUT2
                      
                     br-lan                                                       br-lan 
                     eth1                                                          eth0
                    (br-lan has eth1)                                         (br-lan has eth0)
                        |                                                           |
                        |                                                           |
                      (PC FTP client)                                           (PC FTP server)

**DUT1 config as below:**

uci set network.wan=interface
uci set network.wan.ifname='eth5'
uci set network.wan.ip6addr=2001::1/64
uci set network.wan.ip6gw=2001::2
uci set network.wan.proto=static
uci set network.lan=interface
uci set network.lan.ifname='eth1 eth4'
uci set network.lan.ipaddr=192.168.1.1
uci set network.lan.netmask=255.255.255.0
uci set network.lan.proto=static
uci commit network

sleep 1
/etc/init.d/network restart
sleep 5
/etc/init.d/firewall stop
sleep 2
/etc/init.d/firewall disable
sleep 2

ip -6 tunnel add map-e mode ip4ip6 remote 2001::2 local 2001::1 dev eth5 encaplimit none
ip link set dev map-e mtu 1460
ip -4 addr add 192.0.2.101/32 dev map-e
ip link set dev map-e up
ip route add default dev map-e
iptables -t nat -I POSTROUTING -o map-e -j MASQUERADE
ip -6 r a 2001::1/128 dev eth5

====================================================================================================

**DUT2 configs as below:**

uci set network.wan=interface
uci set network.wan.ifname='eth5'
uci set network.wan.ip6addr=2001::2/64
uci set network.wan.ip6gw=2001::1
uci set network.wan.proto=static
uci set network.lan=interface
uci set network.lan.ifname='eth0 eth4'
uci set network.lan.ipaddr=192.168.2.1
uci set network.lan.netmask=255.255.255.0
uci set network.lan.proto=static
uci commit network

sleep 1
/etc/init.d/network restart
sleep 5
/etc/init.d/firewall stop
sleep 2
/etc/init.d/firewall disable
sleep 2

ip -6 tunnel add map-e mode ip4ip6 remote 2001::1 local 2001::2 dev eth5 encaplimit none
ip link set dev map-e mtu 1460
ip -4 addr add 192.0.2.102/32 dev map-e
ip link set dev map-e up
ip route add default dev map-e
iptables -t nat -I POSTROUTING -o map-e -j MASQUERADE
ip -6 r a 2001::2/128 dev eth5
