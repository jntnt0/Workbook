![[IMG_1624.jpeg]]




| Step | Task                                            | Device | Command                                                        | Expected Result                                                    |
| ---- | ----------------------------------------------- | ------ | -------------------------------------------------------------- | ------------------------------------------------------------------ |
| 1    | Verify ASA interfaces are operational           | ASA1   | `show interface ip brief`                                      | `Gig0/0`, `Gig0/1`, and `Gig0/2` show `up/up`                      |
| 2    | Verify interface-to-zone mapping                | ASA1   | `show nameif`                                                  | `INSIDE` mapped to `Gig0/0`, `OUTSIDE` mapped to `Gig0/1`          |
| 3    | Verify inside interface addressing              | ASA1   | `show interface gigabitEthernet0/0`                            | Interface has `192.168.1.254/24`                                   |
| 4    | Verify outside interface addressing             | ASA1   | `show interface gigabitEthernet0/1`                            | Interface has `192.168.2.254/24`                                   |
| 5    | Verify failover/state interface health          | ASA1   | `show interface gigabitEthernet0/2`                            | Failover interface operational with no drops/errors                |
| 6    | Verify R1 interface state                       | R1     | `show ip interface brief`                                      | Connected interface is `up/up` with expected IP                    |
| 7    | Verify R2 interface state                       | R2     | `show ip interface brief`                                      | Connected interface is `up/up` with expected IP                    |
| 8    | Verify R1 can reach ASA inside interface        | R1     | `ping 192.168.1.254`                                           | Successful ICMP reply                                              |
| 9    | Verify R2 can reach ASA outside interface       | R2     | `ping 192.168.2.254`                                           | Successful ICMP reply                                              |
| 10   | Verify R1 has no route to R2 LAN                | R1     | `show ip route 192.168.2.0`                                    | Route missing or unresolved                                        |
| 11   | Add R1 specific static route                    | R1     | `ip route 192.168.2.0 255.255.255.0 192.168.1.254`             | R1 forwards R2 LAN traffic toward ASA                              |
| 12   | Alternative: add R1 default route               | R1     | `ip route 0.0.0.0 0.0.0.0 192.168.1.254`                       | Unknown destinations forwarded toward ASA                          |
| 13   | Verify R2 has no route to R1 LAN                | R2     | `show ip route 192.168.1.0`                                    | Route missing or unresolved                                        |
| 14   | Add R2 specific static route                    | R2     | `ip route 192.168.1.0 255.255.255.0 192.168.2.254`             | R2 forwards R1 LAN traffic toward ASA                              |
| 15   | Alternative: add R2 default route               | R2     | `ip route 0.0.0.0 0.0.0.0 192.168.2.254`                       | Unknown destinations forwarded toward ASA                          |
| 16   | Verify R1 routing table after changes           | R1     | `show ip route`                                                | Route/default route installed correctly                            |
| 17   | Verify R2 routing table after changes           | R2     | `show ip route`                                                | Route/default route installed correctly                            |
| 18   | Verify ASA connected routing                    | ASA1   | `show route`                                                   | Connected routes for `192.168.1.0/24` and `192.168.2.0/24` present |
| 19   | Verify ASA failover state                       | ASA1   | `show failover`                                                | Active unit operational and standby ready                          |
| 20   | Simulate INSIDE to OUTSIDE ICMP flow            | ASA1   | `packet-tracer input INSIDE icmp 192.168.1.1 8 0 192.168.2.2`  | Forward ICMP flow permitted                                        |
| 21   | Simulate OUTSIDE to INSIDE ICMP return flow     | ASA1   | `packet-tracer input OUTSIDE icmp 192.168.2.2 0 0 192.168.1.1` | Return ICMP flow initially dropped                                 |
| 22   | Identify ASA drop reason                        | ASA1   | Review packet-tracer output                                    | `Drop-reason: (acl-drop)` confirms ICMP blocked                    |
| 23   | Enter configuration mode                        | ASA1   | `conf t`                                                       | Global configuration mode                                          |
| 24   | Enter global policy-map                         | ASA1   | `policy-map global_policy`                                     | Policy-map configuration mode                                      |
| 25   | Enter default inspection class                  | ASA1   | `class inspection_default`                                     | Inspection class configuration mode                                |
| 26   | Enable ICMP inspection                          | ASA1   | `inspect icmp`                                                 | ASA tracks ICMP statefully                                         |
| 27   | Exit configuration mode                         | ASA1   | `end`                                                          | Return to privileged EXEC                                          |
| 28   | Save ASA configuration                          | ASA1   | `write memory`                                                 | Configuration saved persistently                                   |
| 29   | Clear stale ASA connections                     | ASA1   | `clear conn`                                                   | Existing connection entries removed                                |
| 30   | Clear stale NAT translations                    | ASA1   | `clear xlate`                                                  | Existing translations removed                                      |
| 31   | Retest end-to-end connectivity                  | R1     | `ping 192.168.2.2`                                             | Successful ICMP reply from R2                                      |
| 32   | Verify ASA connection tracking                  | ASA1   | `show conn`                                                    | ICMP state entry visible                                           |
| 33   | Verify ASA is not dropping packets              | ASA1   | `show asp drop`                                                | No increasing ICMP ACL drops                                       |
| 34   | Re-run OUTSIDE return-flow simulation after fix | ASA1   | `packet-tracer input OUTSIDE icmp 192.168.2.2 0 0 192.168.1.1` | Return ICMP traffic permitted successfully                         |



