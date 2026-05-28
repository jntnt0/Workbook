
Proper note order

|   |   |   |   |
|---|---|---|---|
|Order|Mechanism Note|Correct Mental Model|Related Lab Names|
|01|IPsec_Underlay_Reachability_And_SA_Baseline.md|IPsec does not fix broken routing. The peers must already reach each other. IKE Phase 1 builds the control-plane SA. IKE Phase 2 builds the IPsec data-plane SAs. Troubleshooting starts by separating peer reachability, IKE SA state, and IPsec packet counters.|ipsec-tunnel-mode-final, ipsec-gre-tunnel-final, ipsec-tunnel-mode-virtual-tunnel-interface-final, ipsec-dynamic-vti-hub-spoke-final, ipsec-easy-vpn-final, ipsec-getvpn-final, ipsec-getvpn-3cGcT3-final, esp-tunnel-mode-through-zbf-final|
|02|Policy_Based_IPsec_Crypto_Map_Tunnel_Mode.md|Classic site-to-site IPsec is policy-based. The crypto ACL defines the proxy IDs, the crypto map binds peer plus transform set plus interesting traffic, and the crypto map must be applied to the correct outside interface. Tunnel mode protects routed traffic between local and remote protected networks.|ipsec-tunnel-mode-final|
|03|ESP_Tunnel_Mode_Through_ZBF.md|ZBF is not part of IPsec negotiation. It is the firewall policy that either allows or kills the IPsec control/data traffic. IKE/NAT-T are control-plane/self-zone flows. ESP is protocol 50 and should usually be explicitly passed, not inspected. Zone-pair direction matters.|esp-tunnel-mode-through-zbf-final|
|04|GRE_Over_IPsec_Tunnel_Protection.md|GRE provides the routed overlay. IPsec provides encryption. GRE is useful when you need multicast, routing protocols, or non-IPsec-friendly overlay behavior. The tunnel interface carries the routing adjacency, while tunnel protection ipsec profile encrypts the GRE traffic.|ipsec-gre-tunnel-final|
|05|GRE_Over_IPsec_Transport_Vs_Tunnel_Mode.md|GRE over IPsec can use IPsec tunnel mode or transport mode. Tunnel mode adds another outer IP header. Transport mode avoids duplicate outer IP headers when GRE already provides the outer tunnel header. The verification point is show crypto ipsec sa showing {Transport} or {Tunnel}.|ipsec-gre-tunnel-final|
|06|Static_VTI_Route_Based_IPsec.md|VTI changes the model from policy-based VPN to route-based VPN. You do not rely on crypto ACLs for every protected subnet. Traffic is encrypted because routing sends it into the tunnel interface. The tunnel destination replaces the crypto-map peer logic.|ipsec-tunnel-mode-virtual-tunnel-interface-final|
|07|Dynamic_VTI_Hub_Spoke_IPsec.md|DVTI is for scalable hub-spoke VPN. The hub uses a virtual-template, and each spoke session creates a virtual-access interface dynamically. Once the interface exists, you can treat it like a routed interface for routing, QoS, firewall policy, and verification.|ipsec-dynamic-vti-hub-spoke-final|
|08|GETVPN_GDOI_Group_Encrypted_Transport.md|GETVPN is not a point-to-point tunnel overlay. It is group encryption over an already-routed private WAN. Group members get keying material from a key server through GDOI. The major mental model is header preservation and any-to-any encryption without changing the WAN routing design. It is a bad fit for NAT-heavy Internet paths.|ipsec-getvpn-final, ipsec-getvpn-3cGcT3-final|
|09|EasyVPN_Hardware_Client_And_Network_Extension_Mode.md|Easy VPN is legacy remote-access/site-extension IPsec. Client mode uses PAT and hides the inside hosts behind a client-assigned address. Network Extension Mode behaves more like site-to-site because real inside subnets remain reachable, but the Easy VPN client still initiates the tunnel.|ipsec-easy-vpn-final|

Do not misfile these

|   |   |   |
|---|---|---|
|Lab|Do Not Treat It As|Correct Placement|
|esp-tunnel-mode-through-zbf-final|A separate IPsec mode|ZBF handling of IPsec/ESP traffic|
|ipsec-tunnel-mode-virtual-tunnel-interface-final|Same thing as crypto-map tunnel mode|Route-based Static VTI|
|ipsec-dynamic-vti-hub-spoke-final|Static VTI with more routers|DVTI hub-spoke scaling model|
|ipsec-getvpn-final and ipsec-getvpn-3cGcT3-final|Two different mechanisms|Same GETVPN note unless the lab failure mode differs|
|ipsec-easy-vpn-final|Modern FlexVPN|Legacy Easy VPN client / NEM model|

Final order to build

01_IPsec_Underlay_Reachability_And_SA_Baseline.md

02_Policy_Based_IPsec_Crypto_Map_Tunnel_Mode.md

03_ESP_Tunnel_Mode_Through_ZBF.md

04_GRE_Over_IPsec_Tunnel_Protection.md

05_GRE_Over_IPsec_Transport_Vs_Tunnel_Mode.md

06_Static_VTI_Route_Based_IPsec.md

07_Dynamic_VTI_Hub_Spoke_IPsec.md

08_GETVPN_GDOI_Group_Encrypted_Transport.md

09_EasyVPN_Hardware_Client_And_Network_Extension_Mode.md

The weak spot in your corpus is GETVPN CLI syntax. The mental model is covered well enough, but when you build the actual copy/paste GETVPN checklist, you will probably need an outside Cisco GETVPN configuration guide unless another project source has the key-server/group-member syntax buried elsewhere.