
| Step | Task                                       | Device       | IOS XE Command                                         | Expected Result                                              |
| ---- | ------------------------------------------ | ------------ | ------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | Verify Atlanta lacks route to Savannah LAN | AtlantaR1    | `show ip route 192.168.50.101`                         | `% Network not in table` confirms missing return route       |
| 2    | Check existing static routes               | AtlantaR1    | `show running-config \| include ^ip route`             | Only route to `192.168.10.0/24` is present                   |
| 3    | Enter config mode                          | AtlantaR1    | `conf t`                                               | Global config mode                                           |
| 4    | Add static route to Savannah LAN           | AtlantaR1    | `ip route 192.168.50.0 255.255.255.0 192.168.250.6 10` | Atlanta knows how to reach `192.168.50.0/24`                 |
| 5    | Exit config mode                           | AtlantaR1    | `end`                                                  | Back to privileged EXEC                                      |
| 6    | Verify route is installed                  | AtlantaR1    | `show ip route 192.168.50.101`                         | Static route via `192.168.250.6` appears                     |
| 7    | Verify CEF entry                           | AtlantaR1    | `show ip cef 192.168.50.101 detail`                    | CEF recursively resolves via `192.168.250.6`                 |
| 8    | Re-test end-to-end path                    | RemoteHMIA_B | `traceroute 192.168.10.101`                            | Expected path: Savannah → Atlanta → Chattanooga → RemoteHMIA |




|Device|Required Route|Purpose|
|---|---|---|
|SavannahR1|`ip route 0.0.0.0 0.0.0.0 192.168.250.5`|Send nonlocal traffic toward Atlanta|
|AtlantaR1|`ip route 192.168.10.0 255.255.255.0 192.168.250.2 10`|Reach Chattanooga LAN|
|AtlantaR1|`ip route 192.168.50.0 255.255.255.0 192.168.250.6 10`|Reach Savannah LAN|