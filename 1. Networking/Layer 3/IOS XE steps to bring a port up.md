|Step|Action|Command|
|---|---|---|
|1|Check current state|`show ip interface brief`|
|2|Enter config mode|`configure terminal`|
|3|Select interface|`interface GigabitEthernet0/1`|
|4|Enable interface|`no shutdown`|
|5|Verify IP address|`ip address <ip> <mask>`|
|6|Exit interface config|`end`|
|7|Recheck state|`show ip interface brief`|
|8|Save config|`write memory`|


|Interface State|Meaning|
|---|---|
|up/up|Good|
|administratively down/down|Needs `no shutdown`|
|down/down|Physical/link issue|
|up/down|Layer 2 issue|
|up/up but no IP|Interface is up, but not usable for IPv4 routing|