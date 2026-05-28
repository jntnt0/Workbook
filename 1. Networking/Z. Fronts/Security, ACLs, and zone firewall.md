
|   |   |   |   |
|---|---|---|---|
|Order|Mechanism Note|Mental Model That Must Be Addressed|Related Lab Names|
|01|ACL_Established_Return_Traffic.md|Extended ACLs are stateless filters. The established keyword is only a TCP flag shortcut for return traffic, not true stateful inspection.|access-list-established-final|
|02|Port_Security_Edge_MAC_Enforcement.md|Port security is the first access-edge identity guardrail. It limits which source MAC addresses may use a switchport and defines the violation behavior.|port-security-final|
|03|IP_Source_Guard_DHCP_Snooping_Binding_Enforcement.md|IP Source Guard protects the edge by validating source IP or MAC/IP use against DHCP snooping or static bindings. The binding table is the enforcement truth source.|ip-source-guard-final|
|04|VACL_IP_Filter_VLAN_Local_Filtering.md|A VACL filters traffic inside a VLAN or across VLAN traffic on the switch fabric. It is VLAN-scoped, not interface-directional like a routed ACL.|vacl-ip-filter-final|
|05|uRPF_Ingress_Source_Validation.md|uRPF is anti-spoofing based on the routing/FIB view of the source address. Strict mode expects the return path on the receiving interface; loose mode only requires reachability.|urpf-final|
|06|CBAC_Transparent_Firewall_Stateful_Inspection.md|CBAC is classic IOS stateful inspection. In transparent mode, the device bridges traffic while still inspecting Layer 3 and Layer 4 flows. This is the bridge between ACL thinking and ZBFW thinking.|transparent-firewall-cbac-final|
|07|Zone_Based_Firewall_Routed_Mode.md|ZBFW replaces interface ACL thinking with zone-pair policy thinking. Traffic between zones is denied until a class-map, policy-map, and zone-pair allow inspect, pass, or drop.|zone-based-firewall-final|
|08|Zone_Based_Firewall_Transparent_Mode.md|Transparent ZBFW keeps the firewall in the Layer 2 path while applying zone policy. The key model is: forwarding can be bridged, but enforcement is still zone-pair based.|zone-based-firewall-transparent-mode-final|
|09|Zone_Based_Firewall_URL_Filtering.md|URL filtering is not the firewall foundation. It is a higher-layer classification/enforcement feature that belongs after ZBFW zones, class maps, policy maps, and inspection are understood.|zone-based-firewall-url-filter-final|
|10|Control_Plane_Policing_CoPP.md|CoPP protects the route processor/CPU from punted control-plane traffic. It does not protect transit traffic. It is MQC policing applied under control-plane.|control-plane-policing-final|
|11|Management_Plane_Protection_MPP.md|MPP restricts which interfaces can receive management traffic destined to the device. It is about management entry points, not routing, firewall inspection, or general packet forwarding.|management-plane-protection-final|

Why This Order Is Correct

Start with simple packet admission, then move toward stateful inspection, then finish with device-plane protection.

1. ACL_Established_Return_Traffic.md comes first because it teaches the old-school filter model.
2. Port_Security, IP_Source_Guard, and VACL come next because they are switch-edge and VLAN enforcement tools.
3. uRPF follows because it is source validation using the routing/FIB table.
4. CBAC comes before ZBFW because it is the older IOS stateful firewall model.
5. ZBFW routed mode comes before transparent mode and URL filtering because those are variants/extensions.
6. CoPP and MPP go last because they protect the device itself, not normal transit flows.

Key Mental Model Corrections To Bake Into The Notes

|   |   |   |
|---|---|---|
|Mechanism|Do Not Teach It As|Correct Mental Model|
|ACL established|A real stateful firewall|TCP flag-based return-traffic shortcut|
|Port Security|User authentication|MAC-based switchport admission control|
|IP Source Guard|A firewall|Source IP/MAC validation using bindings|
|VACL|An interface ACL|VLAN-scoped traffic filtering|
|uRPF|A permit/deny ACL replacement|FIB-based anti-spoofing source validation|
|CBAC|Modern ZBFW|Legacy IOS stateful inspection|
|ZBFW|ACLs with different syntax|Zone-pair stateful firewall policy|
|ZBFW Transparent|Same as routed ZBFW with no differences|L2 forwarding path with zone-based enforcement|
|ZBFW URL Filtering|Base firewalling|Higher-layer classification on top of firewall policy|
|CoPP|Data-plane security|CPU/control-plane rate protection|
|MPP|Authentication or AAA|Management-interface exposure control|
