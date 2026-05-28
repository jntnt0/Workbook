| Check | Device               | Command                                        | What You’re Looking For                                            |
| ----: | -------------------- | ---------------------------------------------- | ------------------------------------------------------------------ |
|     1 | All switches         | `show spanning-tree summary`                   | STP mode should be MST                                             |
|     2 | All switches         | `show spanning-tree mst configuration`         | Region name, revision, and VLAN-to-instance map                    |
|     3 | All switches         | `show spanning-tree mst`                       | MST0, MST1, MST2 root bridge, root port, port roles                |
|     4 | All switches         | `show spanning-tree mst 0`                     | CIST / IST behavior and root placement                             |
|     5 | All switches         | `show spanning-tree mst 1`                     | Whether VLANs mapped to instance 1 follow expected topology        |
|     6 | All switches         | `show spanning-tree mst 2`                     | Whether VLANs mapped to instance 2 follow different topology       |
|     7 | Switch links         | `show spanning-tree mst interface g0/x detail` | Port role, state, boundary/internal behavior                       |
|     8 | All switches         | `show interfaces trunk`                        | Trunks are up and carrying the VLANs mapped into MST instances     |
|     9 | All switches         | `show vlan brief`                              | VLANs actually exist on the switches                               |
|    10 | All switches         | `show mac address-table dynamic`               | L2 learning is happening once traffic is generated                 |
|    11 | Neighboring switches | `show cdp neighbors` or `show lldp neighbors`  | Physical/logical neighbor mapping is sane                          |
|    12 | Test failure         | `shutdown` / `no shutdown` on one trunk        | Blocked alternate port should move to forwarding after convergence |
