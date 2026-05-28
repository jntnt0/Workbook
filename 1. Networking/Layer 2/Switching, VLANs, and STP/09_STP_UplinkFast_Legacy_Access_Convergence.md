STP_UplinkFast_Legacy_Access_Convergence.md
# STP_UplinkFast_Legacy_Access_Convergence

# STP_UplinkFast_Legacy_Access_Convergence_Index

- STP_UplinkFast_Legacy_Access_Convergence_Mental_Model
- STP_UplinkFast_Legacy_Access_Convergence_Configuration_Checklist
- STP_UplinkFast_Legacy_Access_Convergence_Skeleton
- STP_UplinkFast_Legacy_Access_Convergence_Verification_Commands
- STP_UplinkFast_Legacy_Access_Convergence_Rollback
- STP_UplinkFast_Legacy_Access_Convergence_Failure_Checks

# STP_UplinkFast_Legacy_Access_Convergence_Mental_Model

| Concept | Operational Meaning |
|---|---|
| UplinkFast | Cisco-proprietary legacy STP feature that speeds recovery when an access switch loses its active root port |
| Legacy access-layer feature | UplinkFast belongs on access switches, not root bridges, distribution roots, or core switches |
| Direct failure only | UplinkFast helps when the local access switch directly detects its root port going down |
| Uplink group | The access switch tracks the current root port and blocked alternate uplinks toward the root bridge |
| Root port | The active forwarding uplink toward the STP root bridge |
| Alternate uplink | A blocked backup path toward the root bridge that can move to forwarding if the root port fails |
| Fast failover | When the root port fails, UplinkFast moves the best alternate uplink into forwarding quickly |
| Classic STP problem | Without UplinkFast, the blocked backup uplink normally waits through listening and learning before forwarding |
| CAM update behavior | UplinkFast sends dummy multicast frames so upstream switches relearn MAC locations after the new uplink forwards |
| Global configuration | `spanning-tree uplinkfast` is configured globally and affects the whole switch |
| Bridge priority side effect | Enabling UplinkFast raises the local bridge priority so the access switch is unlikely to become root |
| Port cost side effect | Enabling UplinkFast increases port costs so the access switch is less likely to become a transit/designated switch |
| Not a root feature | If a switch should ever be STP root, do not enable UplinkFast on it |
| Not BackboneFast | UplinkFast handles directly connected uplink failure; BackboneFast handles indirect failure |
| Not Rapid PVST replacement | Rapid PVST already has rapid convergence behavior, so UplinkFast is mainly for legacy 802.1D/PVST labs |
| Verification rule | Verify the switch has one root port and at least one blocked alternate before testing UplinkFast |

# STP_UplinkFast_Legacy_Access_Convergence_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the switch is an access-layer switch, not the intended root | SW2 | `show spanning-tree root` | SW2 is not root for the target VLAN |
| 2 | Confirm the current STP mode before using a legacy feature | SW1/SW2/SW3 | `show spanning-tree summary` | STP mode is known before changes |
| 3 | Confirm the target VLAN exists | SW1/SW2/SW3 | `show vlan brief` | VLAN 1 or the target VLAN exists and is active |
| 4 | Confirm interswitch ports are connected | SW1/SW2/SW3 | `show interfaces status` | Uplink interfaces show `connected` |
| 5 | Confirm trunks are operating if testing across VLAN trunks | SW1/SW2/SW3 | `show interfaces trunk` | Interswitch links show `trunking` |
| 6 | Confirm root placement before enabling UplinkFast | SW1/SW2/SW3 | `show spanning-tree vlan 1` | Root bridge is a distribution or upstream switch |
| 7 | Confirm SW2 has a root port before testing | SW2 | `show spanning-tree vlan 1` | One uplink shows `Root FWD` |
| 8 | Confirm SW2 has a blocked alternate uplink before testing | SW2 | `show spanning-tree vlan 1` | One backup uplink shows `Altn BLK` or equivalent blocked state |
| 9 | Confirm UplinkFast current state | SW2 | `show spanning-tree summary | include UplinkFast` | UplinkFast state is visible as enabled or disabled |
| 10 | Record bridge priority before enabling UplinkFast | SW2 | `show spanning-tree vlan 1 | include Bridge ID|priority` | Current bridge priority is documented |
| 11 | Record interface costs before enabling UplinkFast | SW2 | `show spanning-tree vlan 1 | begin Interface` | Current STP interface costs and roles are documented |
| 12 | Enter global configuration mode on the access switch | SW2 | `configure terminal` | Prompt changes to global configuration mode |
| 13 | Enable UplinkFast globally | SW2 | `spanning-tree uplinkfast` | UplinkFast is enabled for the switch |
| 14 | Exit configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 15 | Verify UplinkFast is enabled | SW2 | `show spanning-tree summary | include UplinkFast` | Output shows UplinkFast is enabled |
| 16 | Verify bridge priority was raised by UplinkFast | SW2 | `show spanning-tree vlan 1 | include Bridge ID|priority` | SW2 bridge priority is raised so it is unlikely to become root |
| 17 | Verify port costs were raised by UplinkFast | SW2 | `show spanning-tree vlan 1 | begin Interface` | Interface costs show the UplinkFast cost increase |
| 18 | Verify the active root port and blocked alternate before failure | SW2 | `show spanning-tree vlan 1 | include Fa|Gi|Root|Altn|BLK|FWD` | One uplink is root forwarding and another is blocked alternate |
| 19 | Start continuous traffic before failover test | R1/H1 | `ping <remote-host-ip> repeat 1000000` | Traffic starts so packet loss can be observed |
| 20 | Optionally enable live event observation in a lab | SW2 | `terminal monitor` | Terminal can display live log messages |
| 21 | Optionally enable STP event debugging in a lab | SW2 | `debug spanning-tree events` | STP state changes display live |
| 22 | Enter configuration mode to fail the active root port | SW2 | `configure terminal` | Prompt changes to global configuration mode |
| 23 | Select the active root port | SW2 | `interface FastEthernet0/23` | Prompt changes to interface configuration mode |
| 24 | Shut the active root port to trigger UplinkFast | SW2 | `shutdown` | Active root port goes down |
| 25 | Exit configuration mode quickly | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 26 | Verify the alternate uplink moved to forwarding | SW2 | `show spanning-tree vlan 1 | include Fa|Gi|Root|Altn|BLK|FWD` | Former blocked alternate becomes the root forwarding port |
| 27 | Verify UplinkFast event message if logging is available | SW2 | `show logging | include UPLINK|SPANTREE_FAST|PORT_FWD` | Log shows the alternate port moved to forwarding by UplinkFast |
| 28 | Verify MAC table relearning after failover | SW1/SW2/SW3 | `show mac address-table dynamic` | MAC addresses are learned on the new forwarding path |
| 29 | Verify host traffic recovered | R1/H1 | Review ping output | Packet loss is brief compared to classic STP convergence |
| 30 | Restore the original root port | SW2 | `configure terminal` then `interface FastEthernet0/23` then `no shutdown` | Original uplink comes back up |
| 31 | Exit configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 32 | Observe that restoration is not instant | SW2 | `show spanning-tree vlan 1 | include Fa|Gi|Root|Altn|BLK|FWD` | Restored link transitions normally before returning to its role |
| 33 | Confirm final stable STP state | SW2 | `show spanning-tree vlan 1` | Root port and alternate port settle into expected roles |
| 34 | Disable debugging after the test | SW2 | `undebug all` | Debug output stops |
| 35 | Disable terminal monitoring if enabled | SW2 | `terminal no monitor` | Live terminal log display stops |
| 36 | Save configuration if UplinkFast is intentionally retained | SW2 | `write memory` | UplinkFast configuration survives reload |

# STP_UplinkFast_Legacy_Access_Convergence_Skeleton

Access switch with UplinkFast:

configure terminal
!
spanning-tree uplinkfast
!
end
write memory

Optional legacy PVST lab baseline:

configure terminal
!
spanning-tree mode pvst
!
vlan 1
!
end
write memory

Baseline before enabling UplinkFast:

show spanning-tree summary
show spanning-tree root
show spanning-tree vlan 1
show spanning-tree vlan 1 | begin Interface
show spanning-tree vlan 1 | include Bridge ID|priority
show spanning-tree summary | include UplinkFast

Lab-only event observation:

terminal monitor
debug spanning-tree events

Fail active root port:

configure terminal
interface FastEthernet0/23
 shutdown
end

Verify failover:

show spanning-tree vlan 1
show spanning-tree vlan 1 | include Fa|Gi|Root|Altn|BLK|FWD
show logging | include UPLINK|SPANTREE_FAST|PORT_FWD
show mac address-table dynamic

Restore active uplink:

configure terminal
interface FastEthernet0/23
 no shutdown
end

Stop debug:

undebug all
terminal no monitor

Optional command with update rate:

configure terminal
!
spanning-tree uplinkfast max-update-rate 150
!
end
write memory

# STP_UplinkFast_Legacy_Access_Convergence_Verification_Commands

| Verification Goal | Device | Command | Expected Result |
|---|---|---|---|
| Confirm STP mode | SW1/SW2/SW3 | `show spanning-tree summary` | STP mode is known and appropriate for legacy UplinkFast testing |
| Confirm UplinkFast status | SW2 | `show spanning-tree summary | include UplinkFast` | UplinkFast shows enabled |
| Confirm SW2 is not root | SW2 | `show spanning-tree root` | SW2 is not the root bridge |
| Confirm root bridge for VLAN 1 | SW1/SW2/SW3 | `show spanning-tree vlan 1` | Intended upstream switch is root |
| Confirm root port on access switch | SW2 | `show spanning-tree vlan 1` | One uplink shows `Root FWD` |
| Confirm blocked alternate uplink | SW2 | `show spanning-tree vlan 1` | Backup uplink shows `Altn BLK` or blocked state |
| Confirm UplinkFast bridge-priority change | SW2 | `show spanning-tree vlan 1 | include Bridge ID|priority` | Bridge priority is raised after UplinkFast is enabled |
| Confirm UplinkFast cost change | SW2 | `show spanning-tree vlan 1 | begin Interface` | Port costs are increased after UplinkFast is enabled |
| Confirm live failover event | SW2 | `show logging | include UPLINK|SPANTREE_FAST|PORT_FWD` | Log shows alternate uplink moved to forwarding by UplinkFast |
| Confirm new root port after failure | SW2 | `show spanning-tree vlan 1` | Former alternate uplink becomes root forwarding |
| Confirm MAC table after failover | SW1/SW2/SW3 | `show mac address-table dynamic` | MAC addresses relearn on the new path |
| Confirm traffic recovery | R1/H1 | `ping <remote-host-ip> repeat 100` | Ping succeeds after brief failover loss |
| Confirm no inconsistent ports | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No ports are listed |
| Confirm no err-disabled ports | SW1/SW2/SW3 | `show interfaces status err-disabled` | Required uplinks are not err-disabled |
| Confirm no loop or MAC flap messages | SW1/SW2/SW3 | `show logging | include MACFLAP|LOOP|SPANTREE` | No active loop symptoms appear |
| Confirm restored link state | SW2 | `show interfaces FastEthernet0/23 status` | Restored link is connected after `no shutdown` |
| Confirm final STP stability | SW2 | `show spanning-tree vlan 1` | Root and alternate roles return to expected state |
| Confirm debugging is disabled | SW2 | `show debugging` | No STP debug remains enabled |
| Confirm saved configuration | SW2 | `show startup-config | include spanning-tree uplinkfast` | Startup configuration contains UplinkFast if intentionally retained |

# STP_UplinkFast_Legacy_Access_Convergence_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Stop debugging before rollback | SW2 | `undebug all` | Debug output stops |
| 2 | Disable terminal monitoring if enabled | SW2 | `terminal no monitor` | Terminal log display stops |
| 3 | Restore any test-shutdown uplink | SW2 | `configure terminal` then `interface FastEthernet0/23` then `no shutdown` | Original uplink is administratively up |
| 4 | Exit interface configuration | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 5 | Verify restored uplink state | SW2 | `show interfaces FastEthernet0/23 status` | Interface shows connected if the far end is up |
| 6 | Enter global configuration mode | SW2 | `configure terminal` | Prompt changes to global configuration mode |
| 7 | Disable UplinkFast | SW2 | `no spanning-tree uplinkfast` | UplinkFast is disabled globally |
| 8 | Exit configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 9 | Verify UplinkFast is disabled | SW2 | `show spanning-tree summary | include UplinkFast` | Output shows UplinkFast is disabled |
| 10 | Verify bridge priority returned or was manually corrected | SW2 | `show spanning-tree vlan 1 | include Bridge ID|priority` | Bridge priority matches intended design |
| 11 | Verify interface costs returned or were manually corrected | SW2 | `show spanning-tree vlan 1 | begin Interface` | Interface costs match intended design |
| 12 | If needed, manually reset bridge priority | SW2 | `configure terminal` then `spanning-tree vlan 1 priority 32768` | VLAN bridge priority returns to default base priority |
| 13 | If needed, manually reset interface cost on first uplink | SW2 | `interface FastEthernet0/19` then `no spanning-tree vlan 1 cost` | STP cost returns to default |
| 14 | If needed, manually reset interface cost on second uplink | SW2 | `interface FastEthernet0/23` then `no spanning-tree vlan 1 cost` | STP cost returns to default |
| 15 | Exit configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 16 | Verify final STP topology after rollback | SW2 | `show spanning-tree vlan 1` | Port roles and states reflect normal STP behavior |
| 17 | Save rollback | SW2 | `write memory` | Startup configuration reflects rollback |

# STP_UplinkFast_Legacy_Access_Convergence_Failure_Checks

| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| UplinkFast does nothing during test | Switch has no blocked alternate uplink | SW2 | `show spanning-tree vlan 1` | Add or restore a redundant uplink that STP blocks before testing |
| UplinkFast does nothing during test | Wrong port was shut down | SW2 | `show spanning-tree vlan 1` | Identify and shut the current root port, not a designated or access port |
| UplinkFast is enabled on the root bridge | Feature is on the wrong switch | SW1/SW2/SW3 | `show spanning-tree root` | Remove UplinkFast from root or distribution switches |
| Access switch becomes root unexpectedly | Root priority design is wrong or UplinkFast not applied where expected | SW1/SW2/SW3 | `show spanning-tree vlan 1` | Configure intentional root bridge placement upstream |
| Alternate port stays blocking after root port failure | VLAN is not allowed on backup trunk or backup link is down | SW2 | `show interfaces trunk` | Allow the VLAN and restore the backup trunk |
| Alternate port is missing from STP output | VLAN missing on the backup path | SW1/SW2/SW3 | `show vlan brief` | Create the VLAN on all participating switches |
| Failover still takes about 30 seconds | UplinkFast is disabled or not supported in current STP mode/platform | SW2 | `show spanning-tree summary | include UplinkFast` | Enable UplinkFast for legacy PVST, or use Rapid PVST for modern convergence |
| Logs do not show UplinkFast event | Logging/debug not enabled or event aged out | SW2 | `show logging | include SPANTREE_FAST|UPLINK` | Use `terminal monitor` and `debug spanning-tree events` in a lab |
| MAC table is wrong after failover | CAM tables have not relearned yet | SW1/SW2/SW3 | `show mac address-table dynamic` | Generate traffic and verify MACs relearn on the new path |
| Ping loss is longer than expected | Backup uplink was not in the uplink group or forwarding path is broken | SW2 | `show spanning-tree vlan 1` | Verify root port, alternate port, and VLAN trunking before testing |
| Restored primary uplink does not forward immediately | Expected behavior after restoration | SW2 | `show spanning-tree vlan 1` | Wait for normal STP transition; UplinkFast speeds failure cutover, not restoration |
| Port is err-disabled | Protection feature or link issue disabled the port | SW2 | `show interfaces status err-disabled` | Fix the trigger, then shut/no shut the interface |
| MAC flapping appears | Layer 2 loop or unsafe redundant forwarding | SW1/SW2/SW3 | `show logging | include MACFLAP|LOOP|SPANTREE` | Verify one blocked redundant path exists |
| UplinkFast changed topology unexpectedly | Bridge priority and port costs were changed by the feature | SW2 | `show spanning-tree vlan 1 | include priority` and `show spanning-tree vlan 1 | begin Interface` | Use UplinkFast only on access switches, or remove it and tune STP manually |
| Distribution switch no longer becomes designated as expected | UplinkFast port-cost changes altered STP decisions | SW2 | `show spanning-tree vlan 1` | Remove UplinkFast from non-access switches |
| Rapid PVST lab does not need UplinkFast | RSTP already includes rapid alternate-port behavior | SW1/SW2/SW3 | `show spanning-tree summary` | Use `spanning-tree mode rapid-pvst` instead of legacy UplinkFast design |
| VLAN traffic fails after failover | Trunk allowed list, STP state, or access VLAN is wrong | SW1/SW2/SW3 | `show interfaces trunk` and `show spanning-tree vlan 1` | Fix VLAN allowance, STP state, or access-port VLAN assignment |
| Debug output continues after lab | Debug was not disabled | SW2 | `show debugging` | Run `undebug all` |
| Temporary shutdown is saved | Config was saved while the test uplink was shut | SW2 | `show startup-config interface FastEthernet0/23` | Remove `shutdown`, restore interface, and save again |
| UplinkFast config disappears after reload | Configuration was not saved | SW2 | `show startup-config | include spanning-tree uplinkfast` | Reapply `spanning-tree uplinkfast` and run `write memory` |
