
| Step | Task                                            | Device       | IOS XE Command                           | Expected Result                                   |
| ---- | ----------------------------------------------- | ------------ | ---------------------------------------- | ------------------------------------------------- |
| 1    | Verify Savannah has no route to Chattanooga LAN | SavannahR1   | `show ip route 192.168.10.101`           | `% Network not in table` confirms missing route   |
| 2    | Verify Savannah has no default route            | SavannahR1   | `show ip route 0.0.0.0 0.0.0.0`          | No default route installed                        |
| 3    | Enter config mode                               | SavannahR1   | `conf t`                                 | Global config mode                                |
| 4    | Add default route toward Atlanta                | SavannahR1   | `ip route 0.0.0.0 0.0.0.0 192.168.250.5` | Unknown destinations forward to Atlanta           |
| 5    | Exit config mode                                | SavannahR1   | `end`                                    | Back to privileged EXEC                           |
| 6    | Verify default route                            | SavannahR1   | `show ip route 0.0.0.0 0.0.0.0`          | Default route installed via `192.168.250.5`       |
| 7    | Verify CEF recursion                            | SavannahR1   | `show ip cef 192.168.10.101 detail`      | CEF resolves through `192.168.250.5`              |
| 8    | Test path again                                 | RemoteHMIA_B | `traceroute 192.168.10.101`              | Traffic leaves Savannah instead of returning `!H` |

