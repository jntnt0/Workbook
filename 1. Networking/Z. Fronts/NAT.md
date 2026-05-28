Use 8 mechanism notes. Do not make 14 separate notes. That would fragment the NAT model and make review weaker.

Source basis: _KB_INDEX.md points IOS XE NAT to iosxe_combined_pdfs_.md, lines 2,798 to 3,545. I used Cisco IOS XE NAT docs for the gaps around ALG, route-map NAT, multicast NAT, NVI, and VASI. Cisco’s NAT guide frames the core topics around inside/outside roles, static/dynamic/PAT, inside destination translation, and route-map NAT.  

Proper NAT mechanism note order

01_NAT_Classic_Inside_Outside_Domain_Baseline.md

Mental model:  
Classic IOS NAT is domain-based. You must understand ip nat inside, ip nat outside, inside local, inside global, outside local, and outside global before any static, dynamic, PAT, ALG, or route-map NAT makes sense. NAT does not replace routing. Routing still decides the path, NAT only translates matching traffic as it crosses the marked NAT domains.

Related labs:

- nat-ip-nat-inside-final
- nat-ip-nat-outside-final

Main checks this note should teach:

- Which interface is inside
- Which interface is outside
- Whether the route table sends traffic through the NAT router
- Whether show ip nat statistics shows the expected inside/outside interfaces
- Whether translations appear only after real traffic crosses the router

02_NAT_Static_Extendable_Translations.md

Mental model:  
Static NAT is a fixed translation. extendable matters when the router needs to allow multiple static translations involving the same local or global address, commonly with port-specific mappings or overlapping static entries. The key mental model is uniqueness: NAT entries need enough tuple detail to avoid ambiguity. Static NAT creates a persistent one-to-one mapping, and Cisco documents extendable on static outside source port translations.  

Related labs:

- nat-extendable-final

Main checks this note should teach:

- Static mapping exists before traffic
- The same address can participate in multiple mappings only when the translation is sufficiently specific
- Port-specific static NAT should be verified with protocol and port, not just IP
- show ip nat translations should show the expected static entry

03_NAT_Inside_Destination_Rotary_Translation.md

Mental model:  
Inside destination NAT is not normal outbound source NAT. It changes the destination address, usually to send traffic aimed at one virtual address toward one of several real inside hosts. Think “destination rewrite for service distribution,” not “inside host hides behind public IP.” Cisco’s IOS XE guide ties this to ip nat inside destination-list ... pool ... and rotary pools for TCP load distribution.  

Related labs:

- nat-inside-destination-final

Main checks this note should teach:

- Virtual destination address is matched by ACL
- Real hosts are placed in a rotary NAT pool
- Outside client targets the virtual address
- Router rewrites destination toward a real inside host
- Nonmatching traffic should not be translated

04_NAT_NVI_Domainless_NAT_Model.md

Mental model:  
NVI NAT removes the classic need to mark interfaces as inside or outside. Interfaces are NAT-enabled with ip nat enable, and the translation logic uses NVI behavior instead of classic domain direction. This note should explicitly warn: NVI is a different model from classic ip nat inside/outside, so do not mix the mental models unless the lab is intentionally comparing them. Cisco’s older NAT Virtual Interface documentation says NVI removes the inside/outside domain requirement, applies translation after route decision, and uses ip nat enable; Cisco command reference also provides show ip nat nvi statistics and show ip nat nvi translations.  

Related labs:

- nat-virtual-interface-nat-dynamic-final
- nat-virtual-interface-nat-static-final
- nat-virtual-interface-pat-overload-final
- nat-virtual-interface-pat-static-final

Main checks this note should teach:

- Interfaces use ip nat enable, not ip nat inside/outside
- Dynamic NVI uses a source list and pool
- Static NVI uses a fixed source mapping
- PAT overload uses ports to multiplex many hosts
- Static PAT maps a fixed local service to a fixed global service
- Verification should use NVI-specific show commands when available

05_NAT_Route_Map_Policy_NAT.md

Mental model:  
NAT route maps do not just “permit traffic.” They choose which translation rule applies based on policy. This is where NAT selection becomes conditional: source, destination, interface, or other match criteria can steer traffic into different NAT pools or static mappings. Keep this separate from PBR: PBR changes forwarding decisions, while NAT route maps select translation behavior. Cisco documents NAT route-map support with ip nat inside source route-map ... pool ... reversible, and Cisco’s PBR documentation separately defines route maps as policy tools that influence forwarding.  

Related labs:

- nat-policy-route-maps-final
- nat-policy-route-maps-tXAHEA-final

Main checks this note should teach:

- ACL alone is not the full policy
- Route-map match logic decides which NAT rule wins
- Multiple pools can exist for different traffic classes
- reversible matters when outside-to-inside initiated sessions are expected
- Verify both route-map matches and NAT translations

06_NAT_ALG_DNS_Payload_Rewrite.md

Mental model:  
ALG exists because some protocols carry address or session information inside the payload. Basic NAT rewrites IP and transport headers. ALG inspects deeper and rewrites embedded address or port information when the protocol needs it. DNS ALG belongs here because DNS answers can expose addresses that must align with the translated view. Cisco’s ALG documentation states that protocols embedding IP information in the payload require ALG support for translating embedded IP addresses, ports, or session details.  

Related labs:

- nat-alg-dns-final
- nat-alg-dns2-final

Main checks this note should teach:

- Header NAT and payload NAT are different
- DNS reply contents may need translation
- The NAT table alone may not explain DNS behavior
- Validate with DNS query output, not just ping
- Know when ALG helps and when it causes misleading behavior

07_NAT_Multicast_Dynamic_Source_Translation.md

Mental model:  
Multicast NAT is a special case. It is not normal unicast PAT. Cisco’s IP Multicast Dynamic NAT supports source address translation of multicast packets and establishes one-to-one mappings. It does not support multicast destination translation, PAT overload for multicast, source-and-destination translation, or unicast-to-multicast translation.  

Related labs:

- nat-multicast-final

Main checks this note should teach:

- Multicast routing must work first
- PIM must be enabled where required
- NAT translates multicast source, not multicast group destination
- ACLs may need to permit both pre-NAT and post-NAT source addresses
- Verify multicast state and NAT state separately

08_NAT_VASI_Inter_VRF_Service_Path_NAT.md

Mental model:  
VASI is not ordinary NAT and not the same thing as NVI. VASI is a service-path mechanism used on IOS XE to push traffic between VRFs through services such as NAT, firewall, or IPsec. The key model is “virtual service cable between VRFs.” Cisco’s VASI NAT document states that IOS XE does not support classic inter-VRF NAT the same way older IOS did, and that inter-VRF NAT on IOS XE is achieved through VASI.  

Related labs:

- nat-vasi-final

Main checks this note should teach:

- VRF route leaking alone is not the same as VASI NAT
- Traffic must enter one VASI side and leave the other
- NAT is applied along the forced service path
- Verify VRF routing, VASI interfaces, and NAT translations together
- Treat VASI as a service insertion mechanism, not as basic edge NAT

Final order summary

01_NAT_Classic_Inside_Outside_Domain_Baseline.md

  - nat-ip-nat-inside-final

  - nat-ip-nat-outside-final

  

02_NAT_Static_Extendable_Translations.md

  - nat-extendable-final

  

03_NAT_Inside_Destination_Rotary_Translation.md

  - nat-inside-destination-final

  

04_NAT_NVI_Domainless_NAT_Model.md

  - nat-virtual-interface-nat-dynamic-final

  - nat-virtual-interface-nat-static-final

  - nat-virtual-interface-pat-overload-final

  - nat-virtual-interface-pat-static-final

  

05_NAT_Route_Map_Policy_NAT.md

  - nat-policy-route-maps-final

  - nat-policy-route-maps-tXAHEA-final

  

06_NAT_ALG_DNS_Payload_Rewrite.md

  - nat-alg-dns-final

  - nat-alg-dns2-final

  

07_NAT_Multicast_Dynamic_Source_Translation.md

  - nat-multicast-final

  

08_NAT_VASI_Inter_VRF_Service_Path_NAT.md

  - nat-vasi-final

That order is the cleanest because it goes from classic NAT foundations, to translation types, to destination NAT, to domainless NVI, to policy NAT, then to exceptions: ALG, multicast, and VASI.