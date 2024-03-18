Challenge1:

Configuration changes:
1. R1 - the outgoing interface of r1>r2 static route has been changed from eth3 to eth1.
   
   According to the configured IP address & description on eth1 in r1, eth1 is the interface connecting r1 to r2.
   We can also see that r2's eth1 IP address is in the same subnet as eth1's IP address in r1.
   eth3 is in fact the interface connecting r1 to server1, according to both its IP address configuration & description.
   
3. R1 & R2 - the prefix 172.16.0.0/12 le 32 was removed from the prefix-list MARTIAN.
   
   This prefix covers every IP address in the range 172.16.0.0 - 172.31.255.255 with subnet sizes ranging from 12 to 32.
   All of the subnets containing the IP addresses configured on the servers & hosts in the topology fall within this range, and thus matched by MARTIAN (before removing the prefix of course).
   We can see that in both routers, the route maps ISP-IN and ISP-OUT are configured as in and out route-maps respectively, in the BGP sessions with both ISP routers (R1<>ISP1, and R2<>ISP2).
   In R1, we can see that MARTIAN is matched for a priority 10 (lowest wins) deny statement, in both ISP-IN and ISP-OUT route-maps, which are the in and out route-maps configured for the BGP.
   In R2 its matched only in ISP-IN, but it does not sufficient for full both ways connectivity.
   These deny statements in the route-maps are causing routes to the hosts/servers subnets neither to be advertised (out, only in R1), nor reveiced (in), and thus, no routes between the hosts and the servers are being exchanged over BGP.
   Removing this prefix from the prefix list MARTIAN in both routers excludes the hosts/servers subnets from the deny statement, and the routes proceed to match the permit 30 (permit all) statement & are exchanged over BGP.
   In addition - Martian addresses refer to IP addresses that are reserved and should not appear on the public internet. The aforementioned prefix does not fall in the address ranges widely known as Martian addresses, and so, its removal from the prefix list is safe.


Challenge2:

Configuration changes:
1. R2 - Removed the OSPF passive interface configuration from eth1 which is connected to R1.
   
   An OSPF passive interface configuration makes the interface stop participating in OSPF neighbor discovery and adjacency formation, and in other words, it makes it stop sending "Hello" packets.
   This practice is common for interfaces facing end devices / non layer 3 devices / other routers which are not desired to be included in OSPF.
   When R2's eth1 interface (connecting R2 to R1) is set to be a passive interface, it stops sending OSPF "Hello" packets and so cannot form an OSPF adjacency with R1's eth1 interface.
   As a result, routes are not exchanged between the routers through OSPF, and lacking other routing source, layer 3 inter-subnet connectivity is lost between the end users / servers.

2. R4 & R3 - Changed OSPF area for all interfaces to 0.0.0.0

   OSPF routers only form adjacencies with routers in the same OSPF area (interfaces on both sides should be in the same area).
   In this scenario, we see that all of R4's & R3's interfaces are in area 0.0.0.1, and all of R1's and R2's interfaces are in area 0.0.0.0.
   In order to form the adjacencies, the interfaces connecting R4<>R2 / R3<>R1 should be in the same area - which forces us to change R4&R3 areas to 0.0.0.0.
   In OSPF, area 0.0.0.0, also known as the backbone area, is a requirement for the protocol to function properly - which in our scenario, forces us to choose 0.0.0.0 as the prevailing area.

   
   
