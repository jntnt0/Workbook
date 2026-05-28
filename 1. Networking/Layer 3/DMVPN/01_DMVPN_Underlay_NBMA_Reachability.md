# DMVPN_Underlay_NBMA_Reachability_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Underlay | The real transport network that carries GRE/NHRP/DMVPN packets |
| NBMA address | The physical or transport-facing IP address used to reach another DMVPN router |
| Tunnel address | The overlay address assigned to the DMVPN tunnel interface |
| Tunnel source | The local transport-facing interface or IP used to source GRE packets |
| Transit cloud | The Internet, MPLS, WAN, or simulated provider network between hub and spokes |
| Default route | Simple transport route that points DMVPN routers toward the transit cloud |
| Hard rule | DMVPN does not fix broken transport. Every router must reach remote NBMA addresses before tunnel/NHRP work begins |
# DMVPN_Underlay_NBMA_Reachability_Configuration_Checklist
| Step | Task                                                                                 | Device               | Command                                                                                          | Expected Result                                                                                                |
| ---: | ------------------------------------------------------------------------------------ | -------------------- | ------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------- |
|    1 | Identify each router’s NBMA-facing interface                                         | Hub, Spokes          | `show ip interface brief`                                                                        | The interface facing the transport cloud is known                                                              |
|    2 | Confirm the NBMA addressing plan                                                     | Hub, Spokes, Transit | Addressing check                                                                                 | Each router has a unique transport/NBMA IP and correct next-hop gateway                                        |
|    3 | Confirm the tunnel source candidate is up                                            | Hub, Spokes          | `show ip interface brief`                                                                        | Tunnel source interface shows `up/up`                                                                          |
|    4 | Configure the transit switch as a Layer 3 transport cloud if used                    | Transit SW           | `ip routing`                                                                                     | Transit switch can route between NBMA subnets                                                                  |
|    5 | Convert transit-facing switch ports to routed ports if using IOSv-L2/L3 switch model | Transit SW           | `interface <port>`<br>`no switchport`<br>`no shutdown`                                           | Interface operates as a routed port                                                                            |
|    6 | Configure transit interface toward the hub                                           | Transit SW           | `interface <hub-facing-port>`<br>`ip address <hub-gateway-ip> <mask>`<br>`no shutdown`           | Hub has a reachable gateway on the transport cloud                                                             |
|    7 | Configure transit interface toward Spoke 1                                           | Transit SW           | `interface <spoke1-facing-port>`<br>`ip address <spoke1-gateway-ip> <mask>`<br>`no shutdown`     | Spoke 1 has a reachable gateway on the transport cloud                                                         |
|    8 | Configure transit interface toward Spoke 2                                           | Transit SW           | `interface <spoke2-facing-port>`<br>`ip address <spoke2-gateway-ip> <mask>`<br>`no shutdown`     | Spoke 2 has a reachable gateway on the transport cloud                                                         |
|    9 | Configure transit interface toward additional spokes if present                      | Transit SW           | `interface <spokeN-facing-port>`<br>`ip address <spokeN-gateway-ip> <mask>`<br>`no shutdown`     | Each spoke has a reachable transport gateway                                                                   |
|   10 | Configure the hub NBMA-facing interface                                              | Hub                  | `interface <hub-underlay-interface>`<br>`ip address <hub-nbma-ip> <mask>`<br>`no shutdown`       | Hub transport interface is addressed and up                                                                    |
|   11 | Configure Spoke 1 NBMA-facing interface                                              | Spoke 1              | `interface <spoke1-underlay-interface>`<br>`ip address <spoke1-nbma-ip> <mask>`<br>`no shutdown` | Spoke 1 transport interface is addressed and up                                                                |
|   12 | Configure Spoke 2 NBMA-facing interface                                              | Spoke 2              | `interface <spoke2-underlay-interface>`<br>`ip address <spoke2-nbma-ip> <mask>`<br>`no shutdown` | Spoke 2 transport interface is addressed and up                                                                |
|   13 | Configure additional spoke NBMA-facing interfaces if present                         | Spokes               | `interface <spokeN-underlay-interface>`<br>`ip address <spokeN-nbma-ip> <mask>`<br>`no shutdown` | Each spoke transport interface is addressed and up                                                             |
|   14 | Add hub default route toward the transport cloud                                     | Hub                  | `ip route 0.0.0.0 0.0.0.0 <hub-transit-gateway-ip>`                                              | Hub can forward unknown transport destinations toward the cloud                                                |
|   15 | Add Spoke 1 default route toward the transport cloud                                 | Spoke 1              | `ip route 0.0.0.0 0.0.0.0 <spoke1-transit-gateway-ip>`                                           | Spoke 1 can reach remote NBMA addresses                                                                        |
|   16 | Add Spoke 2 default route toward the transport cloud                                 | Spoke 2              | `ip route 0.0.0.0 0.0.0.0 <spoke2-transit-gateway-ip>`                                           | Spoke 2 can reach remote NBMA addresses                                                                        |
|   17 | Add additional spoke default routes if present                                       | Spokes               | `ip route 0.0.0.0 0.0.0.0 <spokeN-transit-gateway-ip>`                                           | Each spoke can reach remote NBMA addresses                                                                     |
|   18 | Confirm interface state on all transport links                                       | Hub, Spokes, Transit | `show ip interface brief`                                                                        | All underlay interfaces show `up/up`                                                                           |
|   19 | Confirm the hub has a transport route to each spoke NBMA                             | Hub                  | `show ip route <spoke-nbma-ip>`                                                                  | Route resolves through the transport cloud                                                                     |
|   20 | Confirm each spoke has a transport route to the hub NBMA                             | Spokes               | `show ip route <hub-nbma-ip>`                                                                    | Route resolves through the transport cloud                                                                     |
|   21 | Confirm spoke-to-spoke NBMA routing if Phase 2 or Phase 3 will be used               | Spokes               | `show ip route <other-spoke-nbma-ip>`                                                            | Spokes can route to each other’s NBMA addresses through the underlay                                           |
|   22 | Test hub-to-spoke NBMA reachability                                                  | Hub                  | `ping <spoke-nbma-ip>`                                                                           | Hub can ping every spoke NBMA address                                                                          |
|   23 | Test spoke-to-hub NBMA reachability                                                  | Spokes               | `ping <hub-nbma-ip>`                                                                             | Every spoke can ping the hub NBMA address                                                                      |
|   24 | Test spoke-to-spoke NBMA reachability                                                | Spokes               | `ping <other-spoke-nbma-ip>`                                                                     | Spokes can reach each other through the underlay                                                               |
|   25 | Trace failed NBMA paths before configuring DMVPN                                     | Any Failed Node      | `traceroute <remote-nbma-ip>`                                                                    | Failure is isolated to addressing, gateway, routing, or interface state                                        |
|   26 | Stop before creating tunnels if any NBMA test fails                                  | Hub, Spokes          | Validation checkpoint                                                                            | No tunnel, NHRP, routing overlay, IPsec, multicast, IPv6, or QoS config is added until NBMA reachability works |
# DMVPN_Underlay_NBMA_Reachability_Skeleton
```
! Transit switch acting as Internet/provider cloud

conf t
ip routing
!
interface <hub-facing-port>
 no switchport
 ip address <hub-gateway-ip> <mask>
 no shutdown
!
interface <spoke1-facing-port>
 no switchport
 ip address <spoke1-gateway-ip> <mask>
 no shutdown
!
interface <spoke2-facing-port>
 no switchport
 ip address <spoke2-gateway-ip> <mask>
 no shutdown
end
! Hub
conf t
interface <hub-underlay-interface>
 ip address <hub-nbma-ip> <mask>
 no shutdown
exit
ip route 0.0.0.0 0.0.0.0 <hub-transit-gateway-ip>
end
! Spoke
conf t
interface <spoke-underlay-interface>
 ip address <spoke-nbma-ip> <mask>
 no shutdown
exit
ip route 0.0.0.0 0.0.0.0 <spoke-transit-gateway-ip>
end
! Required before DMVPN
ping <remote-nbma-ip>
traceroute <remote-nbma-ip>
show ip route <remote-nbma-ip>

```

# DMVPN_Underlay_NBMA_Reachability_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| Interface state | Hub, Spokes, Transit | `show ip interface brief` | All transport interfaces show `up/up` |
| Hub default route | Hub | `show ip route 0.0.0.0` | Static default points to hub transit gateway |
| Spoke default route | Spokes | `show ip route 0.0.0.0` | Static default points to spoke transit gateway |
| Specific NBMA route | Hub, Spokes | `show ip route <remote-nbma-ip>` | Route resolves through the underlay, not a tunnel |
| Hub to spoke NBMA ping | Hub | `ping <spoke-nbma-ip>` | ICMP succeeds |
| Spoke to hub NBMA ping | Spokes | `ping <hub-nbma-ip>` | ICMP succeeds |
| Spoke to spoke NBMA ping | Spokes | `ping <other-spoke-nbma-ip>` | ICMP succeeds |
| Failed path isolation | Any Router | `traceroute <remote-nbma-ip>` | Path shows where underlay reachability breaks |
| No tunnel dependency | Hub, Spokes | `show ip interface brief | include Tunnel` | Tunnel does not need to exist for NBMA reachability to work |
# DMVPN_Underlay_NBMA_Reachability_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Remove hub default route | Hub | `no ip route 0.0.0.0 0.0.0.0 <hub-transit-gateway-ip>` | Hub no longer uses that transport default route |
| 2 | Remove spoke default route | Spokes | `no ip route 0.0.0.0 0.0.0.0 <spoke-transit-gateway-ip>` | Spoke no longer uses that transport default route |
| 3 | Remove hub NBMA address | Hub | `interface <hub-underlay-interface>`<br>`no ip address` | Hub transport address is removed |
| 4 | Remove spoke NBMA address | Spokes | `interface <spoke-underlay-interface>`<br>`no ip address` | Spoke transport address is removed |
| 5 | Shut down transport interface if resetting the lab | Hub, Spokes | `interface <underlay-interface>`<br>`shutdown` | Transport interface is administratively down |
| 6 | Remove transit interface addressing | Transit SW | `interface <transport-port>`<br>`no ip address` | Transit routed port address is removed |
| 7 | Revert transit routed port if needed | Transit SW | `interface <transport-port>`<br>`switchport` | Port returns to Layer 2 mode |
| 8 | Disable transit routing only if fully reverting the transit switch | Transit SW | `no ip routing` | Transit switch stops routing between transport subnets |
# DMVPN_Underlay_NBMA_Reachability_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Router cannot ping local gateway | Interface down, wrong IP, wrong mask, or wrong gateway | `show ip interface brief`<br>`show running-config interface <interface>` | Correct IP/mask and issue `no shutdown` |
| Hub cannot ping spoke NBMA | Missing hub route, wrong transit gateway, or transit cloud not routing | `show ip route <spoke-nbma-ip>` | Add correct static default or specific route |
| Spoke cannot ping hub NBMA | Missing spoke route or wrong default gateway | `show ip route <hub-nbma-ip>` | Add correct `ip route 0.0.0.0 0.0.0.0 <gateway>` |
| Spoke-to-spoke NBMA fails | Transit device is not routing between NBMA subnets | `show running-config | include ip routing` on transit switch | Configure `ip routing` on the transit Layer 3 switch |
| Same-subnet pings work but remote-subnet pings fail | Transit ports are Layer 2 or lack routed IPs | `show running-config interface <port>` | Configure `no switchport`, IP address, and `no shutdown` |
| Route points through the tunnel | Overlay route is being used for transport reachability | `show ip route <remote-nbma-ip>` | Add specific underlay route to remote NBMA through transport gateway |
| Tunnel later comes up with wrong NBMA mapping | Wrong tunnel source or wrong underlay interface selected | `show running-config interface Tunnel<id>`<br>`show ip nhrp` | Correct the tunnel source after underlay is verified |
| NHRP fails after tunnel config | Underlay was never clean | `ping <remote-nbma-ip>` | Fix NBMA reachability before troubleshooting NHRP |
| Routing protocol fails later | Multicast/NHRP problem, but only after underlay is proven | `show dmvpn`<br>`show ip nhrp` | Confirm underlay first, then troubleshoot mGRE/NHRP/routing overlay |


##### Source_Basis
# DMVPN_Underlay_NBMA_Reachability_Mental_Model
# DMVPN_Underlay_NBMA_Reachability_Configuration_Checklist
# DMVPN_Underlay_NBMA_Reachability_Skeleton
# DMVPN_Underlay_NBMA_Reachability_Verification_Commands
# DMVPN_Underlay_NBMA_Reachability_Rollback
# DMVPN_Underlay_NBMA_Reachability_Failure_Checks
