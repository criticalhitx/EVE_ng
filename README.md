1. Using Partner Interconnect Layer 2, Establish First EBGP Connection between CloudRouter1 and DRC
2. Advertise CloudRouter VPC Subnets (10.55.1.0/29) to BGP which contain main interface of CSRv1 (G0/0 ip:10.55.1.1)
3. From On Prem(DRC) Loopback 10.20.1.1 to BGP, so this ip is advertise to CloudRouterVPC
4. Create CSRv1 in CloudRouterVPC (G0) and Shared-VPC (G1[10.193.1.2]). CSR have default route (can't be disabled) to CRVPC (10.55.1.3)
5. Establish IPSEC tunnel0 from CSRv1 g0/0(10.55.1.1) to DRC Loopback(10.20.1.1), TunnelIP=192.168.1.1&192.168.1.2
6. Establish overlay EBGP session using tunnel interface. dont advertise 10.20.1.1 to Overlay BGP might cause Recursive Routing
7. Create static routes in CSRv1 to shared-vpc subnets, next hop 10.193.1.1 (via G1), redis to overlay BGP
8. Redirect Internet Connectivity on shared-vpc for scalability to CSRv1 10.193.1.1
9. Do Source NAT on CSRv1 G1 to G0, so Internet traffic have source IP of CSRv1 G0[10.55.1.1] (Cloud NAT only accept source IP inside its VPC[CRVPC]).
Note that traffic destined to On-PREM is directed through Tunnel, so its not NAT-ted
10. Create Cloud Router(IGW) which has NAT function, to translate 10.55.1.0 network overload to G1
