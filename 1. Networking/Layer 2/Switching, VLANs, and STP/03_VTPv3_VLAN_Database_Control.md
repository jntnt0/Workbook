VTPv3_VLAN_Database_Control.md
# VTPv3_VLAN_Database_Control

# VTPv3_VLAN_Database_Control_Index

- VTPv3_VLAN_Database_Control_Mental_Model
- VTPv3_VLAN_Database_Control_Configuration_Checklist
- VTPv3_VLAN_Database_Control_Skeleton
- VTPv3_VLAN_Database_Control_Verification_Commands
- VTPv3_VLAN_Database_Control_Rollback
- VTPv3_VLAN_Database_Control_Failure_Checks

# VTPv3_VLAN_Database_Control_Mental_Model

| Concept | Operational Meaning |
|---|---|
| VTP domain | Switches only exchange VLAN database information when they share the same VTP domain |
| VTPv3 | Version 3 supports normal and extended VLANs and adds safer control with a primary server requirement |
| Primary server | In VTPv3, VLAN creation, deletion, and modification should be performed from the primary server |
| Server mode | A server participates in VTP and can become the primary server for VLAN changes |
| Client mode | A client receives VLAN database updates from the VTP domain and cannot locally create, delete, or modify VLANs |
| Transparent mode | A transparent switch forwards VTP advertisements but does not use them to overwrite its local VLAN database |
| Off mode | A switch in off mode does not participate in VTP and does not forward VTP advertisements |
| Revision number | The revision number increments when the VLAN database changes and is used to determine newer VLAN information |
| VTP password | A shared password helps prevent unauthorized switches from joining and modifying the VTP domain |
| Trunk dependency | VTP advertisements are carried across trunk links, so broken trunking breaks VLAN propagation |
| VLAN database control | VTP controls VLAN existence and naming, not host IP addressing, routing, or trunk allowed VLAN forwarding |
| Pruning | VTP pruning can reduce unnecessary flooded traffic, but it must be enabled intentionally and verified carefully |

# VTPv3_VLAN_Database_Control_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm interswitch links are physically up | SW1/SW2/SW3 | `show interfaces status` | Uplink interfaces show `connected` |
| 2 | Confirm interswitch links are trunking before relying on VTP | SW1/SW2/SW3 | `show interfaces trunk` | Trunk ports show `trunking` |
| 3 | Confirm the current VTP state before making changes | SW1/SW2/SW3 | `show vtp status` | Current VTP version, domain, mode, and revision are known |
| 4 | Check the existing VLAN database before enabling VTP control | SW1/SW2/SW3 | `show vlan brief` | Existing VLANs are documented before VTP changes |
| 5 | Enter global configuration mode on the intended server | SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 6 | Set VTP version 3 on the server | SW1 | `vtp version 3` | Switch runs VTP version 3 |
| 7 | Set the VTP domain name on the server | SW1 | `vtp domain CAMPUS` | Server joins or creates the `CAMPUS` VTP domain |
| 8 | Set the server role | SW1 | `vtp mode server` | Switch is a VTP server for the VLAN feature |
| 9 | Configure the VTP password | SW1 | `vtp password VTP_SECRET` | Server uses the shared VTP password |
| 10 | Exit to privileged EXEC mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 11 | Make SW1 the VTPv3 primary server | SW1 | `vtp primary` | SW1 becomes the primary server for VLAN changes |
| 12 | Enter global configuration mode on the first client | SW2 | `configure terminal` | Prompt changes to global configuration mode |
| 13 | Set VTP version 3 on the client | SW2 | `vtp version 3` | Client runs VTP version 3 |
| 14 | Set the same VTP domain on the client | SW2 | `vtp domain CAMPUS` | Client joins the `CAMPUS` VTP domain |
| 15 | Set the client role | SW2 | `vtp mode client` | SW2 receives VLAN database changes from the domain |
| 16 | Configure the same VTP password on the client | SW2 | `vtp password VTP_SECRET` | Client can authenticate VTP updates |
| 17 | Exit configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 18 | Enter global configuration mode on the second client | SW3 | `configure terminal` | Prompt changes to global configuration mode |
| 19 | Set VTP version 3 on the second client | SW3 | `vtp version 3` | Client runs VTP version 3 |
| 20 | Set the same VTP domain on the second client | SW3 | `vtp domain CAMPUS` | Client joins the `CAMPUS` VTP domain |
| 21 | Set the client role | SW3 | `vtp mode client` | SW3 receives VLAN database changes from the domain |
| 22 | Configure the same VTP password on the second client | SW3 | `vtp password VTP_SECRET` | Client can authenticate VTP updates |
| 23 | Exit configuration mode | SW3 | `end` | Prompt returns to privileged EXEC mode |
| 24 | Verify VTPv3 status on the primary server | SW1 | `show vtp status` | Version is 3, domain is `CAMPUS`, mode is primary server, and revision is visible |
| 25 | Verify VTPv3 status on clients | SW2/SW3 | `show vtp status` | Version is 3, domain is `CAMPUS`, mode is client, and revision is visible |
| 26 | Create VLAN 10 from the primary server | SW1 | `configure terminal` then `vlan 10` then `name USERS` | VLAN 10 is created on the primary server |
| 27 | Create VLAN 20 from the primary server | SW1 | `vlan 20` then `name VOICE` | VLAN 20 is created on the primary server |
| 28 | Create VLAN 30 from the primary server | SW1 | `vlan 30` then `name GUEST` | VLAN 30 is created on the primary server |
| 29 | Exit VLAN configuration mode | SW1 | `end` | VLAN changes are committed and VTP revision increments |
| 30 | Verify the server revision incremented | SW1 | `show vtp status | include version running|Operating|Domain|Revision|Primary` | SW1 shows VTPv3 primary server state and an increased revision number |
| 31 | Verify client revision matches the server | SW2/SW3 | `show vtp status | include version running|Operating|Domain|Revision` | Client revision matches the server revision after update propagation |
| 32 | Verify VLANs propagated to clients | SW2/SW3 | `show vlan brief` | VLANs 10, 20, and 30 appear on the clients |
| 33 | Confirm clients cannot locally modify the VLAN database | SW2 | `configure terminal` then `vlan 40` | Client should reject local VLAN creation in VTP client mode |
| 34 | Verify trunks still carry the required VLANs | SW1/SW2/SW3 | `show interfaces trunk` | Required VLANs are allowed and active on trunk links |
| 35 | Optionally enable VTP pruning from the server | SW1 | `configure terminal` then `vtp pruning` | VTP pruning is enabled for the VTP domain |
| 36 | Verify pruning state | SW1/SW2/SW3 | `show vtp status | include Pruning` | Pruning mode reflects the intended state |
| 37 | Save configuration on all switches | SW1/SW2/SW3 | `write memory` | VTP, trunk, and VLAN database settings survive reload |

# VTPv3_VLAN_Database_Control_Skeleton

Primary VTPv3 server:

configure terminal
!
vtp version 3
vtp domain CAMPUS
vtp mode server
vtp password VTP_SECRET
!
end
vtp primary
!
configure terminal
vlan 10
 name USERS
vlan 20
 name VOICE
vlan 30
 name GUEST
end
write memory

VTPv3 client switch:

configure terminal
!
vtp version 3
vtp domain CAMPUS
vtp mode client
vtp password VTP_SECRET
!
end
write memory

VTPv3 transparent switch:

configure terminal
!
vtp version 3
vtp domain CAMPUS
vtp mode transparent
vtp password VTP_SECRET
!
end
write memory

VTP off mode switch:

configure terminal
!
vtp mode off
!
end
write memory

Optional VTP pruning from the server:

configure terminal
!
vtp pruning
!
end
write memory

# VTPv3_VLAN_Database_Control_Verification_Commands

| Verification Goal | Device | Command | Expected Result |
|---|---|---|---|
| Confirm VTP version, domain, mode, and revision | SW1/SW2/SW3 | `show vtp status` | Version, domain, mode, and revision match design |
| Confirm primary server state | SW1 | `show vtp status | include Operating|Primary|Revision` | SW1 shows primary server state for VLAN control |
| Confirm clients are clients | SW2/SW3 | `show vtp status | include Operating|Revision` | SW2 and SW3 show client mode and matching revision |
| Confirm VTP domain name | SW1/SW2/SW3 | `show vtp status | include Domain` | All switches show the same VTP domain |
| Confirm VTP version 3 is running | SW1/SW2/SW3 | `show vtp status | include version running` | All switches show VTP version 3 |
| Confirm revision synchronization | SW1/SW2/SW3 | `show vtp status | include Revision` | Revision numbers match after VLAN updates propagate |
| Confirm VLAN propagation | SW2/SW3 | `show vlan brief` | VLANs created on the primary server appear on clients |
| Confirm VLAN names propagated | SW2/SW3 | `show vlan brief` | VLAN names match the primary server |
| Confirm trunks exist for VTP transport | SW1/SW2/SW3 | `show interfaces trunk` | Interswitch links are trunking |
| Confirm required VLANs are active on trunks | SW1/SW2/SW3 | `show interfaces trunk` | Propagated VLANs are allowed and active |
| Confirm native VLAN consistency | SW1/SW2/SW3 | `show interfaces trunk` | Native VLANs match across trunk links |
| Confirm local VTP configuration | SW1/SW2/SW3 | `show running-config | include ^vtp` | VTP version, domain, mode, and password configuration are present |
| Confirm pruning state | SW1/SW2/SW3 | `show vtp status | include Pruning` | Pruning shows the intended state |
| Confirm client cannot create VLANs | SW2/SW3 | `configure terminal` then `vlan 999` | VLAN creation is rejected in client mode |
| Confirm transparent switch local behavior | SW-TRANSPARENT | `show vtp status` and `show vlan brief` | Transparent switch forwards VTP but keeps local VLAN control |
| Confirm no native VLAN mismatch messages | SW1/SW2/SW3 | `show logging | include NATIVE|native|VTP` | No unexpected VTP or native VLAN mismatch messages appear |

# VTPv3_VLAN_Database_Control_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Document current VTP state before rollback | SW1/SW2/SW3 | `show vtp status` | Current version, domain, mode, and revision are recorded |
| 2 | Document current VLAN database before rollback | SW1/SW2/SW3 | `show vlan brief` | VLAN state is recorded |
| 3 | Enter configuration mode on all switches | SW1/SW2/SW3 | `configure terminal` | Prompt changes to global configuration mode |
| 4 | Remove active VTP participation | SW1/SW2/SW3 | `vtp mode transparent` | Switches stop using VTP to overwrite local VLAN database |
| 5 | Remove VTP password if decommissioning the domain | SW1/SW2/SW3 | `no vtp password` | VTP password is removed |
| 6 | Optionally move switches fully out of VTP advertisement forwarding | SW1/SW2/SW3 | `vtp mode off` | Switches do not participate in or forward VTP advertisements |
| 7 | Remove test VLAN 10 if no longer needed | SW1/SW2/SW3 | `no vlan 10` | VLAN 10 is removed locally |
| 8 | Remove test VLAN 20 if no longer needed | SW1/SW2/SW3 | `no vlan 20` | VLAN 20 is removed locally |
| 9 | Remove test VLAN 30 if no longer needed | SW1/SW2/SW3 | `no vlan 30` | VLAN 30 is removed locally |
| 10 | Disable pruning if it was enabled only for the test | SW1 | `no vtp pruning` | VTP pruning is disabled |
| 11 | Exit configuration mode | SW1/SW2/SW3 | `end` | Prompt returns to privileged EXEC mode |
| 12 | Save rollback | SW1/SW2/SW3 | `write memory` | Startup configuration reflects rollback |
| 13 | Verify rollback VTP mode | SW1/SW2/SW3 | `show vtp status` | Switches show transparent or off mode as intended |
| 14 | Verify VLAN cleanup | SW1/SW2/SW3 | `show vlan brief` | Test VLANs are removed unless intentionally retained |
| 15 | Verify trunks still operate after rollback | SW1/SW2/SW3 | `show interfaces trunk` | Trunking remains healthy if still required |

# VTPv3_VLAN_Database_Control_Failure_Checks

| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| VLANs do not propagate to clients | Trunk between switches is down or not trunking | SW1/SW2/SW3 | `show interfaces trunk` | Fix trunk mode, cabling, allowed VLANs, or shutdown state |
| Client does not join the VTP domain | Domain name mismatch | SW1/SW2/SW3 | `show vtp status | include Domain` | Configure the same `vtp domain <name>` on all participating switches |
| Client ignores updates | VTP password mismatch | SW1/SW2/SW3 | `show running-config | include ^vtp` | Configure the same `vtp password <password>` on all participating switches |
| Client has wrong revision | Client has not received the latest advertisements | SW2/SW3 | `show vtp status | include Revision` | Verify trunks, domain, password, and VTP version |
| Server cannot create VLAN changes in VTPv3 | Server is not primary | SW1 | `show vtp status | include Operating|Primary` | Run `vtp primary` from the intended server |
| VLAN creation fails on client | Expected VTP client behavior | SW2/SW3 | `show vtp status | include Operating` | Make VLAN changes on the VTPv3 primary server |
| VLANs exist on server but not clients | Clients are in transparent or off mode | SW2/SW3 | `show vtp status | include Operating` | Change clients to `vtp mode client` if they should learn VLANs |
| Transparent switch does not update its VLAN database | Expected transparent behavior | SW-TRANSPARENT | `show vtp status` | Use client mode if the switch should learn VLANs from VTP |
| Transparent switch does not pass updates downstream | Trunking issue or switch is in off mode | SW-TRANSPARENT | `show interfaces trunk` and `show vtp status` | Use transparent mode and fix trunking |
| VTP advertisements stop at a switch | Switch is in VTP off mode | SW-MIDPOINT | `show vtp status` | Use transparent or client mode if advertisements must pass through |
| Unexpected VLAN deletion or change | Wrong switch became primary or wrong domain joined | SW1/SW2/SW3 | `show vtp status` | Isolate the switch, correct domain/password, and restore VLAN database |
| Revision number unexpectedly high | Switch has stale VTP database from another environment | NEW-SW | `show vtp status` | Reset by changing VTP domain, then set correct domain, mode, and password |
| VLAN allowed on trunk but traffic fails | VLAN exists but is not forwarding or not allowed on path | SW1/SW2/SW3 | `show interfaces trunk` | Add VLAN to allowed list and verify STP forwarding |
| VLAN propagated but hosts cannot communicate | VTP only created VLANs, it did not configure access ports | SW1/SW2/SW3 | `show vlan brief` | Assign access ports to the correct VLAN |
| Extended VLANs do not propagate | Switch is not running VTP version 3 | SW1/SW2/SW3 | `show vtp status | include version running` | Configure `vtp version 3` |
| Pruning removes traffic unexpectedly | VTP pruning enabled without validating downstream VLAN use | SW1/SW2/SW3 | `show vtp status | include Pruning` and `show interfaces trunk` | Disable pruning or adjust VLAN placement/design |
| Native VLAN mismatch warnings appear | Trunk native VLANs differ | SW1/SW2/SW3 | `show logging | include NATIVE|native` | Match native VLAN configuration on both trunk ends |
| VTP state changes after reload | Configuration was not saved | SW1/SW2/SW3 | `show startup-config | include ^vtp` | Reapply configuration and run `write memory` |
| Client still missing VLANs after all settings match | No recent VLAN change has triggered update or link recently recovered | SW1/SW2/SW3 | `show vtp status` and `show interfaces trunk` | Make a controlled VLAN database change on the primary server or bounce the lab trunk if needed |
