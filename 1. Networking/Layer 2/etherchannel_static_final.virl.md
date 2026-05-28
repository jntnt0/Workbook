![[Pasted image 20260513150436.png]]



|Step|Task|Device|Command|What Was Found / Why It Mattered|
|--:|---|---|---|---|
|1|Confirm physical link state|SW1, SW2|`show ip interface brief`|`Gi0/1` and `Gi0/2` were `up/up`, so the physical links between the switches were alive. `Gi0/0` was down, but that was not the EtherChannel issue.|
|2|Check whether EtherChannel existed|SW1, SW2|`show etherchannel summary`|Output showed `Number of channel-groups in use: 0`, meaning no EtherChannel had been created.|
|3|Check Port-channel membership|SW1, SW2|`show etherchannel port-channel`|Empty output confirmed there was no Port-channel interface or member ports.|
|4|Check member interface config|SW1, SW2|`show running-config interface g0/1`|Interface only had `negotiation auto`; no `channel-group` command was present.|
|5|Check whether Port-channel1 existed|SW1, SW2|`show running-config interface port-channel 1`|Command failed because `Port-channel1` did not exist yet.|
|6|Check trunk state|SW1, SW2|`show interfaces trunk`|Blank output showed no active trunks.|
|7|Check VLAN database|SW1, SW2|`show vlan brief`|Only VLAN 1 existed. That was fine for basic testing, but no custom VLANs were present.|
|8|Check STP behavior before fix|SW2|`show spanning-tree`|`Gi0/1` was forwarding and `Gi0/2` was blocking. STP saw two separate links, not one bundled EtherChannel.|
|9|Identify root cause|SW1, SW2|Review of prior outputs|The problem was not physical cabling. The issue was that static EtherChannel was never configured. SW1 logs showed the same condition as SW2: no channel group, no Port-channel, and standalone links.|
|10|Begin EtherChannel configuration|SW2|`interface range g0/1 - 2` then `shutdown`|Member interfaces were shut down first to make the bundle configuration cleaner and avoid inconsistent live changes.|
|11|Attempt to force trunk mode|SW2|`switchport mode trunk`|Command failed: trunk encapsulation was still set to `Auto`, so the switch rejected trunk mode.|
|12|Correct trunk encapsulation issue|SW1, SW2|`switchport trunk encapsulation dot1q`|Dot1Q encapsulation had to be manually set before trunk mode could be applied.|
|13|Configure member interfaces as trunks|SW1, SW2|`switchport mode trunk`|After Dot1Q was set, trunk mode could be applied to `Gi0/1` and `Gi0/2`.|
|14|Create static EtherChannel|SW1, SW2|`channel-group 1 mode on`|This created the static EtherChannel. Since this is `mode on`, there is no LACP or PAgP negotiation.|
|15|Bring member links back up|SW1, SW2|`no shutdown`|The physical links were brought back online after being assigned to the channel group.|
|16|Configure logical Port-channel|SW1, SW2|`interface port-channel 1`|`Port-channel1` was created as the logical bundled interface.|
|17|Configure Port-channel as trunk|SW1, SW2|`switchport trunk encapsulation dot1q` and `switchport mode trunk`|The logical EtherChannel was configured as the trunk. This is the interface STP and trunking should use after bundling.|
|18|Verify EtherChannel formed|SW1, SW2|`show etherchannel summary`|Expected healthy result: `Po1(SU)` with `Gi0/1(P)` and `Gi0/2(P)`.|
|19|Verify trunk moved to Port-channel|SW1, SW2|`show interfaces trunk`|Expected result: `Po1` appears as the trunk instead of separate physical links.|
|20|Verify STP now sees one logical link|SW1, SW2|`show spanning-tree`|Expected result: STP references `Po1`, not separate `Gi0/1` and blocked `Gi0/2`.|
|21|Add temporary SVI on SW1|SW1|`interface vlan 1` / `ip address 10.10.10.1 255.255.255.252` / `no shutdown`|Created a Layer 3 test endpoint on SW1 over VLAN 1.|
|22|Add temporary SVI on SW2|SW2|`interface vlan 1` / `ip address 10.10.10.2 255.255.255.252` / `no shutdown`|Created a Layer 3 test endpoint on SW2 over VLAN 1.|
|23|Test actual communication|SW1 or SW2|`ping 10.10.10.1` or `ping 10.10.10.2`|Ping succeeded, proving the switches can communicate across the EtherChannel path.|
|24|Final conclusion|SW1, SW2|Verification review|The fix worked. The original issue was missing static EtherChannel configuration, plus the need to set Dot1Q encapsulation before forcing trunk mode.|