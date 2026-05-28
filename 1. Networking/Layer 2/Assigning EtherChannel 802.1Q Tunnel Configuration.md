etherchannel_8021q_tunnel_final.virl
![[Pasted image 20260513142709.png]]

|Step|Task|Device|Command|What You’re Looking For|
|---|---|---|---|---|

|   |   |   |   |   |
|---|---|---|---|---|
|1|Confirm router interfaces were dead before troubleshooting deeper|R1, R2|`show ip interface brief`|Router interfaces were `administratively down/down` and had no IPs, meaning ARP and ping could not work yet|

|   |   |   |   |   |
|---|---|---|---|---|
|2|Confirm routers had no ARP entries|R1, R2|`show ip arp`|Empty ARP table was expected because the router interfaces were down and unaddressed|

|   |   |   |   |   |
|---|---|---|---|---|
|3|Confirm switch physical links existed|SW1, SW2, SW3, SW4|`show ip interface brief`|Switch links in the topology should show `up/up` before worrying about VLANs, trunks, or QinQ|

|   |   |   |   |   |
|---|---|---|---|---|
|4|Confirm EtherChannels were missing|SW1, SW2, SW3, SW4|`show etherchannel summary`|Initially showed `Number of channel-groups in use: 0`, meaning no Port-channel existed|

|   |   |   |   |   |
|---|---|---|---|---|
|5|Confirm trunks were missing|SW1, SW2, SW3, SW4|`show interfaces trunk`|Empty output meant no active trunk links were carrying VLANs|

|   |   |   |   |   |
|---|---|---|---|---|
|6|Confirm VLANs were missing|SW1, SW2, SW3, SW4|`show vlan brief`|Initially only VLAN 1 existed, so customer/service VLANs had to be created|

|   |   |   |   |   |
|---|---|---|---|---|
|7|Confirm Port-channel interface did not exist yet|SW1, SW2, SW3, SW4|`show running-config interface port-channel 1`|Invalid input or missing config confirmed the Port-channel had not been created|

|   |   |   |   |   |
|---|---|---|---|---|
|8|Configure R1 LAN interface|R1|`interface g0/1`  <br>`ip address 192.168.10.1 255.255.255.0`  <br>`no shutdown`|R1 interface should become `up/up` with the correct IP|

|   |   |   |   |   |
|---|---|---|---|---|
|9|Configure R2 LAN interface|R2|`interface g0/1`  <br>`ip address 192.168.10.2 255.255.255.0`  <br>`no shutdown`|R2 interface should become `up/up` with the correct IP|

|   |   |   |   |   |
|---|---|---|---|---|
|10|Create customer VLAN on left customer switch|SW1|`vlan 10`  <br>`name CUSTOMER`|VLAN 10 should appear in `show vlan brief`|

|   |   |   |   |   |
|---|---|---|---|---|
|11|Configure SW1 router-facing access port|SW1|`interface g0/3`  <br>`switchport mode access`  <br>`switchport access vlan 10`  <br>`no shutdown`|R1 traffic should enter customer VLAN 10|

|   |   |   |   |   |
|---|---|---|---|---|
|12|Configure SW1 EtherChannel member links|SW1|`interface range g0/1 - 2`  <br>`switchport trunk encapsulation dot1q`  <br>`switchport mode trunk`  <br>`switchport trunk allowed vlan 10`  <br>`channel-group 1 mode on`  <br>`no shutdown`|G0/1 and G0/2 should bundle into Port-channel1|

|   |   |   |   |   |
|---|---|---|---|---|
|13|Configure SW1 Port-channel trunk|SW1|`interface port-channel1`  <br>`switchport trunk encapsulation dot1q`  <br>`switchport mode trunk`  <br>`switchport trunk allowed vlan 10`|Po1 should trunk customer VLAN 10 toward SW2|

|   |   |   |   |   |
|---|---|---|---|---|
|14|Create provider service VLAN on SW2|SW2|`vlan 100`  <br>`name SERVICE_VLAN`|VLAN 100 should exist on SW2|

|   |   |   |   |   |
|---|---|---|---|---|
|15|Configure SW2 customer-facing QinQ EtherChannel members|SW2|`interface range g0/1 - 2`  <br>`switchport access vlan 100`  <br>`switchport mode dot1q-tunnel`  <br>`channel-group 1 mode on`  <br>`no shutdown`|SW2 should treat the customer-facing bundle as a QinQ tunnel UNI|

|   |   |   |   |   |
|---|---|---|---|---|
|16|Configure SW2 QinQ Port-channel|SW2|`interface port-channel1`  <br>`switchport access vlan 100`  <br>`switchport mode dot1q-tunnel`|Customer VLAN 10 should be encapsulated inside provider service VLAN 100|

|   |   |   |   |   |
|---|---|---|---|---|
|17|Configure SW2 provider core trunk|SW2|`interface g0/3`  <br>`switchport trunk encapsulation dot1q`  <br>`switchport mode trunk`  <br>`switchport trunk allowed vlan 100`  <br>`no shutdown`|SW2-SW3 trunk should carry service VLAN 100|

|   |   |   |   |   |
|---|---|---|---|---|
|18|Create provider service VLAN on SW3|SW3|`vlan 100`  <br>`name SERVICE_VLAN`|VLAN 100 should exist on SW3|

|   |   |   |   |   |
|---|---|---|---|---|
|19|Configure SW3 provider core trunk|SW3|`interface g0/3`  <br>`switchport trunk encapsulation dot1q`  <br>`switchport mode trunk`  <br>`switchport trunk allowed vlan 100`  <br>`no shutdown`|SW3 should receive service VLAN 100 from SW2|

|   |   |   |   |   |
|---|---|---|---|---|
|20|Configure SW3 customer-facing QinQ EtherChannel members|SW3|`interface range g0/1 - 2`  <br>`switchport access vlan 100`  <br>`switchport mode dot1q-tunnel`  <br>`channel-group 1 mode on`  <br>`no shutdown`|SW3 should form the right-side provider QinQ bundle|

|   |   |   |   |   |
|---|---|---|---|---|
|21|Configure SW3 QinQ Port-channel|SW3|`interface port-channel1`  <br>`switchport access vlan 100`  <br>`switchport mode dot1q-tunnel`|SW3 should decapsulate/forward customer VLAN traffic toward SW4|

|   |   |   |   |   |
|---|---|---|---|---|
|22|Create customer VLAN on right customer switch|SW4|`vlan 10`  <br>`name CUSTOMER`|VLAN 10 should appear on SW4|

|   |   |   |   |   |
|---|---|---|---|---|
|23|Configure SW4 EtherChannel member links|SW4|`interface range g0/1 - 2`  <br>`switchport trunk encapsulation dot1q`  <br>`switchport mode trunk`  <br>`switchport trunk allowed vlan 10`  <br>`channel-group 1 mode on`  <br>`no shutdown`|SW4 links toward SW3 should bundle into Port-channel1|

|   |   |   |   |   |
|---|---|---|---|---|
|24|Configure SW4 Port-channel trunk|SW4|`interface port-channel1`  <br>`switchport trunk encapsulation dot1q`  <br>`switchport mode trunk`  <br>`switchport trunk allowed vlan 10`|Po1 should trunk customer VLAN 10 toward SW3|

|   |   |   |   |   |
|---|---|---|---|---|
|25|Configure SW4 router-facing access port|SW4|`interface g0/3`  <br>`switchport mode access`  <br>`switchport access vlan 10`  <br>`no shutdown`|R2 traffic should enter customer VLAN 10|

|   |   |   |   |   |
|---|---|---|---|---|
|26|Verify router interfaces after config|R1, R2|`show ip interface brief`|R1 and R2 G0/1 should be `up/up` with 192.168.10.1 and 192.168.10.2|

|   |   |   |   |   |
|---|---|---|---|---|
|27|Verify EtherChannel formation|SW1, SW2, SW3, SW4|`show etherchannel summary`|Port-channel1 should exist and member links should show bundled, usually `P` with Po1 `SU`|

|   |   |   |   |   |
|---|---|---|---|---|
|28|Check for disabled member ports|SW1, SW2, SW3, SW4|`show interface status err-disabled`|No EtherChannel member should be err-disabled|

|   |   |   |   |   |
|---|---|---|---|---|
|29|Verify customer trunks|SW1, SW4|`show interfaces trunk`|Po1 should be trunking and allowing VLAN 10|

|   |   |   |   |   |
|---|---|---|---|---|
|30|Verify provider trunk|SW2, SW3|`show interfaces trunk`|G0/3 should be trunking and allowing VLAN 100|

|   |   |   |   |   |
|---|---|---|---|---|
|31|Verify customer VLAN presence|SW1, SW4|`show vlan brief`|VLAN 10 should exist and router-facing ports should be assigned correctly|

|   |   |   |   |   |
|---|---|---|---|---|
|32|Verify provider service VLAN presence|SW2, SW3|`show vlan brief`|VLAN 100 should exist on both provider switches|

|   |   |   |   |   |
|---|---|---|---|---|
|33|Verify QinQ tunnel mode|SW2, SW3|`show running-config interface port-channel1`|Po1 should show `switchport access vlan 100` and `switchport mode dot1q-tunnel`|

|   |   |   |   |   |
|---|---|---|---|---|
|34|Verify customer Port-channel trunk config|SW1, SW4|`show running-config interface port-channel1`|Po1 should show trunk mode and allow VLAN 10|

|   |   |   |   |   |
|---|---|---|---|---|
|35|Verify STP did not block the only valid path|SW1, SW2, SW3, SW4|`show spanning-tree vlan 10`  <br>`show spanning-tree vlan 100`|Customer VLAN should be sane on SW1/SW4. Service VLAN should be sane on SW2/SW3|

|   |   |   |   |   |
|---|---|---|---|---|
|36|Verify MAC learning near routers|SW1, SW4|`show mac address-table dynamic`|R1 MAC should appear on SW1 toward G0/3. R2 MAC should appear on SW4 toward G0/3|

|   |   |   |   |   |
|---|---|---|---|---|
|37|Verify MAC learning through provider path|SW2, SW3|`show mac address-table dynamic`|MACs should be learned through Po1 and G0/3 in the expected direction|

|   |   |   |   |   |
|---|---|---|---|---|
|38|Test end-to-end ICMP|R1|`ping 192.168.10.2`|Ping should succeed from R1 to R2|

|   |   |   |   |   |
|---|---|---|---|---|
|39|Test reverse ICMP|R2|`ping 192.168.10.1`|Ping should succeed from R2 to R1|

|   |   |   |   |   |
|---|---|---|---|---|
|40|Confirm ARP resolution|R1, R2|`show ip arp`|Each router should have the other router’s IP mapped to a MAC address|

|   |   |   |   |   |
|---|---|---|---|---|
|41|Confirm same-VLAN behavior|R1 or R2|`traceroute <opposite-router-ip>`|Since R1 and R2 are in the same stretched customer VLAN, there should not be an extra routed hop|

|   |   |   |   |   |
|---|---|---|---|---|
|42|Save working configs|R1, R2, SW1, SW2, SW3, SW4|`copy running-config startup-config`|Working state survives reload|