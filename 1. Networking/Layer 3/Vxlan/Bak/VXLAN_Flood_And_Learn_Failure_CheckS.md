

| Symptom | Command | What Usually Broke |
|---|---|---|
| VNI does not come up | `show nve vni` | Missing `feature nv overlay`, missing `interface nve1`, wrong source interface, or NVE shut |
| VLAN does not map to VXLAN | `show vxlan` | Missing `vn-segment` under VLAN |
| No NVE peers after traffic | `show nve peers` | No host traffic, multicast failure, or underlay reachability failure |
| NVE peer stays down | `show nve peers detail` | Remote VTEP loopback unreachable or multicast group not working |
| Multicast group missing | `show ip mroute <multicast-group>` | PIM not enabled, RP missing, or RPF failure |
| PIM neighbors missing | `show ip pim neighbor` | PIM not enabled on underlay interfaces or routed adjacency down |
| RP missing or wrong | `show ip pim rp` | RP not configured consistently |
| Remote MAC not learned | `show mac address-table dynamic` | No traffic, VNI mismatch, multicast failure, or wrong VLAN-to-VNI mapping |
| Ping fails but VNI is up | `show mac address-table dynamic` | Host VLAN/access-port problem or remote MAC not learned over NVE |
| vPC VTEP inconsistency | `show vpc consistency-parameters global` | NVE source, VNI, VLAN-to-VNI, or secondary loopback mismatch |