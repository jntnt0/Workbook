Do not make one note per lab. That would be messy and redundant. Make mechanism notes, then attach the labs underneath.

Source_Basis

|   |   |
|---|---|
|Source|Use|
|_KB_INDEX.md|Points IPv6 addressing, DHCPv6, ND, RA Guard, and SLAAC to Ios-xs_combined_epubs.md; points broader IPv6 routing and transition topics to All_combined_part3.md|
|Ios-xs_combined_epubs.md|IPv6 addressing services, SLAAC, DHCPv6, ND, RA Guard, IPv6 snooping|
|CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md|EIGRPv6, OSPFv3, DHCPv6 troubleshooting, IPv6 first-hop security|
|All_combined_part3.md|CCIE R&S Foundations IPv6 chapter: addressing, OSPFv3, EIGRPv6, IPv6 transition mechanisms|
|CiscoPress_combined_part1.md|Higher-level IPv6 transition design notes for ISATAP and 6rd|

Proper IPv6 Mechanism Note Order

|   |   |   |   |
|---|---|---|---|
|Order|Mechanism Note|Mental Model To Address|Related Labs|
|01|IPv6_Addressing_Link_Local_And_Unicast_Routing.md|IPv6 forwarding is not just “put an address on an interface.” Global unicast, link-local, local routes, connected routes, and ipv6 unicast-routing all matter. Link-local is not optional operationally.|ipv6-r1-r2-final|
|02|IPv6_ND_SLAAC_DAD_And_Router_Advertisements.md|IPv6 does not use ARP. Neighbor Discovery, Router Solicitation, Router Advertisement, Duplicate Address Detection, and solicited-node multicast are the real foundation. RA provides default gateway behavior.|ipv6-router-advertisement-final|
|03|IPv6_DHCPv6_Stateful_Stateless_Server_And_Relay.md|DHCPv6 does not replace RA for default gateway discovery. RA flags tell hosts whether to use SLAAC, stateless DHCPv6, or stateful DHCPv6. DHCPv6 supplies addresses/options, not the default router.|ipv6-dhcp-finalipv6-dhcpv6-server-final|
|04|IPv6_Prefix_Delegation_And_General_Prefixes.md|Do not hard-code everything when the upstream prefix may change. Prefix delegation gives a downstream router a prefix block; general prefixes let interfaces inherit from a named prefix.|ipv6-prefix-delegation-finalipv6-general-prefixes-final|
|05|IPv6_ACL_And_ICMPv6_Control_Plane.md|IPv6 ACLs can break the network fast if they block ICMPv6. ND, RA, RS, DAD, PMTUD, and routing protocol traffic must be understood before filtering.|ipv6-access-list-final|
|06|IPv6_First_Hop_Security_RA_Guard_And_ND_Inspection.md|First-hop security protects the access edge from rogue IPv6 control-plane behavior. RA Guard blocks unauthorized routers; ND inspection/snooping builds trust around neighbor bindings.|ipv6-ra-guard-finalipv6-nd-inspection-final|
|07|RIPng_Interface_Based_IPv6_Routing.md|RIPng is RIP for IPv6, but enabled per interface and carried over IPv6. It uses IPv6 multicast and link-local adjacency behavior. Treat it as legacy but useful for learning IPv6 routing basics.|ipv6-ripng-final|
|08|OSPFv3_IPv6_Routing_And_Default_Route.md|OSPFv3 is link-based, not subnet-based like old OSPF thinking. It uses IPv6 link-local addresses for neighbor communication and still needs a 32-bit router ID. Default route injection is a separate control-plane act.|ipv6-ospf-three-routers-finalipv6-ospfv3-default-route-final|
|09|EIGRPv6_Link_Local_Neighbors_Classic_And_Named_Mode.md|EIGRPv6 keeps EIGRP logic, but neighbor identity and next-hop behavior lean heavily on link-local addresses. Classic mode requires interface enablement; named mode can behave differently. Router ID still matters.|ipv6-eigrp-final|
|10|IPv6_Manual_GRE_Over_IPv4.md|GRE does not fix broken underlay reachability. IPv6 is the passenger, IPv4 is the transport, and the tunnel interface is the overlay link. Verify IPv4 reachability before blaming IPv6.|ipv6-over-ipv4-gre-final|
|11|IPv6_Automatic_6to4_Tunneling.md|6to4 is automatic IPv6-over-IPv4 tunneling using the 2002::/16 prefix and embedded IPv4 addressing. It is a transition mechanism, not a clean modern enterprise design.|ipv6-automatic-6to4-tunneling-final|
|12|IPv6_ISATAP_Intra_Site_Tunneling.md|ISATAP is IPv6-over-IPv4 for an internal site where the IPv4 network acts like a virtual NBMA link. It is a host/site transition mechanism, not the same thing as GRE.|ipv6-isatap-final|
|13|IPv6_6RD_Rapid_Deployment.md|6rd is provider-style rapid IPv6 deployment over an IPv4 access network. It resembles 6to4 conceptually, but uses the provider’s IPv6 prefix and border relay model instead of public 2002::/16 behavior.|ipv6-6rd-finalipv6-6rd-rapid-deployment-finalipv6-rd-final|
|14|IPv6_NPTv6_Prefix_Translation.md|NPTv6 is not NAT overload. It translates one IPv6 prefix to another while preserving host bits. Think prefix translation, not port translation.|ipv6-nptv6-final|

Clean Grouping Decision

ipv6-rd-final should go under IPv6_6RD_Rapid_Deployment.md unless the actual lab config proves it means something else. In this lab set, it almost certainly means rapid deployment, not route distinguisher.

Final Note Sequence

01_IPv6_Addressing_Link_Local_And_Unicast_Routing.md

02_IPv6_ND_SLAAC_DAD_And_Router_Advertisements.md

03_IPv6_DHCPv6_Stateful_Stateless_Server_And_Relay.md

04_IPv6_Prefix_Delegation_And_General_Prefixes.md

05_IPv6_ACL_And_ICMPv6_Control_Plane.md

06_IPv6_First_Hop_Security_RA_Guard_And_ND_Inspection.md

07_RIPng_Interface_Based_IPv6_Routing.md

08_OSPFv3_IPv6_Routing_And_Default_Route.md

09_EIGRPv6_Link_Local_Neighbors_Classic_And_Named_Mode.md

10_IPv6_Manual_GRE_Over_IPv4.md

11_IPv6_Automatic_6to4_Tunneling.md

12_IPv6_ISATAP_Intra_Site_Tunneling.md

13_IPv6_6RD_Rapid_Deployment.md

14_IPv6_NPTv6_Prefix_Translation.md