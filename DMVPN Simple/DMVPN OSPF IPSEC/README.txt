OSPF DMVPN IPSEC


Loopback10 10.1.x.x/24

OSPF
R2#sh run | sec router ospf
router ospf 111
 router-id 10.1.2.2
 network 10.1.2.0 0.0.0.255 area 0
 network 10.100.1.0 0.0.0.3 area 0
 network 172.16.1.0 0.0.0.255 area 0

R3#sh run | sec router
router ospf 111
 router-id 10.1.3.3
 area 1 nssa no-summary
 area 1 range 10.193.56.0 255.255.254.0
 network 10.1.3.0 0.0.0.255 area 1
 network 10.193.56.0 0.0.0.127 area 1
 network 10.193.56.128 0.0.0.127 area 1
 network 10.193.57.0 0.0.0.127 area 1
 network 10.193.57.128 0.0.0.127 area 1
 network 172.16.1.0 0.0.0.255 area 0


R4#sh run | sec router
router ospf 111
 router-id 10.1.4.4
 area 1 nssa no-summary
 area 1 range 10.193.54.0 255.255.254.0
 network 10.1.4.0 0.0.0.255 area 1
 network 172.16.1.0 0.0.0.255 area 0
 network 172.16.47.0 0.0.0.255 area 1

R7#sh run | sec router
router ospf 1
 router-id 10.1.7.7
 area 1 nssa
 network 10.1.7.0 0.0.0.255 area 1
 network 10.193.54.0 0.0.0.127 area 1
 network 10.193.54.128 0.0.0.127 area 1
 network 10.193.55.0 0.0.0.127 area 1
 network 10.193.55.128 0.0.0.127 area 1
 network 172.16.47.0 0.0.0.255 area 1

IPSEC
crypto isakmp policy 10
 hash md5
 authentication pre-share
crypto isakmp key cisco123 address 0.0.0.0
!
!
crypto ipsec transform-set strong esp-3des esp-md5-hmac
 mode tunnel
!
crypto ipsec profile cisco
 set security-association lifetime seconds 120
 set transform-set strong

DMVPN TUNNEL

R2#?interface Tunnel0
 ip address 172.16.1.2 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp authentication gmlabs
 ip nhrp map multicast dynamic
 ip nhrp network-id 111
 ip nhrp shortcut
 ip nhrp redirect
 ip ospf network point-to-multipoint
 tunnel source GigabitEthernet0/0
 tunnel mode gre multipoint
 tunnel key 123
 tunnel protection ipsec profile cisco
end


R3?interface Tunnel0
 ip address 172.16.1.3 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp authentication gmlabs
 ip nhrp map 172.16.1.2 26.2.6.2
 ip nhrp map multicast 26.2.6.2
 ip nhrp network-id 111
 ip nhrp holdtime 60
 ip nhrp nhs 172.16.1.2
 ip nhrp registration timeout 30
 ip nhrp shortcut
 ip ospf network point-to-multipoint
 tunnel source GigabitEthernet0/0
 tunnel mode gre multipoint
 tunnel key 123
 tunnel protection ipsec profile cisco
end

R4?interface Tunnel0
 ip address 172.16.1.4 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp authentication gmlabs
 ip nhrp map 172.16.1.2 26.2.6.2
 ip nhrp map multicast 26.2.6.2
 ip nhrp network-id 111
 ip nhrp holdtime 60
 ip nhrp nhs 172.16.1.2
 ip nhrp registration timeout 30
 ip nhrp shortcut
 ip ospf network point-to-multipoint
 tunnel source GigabitEthernet0/0
 tunnel mode gre multipoint
 tunnel key 123
 tunnel protection ipsec profile cisco
end

NAT

R1_Internet#

router ospf 1
 network 1.1.1.1 0.0.0.0 area 0
 network 10.100.1.4 0.0.0.3 area 0
 default-information originate

interface GigabitEthernet0/0
	ip nat outside
interface GigabitEthernet0/3
	ip nat inside
ip nat inside source list 1 interface GigabitEthernet0/0 overload

access-list 1 permit 10.193.54.0 0.0.0.255
access-list 1 permit 10.193.56.0 0.0.0.255
