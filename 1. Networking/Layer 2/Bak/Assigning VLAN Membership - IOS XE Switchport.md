| Step | Task                    | IOS XE Command                              |
| ---- | ----------------------- | ------------------------------------------- |
| 1    | Check existing VLANs    | `show vlan brief`                           |
| 2    | Create VLAN if missing  | `conf t` → `vlan <vlan-id>` → `name <name>` |
| 3    | Select access interface | `interface <interface>`                     |
| 4    | Make it an access port  | `switchport mode access`                    |
| 5    | Assign VLAN             | `switchport access vlan <vlan-id>`          |
| 6    | Enable port             | `no shutdown`                               |
| 7    | Verify membership       | `show vlan brief`                           |
| 8    | Verify port config      | `show running-config interface <interface>` |



|Step|Command|
|---|---|
|Enter config|`conf t`|
|Create VLAN|`vlan 100`|
|Name VLAN|`name SERVER_LAN`|
|Select port|`interface e0/0`|
|Access mode|`switchport mode access`|
|Assign VLAN|`switchport access vlan 100`|
|Enable port|`no shutdown`|
|Finish|`end`|
|Verify|`show vlan brief`|



|Config Line|Meaning|
|---|---|
|`switchport mode access`|Port carries one VLAN|
|`switchport access vlan 100`|Port belongs to VLAN 100|
|`no shutdown`|Port enabled|