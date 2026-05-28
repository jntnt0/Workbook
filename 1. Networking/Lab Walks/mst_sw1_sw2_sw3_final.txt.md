Run these on SW1, SW2, and SW3 unless the task says otherwise.

|   |   |
|---|---|
|Checklist item|Shell command|
|Verify switch is running MST|show spanning-tree summary|
|Verify MST region name, revision, and VLAN mapping|show spanning-tree mst config|
|Verify current MST instance state|show spanning-tree mst|
|Verify MST instance 2 exists and is active|show spanning-tree mst 2|
|Verify MST instance 3 exists and is active|show spanning-tree mst 3|
|Confirm VLANs actually exist locally|show vlan brief|
|Create VLANs mapped to MST instance 2 and 3|conf tvlan 10vlan 20vlan 30vlan 40vlan 50vlan 60end|
|Verify VLANs are now active|show vlan brief|
|Verify trunk status|show interfaces trunk|
|Verify trunks allow all required VLANs|show interfaces trunk|
|Verify VLANs are active on trunks|show interfaces trunk|
|Verify MST port role and state on G0/1|show spanning-tree mst interface g0/1 detail|
|Verify MST port role and state on G0/2|show spanning-tree mst interface g0/2 detail|
|Verify CDP neighbor mapping|show cdp neighbors|
|Verify interface status|show ip interface brief|
|Verify physical interface health on G0/1|show interfaces g0/1|
|Verify physical interface health on G0/2|show interfaces g0/2|
|Verify dynamic MAC learning|show mac address-table dynamic|
|Save working config|write memory|

The main fix command set is this:

conf t

vlan 10

vlan 20

vlan 30

vlan 40

vlan 50

vlan 60

end

write memory

Then check:

show vlan brief

show interfaces trunk

show spanning-tree mst 2

show spanning-tree mst 3