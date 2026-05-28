Use this order. EVN should not be first in the learning path. Cisco describes EVN as building on VRF-Lite, with simpler path isolation, management, and shared-service handling, so VRF-Lite foundations come first.  

01_VRF_Lite_Routing_Table_Isolation.md

Mental model:

VRF-Lite creates separate routing and forwarding tables on the same physical router. The global table is only one routing context. A route, ping, interface, or routing process must be checked inside the correct VRF or you are looking in the wrong place.

  

Related labs:

- vrf-lite-final

- vrf-lite-routing-final

- vrf-lite-route-leaking-static-routes-final

- vrf-lite-route-leaking-bgp-final

- cisco-easy-virtual-network-final

02_VRF_Lite_Interface_Binding_And_8021Q_Transport.md

Mental model:

Interfaces are the boundary where traffic enters a VRF. One interface belongs to one VRF. If one physical link must carry multiple VRFs, use 802.1Q subinterfaces and bind each subinterface to the proper VRF before assigning the IP address.

  

Related labs:

- vrf-lite-final

- vrf-lite-routing-final

- vrf-lite-route-leaking-static-routes-final

- vrf-lite-route-leaking-bgp-final

03_VRF_Lite_Per_VRF_Routing.md

Mental model:

Creating VRFs gives you isolated connected routes, not end-to-end reachability. Each VRF needs its own routing logic: static routes, OSPF process per VRF, EIGRP address family per VRF, or BGP address family per VRF.

  

Related labs:

- vrf-lite-routing-final

- vrf-lite-final

04_VRF_Lite_Static_Route_Leaking.md

Mental model:

Static route leaking is a deliberate exception to VRF isolation. You manually install selected routes between routing contexts. It is simple and good for small labs, but it does not scale well and has weak policy control.

  

Related labs:

- vrf-lite-route-leaking-static-routes-final

05_VRF_Lite_BGP_Route_Leaking_RD_RT.md

Mental model:

BGP route leaking is the policy-based way to share selected routes between VRFs. The RD makes overlapping prefixes unique. Route targets control import and export. The leak works only when export and import policy line up correctly.

  

Related labs:

- vrf-lite-route-leaking-bgp-final

06_Cisco_Easy_Virtual_Network_EVN_VNET_Trunks.md

Mental model:

EVN is VRF-Lite with a simpler operational model. Instead of manually building every VRF subinterface across the core, EVN uses VNET tags and VNET trunks to carry multiple virtual networks across the same infrastructure. Cisco documents EVN trunking with `vrf definition`, `vnet tag`, `vnet trunk`, and `vnet name` workflow.  [oai_citation:1‡Cisco](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/evn/configuration/xe-3s/evn-xe-3s-book/evn-confg.html)

  

Related labs:

- cisco-easy-virtual-network-final

07_EVN_Shared_Services_Route_Replication.md

Mental model:

EVN shared services are not the same mental model as VRF-Lite BGP leaking. In VRF-Lite, shared services commonly use BGP import/export with RD/RT. In EVN, route replication can copy selected routes between virtual networks without BGP, RD, or route targets. Cisco also notes route replication is for static, EIGRP, and OSPF routes, not BGP routes.  [oai_citation:2‡Cisco](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/evn/configuration/xe-3s/evn-xe-3s-book/evn-shared-svcs.html)

  

