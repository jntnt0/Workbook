Yes, but not in the old “manually babysit every blocked port” way.

  

Modern enterprise topologies usually still have STP intentionally configured, but the design tries to make STP boring.

|   |   |
|---|---|
|Modern Design Pattern|STP Role|
|Layer 2 access with Layer 3 distribution/core|STP exists mostly at the access layer as a safety net|
|Routed access layer|STP is minimized because uplinks are Layer 3, not trunks|
|VSS / StackWise Virtual / vPC / MLAG designs|STP still runs, but port-channels reduce blocking and make dual-homing cleaner|
|Fabric designs like SDA / VXLAN EVPN|STP is usually pushed to the edge or avoided inside the fabric|
|Small/simple campus|STP is still manually shaped because L2 loops are realistic|

What is usually manually configured:

|   |   |
|---|---|
|STP Item|Common Modern Practice|
|STP mode|Set explicitly, often Rapid PVST+ or MST|
|Root bridge|Manually set primary/secondary root|
|Edge ports|PortFast enabled|
|BPDU Guard|Enabled on access/edge ports|
|Root Guard|Used on ports where downstream switches should never become root|
|Loop Guard|Used selectively on non-designated links|
|Trunk allowed VLANs|Manually restricted|
|EtherChannel/LACP|Used so STP sees one logical link|

What is usually not done anymore:

|   |   |
|---|---|
|Old/Bad Practice|Why|
|Letting default bridge priority decide root|Accidental root bridge risk|
|Extending VLANs everywhere|Creates huge L2 failure domains|
|Relying on STP as the main redundancy design|Slow, messy, fragile compared to routed/fabric designs|
|Manually forcing STP cost everywhere without reason|Hard to maintain unless design requires it|

The real answer:

  

Modern networks usually configure STP intentionally, but they design the network so STP rarely has to make dramatic decisions.

  

For your lab, you should absolutely manually configure:

|   |   |
|---|---|
|Role|Recommended STP Config|
|Intended root switch|spanning-tree vlan <vlan-list> root primary|
|Backup/root secondary|spanning-tree vlan <vlan-list> root secondary|
|Host/access ports|spanning-tree portfast and spanning-tree bpduguard enable|
|Switch uplinks|No PortFast, trunk correctly, LACP if bundled|
|Downstream switch-facing ports|Consider Root Guard if downstream should never become root|

Bottom line: yes, configure STP manually enough to control root placement and protect edge ports. But don’t build a topology where STP is your primary traffic-engineering tool.