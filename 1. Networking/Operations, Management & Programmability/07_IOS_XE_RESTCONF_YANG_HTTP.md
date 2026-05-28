IOS_XE_RESTCONF_YANG_HTTP.md

IOS_XE_RESTCONF_YANG_HTTP

# IOS_XE_RESTCONF_YANG_HTTP_Mental_Model
| Concept | Operational Meaning |
|---|---|
| RESTCONF | HTTP-based management protocol for reading and changing YANG-modeled data |
| YANG | The schema/tree that defines the data exposed through RESTCONF |
| RESTCONF is not NETCONF replacement | RESTCONF and NETCONF use the same YANG model foundation, but expose it through different protocol workflows |
| Transport | RESTCONF commonly uses HTTPS on TCP/443 |
| Encoding | RESTCONF can use JSON or XML |
| Resource root | IOS XE RESTCONF calls usually begin with `https://<DEVICE_IP>/restconf/` |
| Data resource | `/restconf/data/` is used for configuration and state data resources |
| Operations resource | `/restconf/operations/` is used for model-specific RPC-style operations when supported |
| HTTP method model | GET reads, POST creates or invokes operations, PUT creates or replaces, PATCH updates, DELETE removes |
| HTTP status model | 2xx means success, 4xx usually means client/request/auth/path issue, 5xx usually means device-side failure |
| Header dependency | `Accept` controls response format. `Content-Type` controls payload format for writes |
| JSON media type | Use `application/yang-data+json` for JSON RESTCONF requests and responses |
| XML media type | Use `application/yang-data+xml` for XML RESTCONF requests and responses |
| Auth dependency | RESTCONF uses HTTP authentication settings, commonly local authentication in CML labs |
| HTTP server dependency | Enabling `restconf` alone is not enough if HTTPS and HTTP authentication are not correctly configured |
| Model path dependency | A correct RESTCONF URL depends on the module name, namespace, container, list key, and leaf path |
| Read before write | Validate GET against the target path before using POST, PUT, PATCH, or DELETE |
| No transaction lock model | RESTCONF is simpler than NETCONF but does not provide the same lock/transaction behavior |
| Related labs | `ios-xe-netconf-restconf-final`, `restconf-r1-bridge-final`, `ios-xe-rest-api-final` |
# IOS_XE_RESTCONF_YANG_HTTP_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the IOS XE device has a reachable management IP | IOS XE | `show ip interface brief` | Management interface or SVI has the expected IP and is up/up |
| 2 | Confirm the IOS XE device has a return route to the RESTCONF client | IOS XE | `show ip route <CLIENT_IP>` | Route exists back toward the RESTCONF client |
| 3 | Confirm client-to-device reachability | RESTCONF client | `ping <DEVICE_IP>` | Client can reach the IOS XE management IP |
| 4 | Enter global configuration mode | IOS XE | `configure terminal` | Device enters global config mode |
| 5 | Configure a stable hostname | IOS XE | `hostname <DEVICE_NAME>` | Prompt changes to the configured hostname |
| 6 | Configure a domain name if HTTPS crypto material or SSH services need it in the lab baseline | IOS XE | `ip domain-name <DOMAIN_NAME>` | Domain name is configured |
| 7 | Create a local privilege user using strong hashing if supported | IOS XE | `username <USER> privilege 15 algorithm-type scrypt secret <SECRET>` | Local privilege 15 user exists with type 9 secret if supported |
| 8 | Use fallback local user syntax if scrypt is unsupported | IOS XE | `username <USER> privilege 15 secret <SECRET>` | Local privilege 15 user exists with a hashed secret |
| 9 | Enable AAA if the lab uses default AAA method lists | IOS XE | `aaa new-model` | AAA method-list configuration becomes available |
| 10 | Set default login authentication to local for lab access | IOS XE | `aaa authentication login default local` | Default login authentication can use the local username database |
| 11 | Set HTTP authentication to local | IOS XE | `ip http authentication local` | HTTP/HTTPS services use local usernames |
| 12 | Enable HTTPS server for RESTCONF transport | IOS XE | `ip http secure-server` | HTTPS server is enabled |
| 13 | Optional: disable cleartext HTTP if the lab does not need it | IOS XE | `no ip http server` | HTTP on TCP/80 is disabled while HTTPS remains available |
| 14 | Optional: set HTTPS port explicitly if needed | IOS XE | `ip http secure-port 443` | HTTPS listens on TCP/443 |
| 15 | Enable RESTCONF | IOS XE | `restconf` | RESTCONF service is enabled |
| 16 | Optional: create a standard ACL for allowed RESTCONF client source | IOS XE | `access-list <ACL_NUM> permit <CLIENT_IP>` | ACL permits the intended RESTCONF client |
| 17 | Optional: deny other RESTCONF client sources | IOS XE | `access-list <ACL_NUM> deny any log` | Other client sources are denied and can increment/log |
| 18 | Optional: apply HTTP server source restriction | IOS XE | `ip http access-class <ACL_NUM>` | HTTP/HTTPS access is restricted to permitted sources |
| 19 | Exit configuration mode | IOS XE | `end` | Device returns to privileged EXEC mode |
| 20 | Save the RESTCONF baseline | IOS XE | `write memory` | Startup-config keeps RESTCONF and HTTPS baseline |
| 21 | Verify YANG management process state | IOS XE | `show platform software yang-management process` | YANG management processes are present and running |
| 22 | Verify RESTCONF and HTTP server config | IOS XE | `show running-config | include ^restconf|^ip http` | RESTCONF, HTTPS, and local HTTP authentication are present |
| 23 | Test RESTCONF root reachability | RESTCONF client | `curl -k -u <USER>:<SECRET> https://<DEVICE_IP>/.well-known/host-meta` | Client receives RESTCONF root discovery response if supported |
| 24 | Test RESTCONF data resource reachability | RESTCONF client | `curl -k -u <USER>:<SECRET> -H "Accept: application/yang-data+json" https://<DEVICE_IP>/restconf/data/` | Client receives JSON response or valid RESTCONF error, not timeout |
| 25 | Test a known model path with GET | RESTCONF client | `curl -k -u <USER>:<SECRET> -H "Accept: application/yang-data+json" https://<DEVICE_IP>/restconf/data/<MODULE>:<PATH>` | Client receives modeled JSON data |
| 26 | Compare RESTCONF result to CLI truth | IOS XE | `show <EQUIVALENT_CLI_COMMAND>` | CLI output matches the model data returned by RESTCONF |
| 27 | Attempt write only after path and payload are validated | RESTCONF client | `POST`, `PUT`, or `PATCH` to `<CONFIG_PATH>` with `Content-Type: application/yang-data+json` | Device returns expected 2xx success code |
| 28 | Verify write effect from CLI | IOS XE | `show running-config | section <FEATURE>` | Running config reflects the intended RESTCONF change |
# IOS_XE_RESTCONF_YANG_HTTP_Skeleton
! =========================
! IOS XE RESTCONF/YANG over HTTPS baseline
! =========================
configure terminal
!
hostname <DEVICE_NAME>
ip domain-name <DOMAIN_NAME>
!
username <USER> privilege 15 algorithm-type scrypt secret <SECRET>
! If unsupported:
! username <USER> privilege 15 secret <SECRET>
!
aaa new-model
aaa authentication login default local
!
ip http authentication local
ip http secure-server
no ip http server
!
restconf
!
end
write memory
! =========================
! Optional HTTP/HTTPS source restriction
! =========================
configure terminal
!
access-list <ACL_NUM> permit <CLIENT_IP>
access-list <ACL_NUM> deny any log
!
ip http access-class <ACL_NUM>
!
end
write memory
! =========================
! IOS XE verification
! =========================
show running-config | include ^restconf|^ip http|^aaa authentication login default|^username
show platform software yang-management process
show access-lists <ACL_NUM>
! =========================
! Client RESTCONF root discovery test
! =========================
curl -k -u <USER>:<SECRET> \
  https://<DEVICE_IP>/.well-known/host-meta
! =========================
! Client RESTCONF data resource test
! =========================
curl -k -u <USER>:<SECRET> \
  -H "Accept: application/yang-data+json" \
  https://<DEVICE_IP>/restconf/data/
! =========================
! Client RESTCONF GET model path pattern
! =========================
curl -k -u <USER>:<SECRET> \
  -H "Accept: application/yang-data+json" \
  https://<DEVICE_IP>/restconf/data/<MODULE>:<PATH>
! =========================
! Client RESTCONF POST pattern
! =========================
curl -k -u <USER>:<SECRET> \
  -X POST \
  -H "Accept: application/yang-data+json" \
  -H "Content-Type: application/yang-data+json" \
  -d '<JSON_PAYLOAD>' \
  https://<DEVICE_IP>/restconf/data/<MODULE>:<PARENT_PATH>
! =========================
! Client RESTCONF PATCH pattern
! =========================
curl -k -u <USER>:<SECRET> \
  -X PATCH \
  -H "Accept: application/yang-data+json" \
  -H "Content-Type: application/yang-data+json" \
  -d '<JSON_PAYLOAD>' \
  https://<DEVICE_IP>/restconf/data/<MODULE>:<CONFIG_PATH>
! =========================
! Client RESTCONF DELETE pattern
! =========================
curl -k -u <USER>:<SECRET> \
  -X DELETE \
  https://<DEVICE_IP>/restconf/data/<MODULE>:<CONFIG_PATH>
# IOS_XE_RESTCONF_YANG_HTTP_Verification_Commands
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Verify management IP state | IOS XE | `show ip interface brief` | Management IP is correct and interface is up/up |
| 2 | Verify return route to RESTCONF client | IOS XE | `show ip route <CLIENT_IP>` | Device has a return path to the client |
| 3 | Verify local user exists | IOS XE | `show running-config | include ^username` | Expected RESTCONF user exists |
| 4 | Verify HTTP authentication method | IOS XE | `show running-config | include ^ip http authentication` | HTTP authentication is set to local |
| 5 | Verify HTTPS server is enabled | IOS XE | `show running-config | include ^ip http secure-server` | HTTPS server is enabled |
| 6 | Verify cleartext HTTP is disabled if required | IOS XE | `show running-config | include ^ip http server` | `no ip http server` is present if cleartext HTTP is intentionally disabled |
| 7 | Verify RESTCONF is enabled | IOS XE | `show running-config | include ^restconf` | `restconf` is present |
| 8 | Verify YANG management processes | IOS XE | `show platform software yang-management process` | YANG management processes are running |
| 9 | Verify HTTP source ACL if configured | IOS XE | `show running-config | include ^ip http access-class` | Expected HTTP access class is attached |
| 10 | Verify HTTP source ACL counters if configured | IOS XE | `show access-lists <ACL_NUM>` | RESTCONF client permit entry shows hits after testing |
| 11 | Verify HTTPS reachability from client | RESTCONF client | `curl -k -u <USER>:<SECRET> https://<DEVICE_IP>/` | Client receives an HTTPS response instead of timeout |
| 12 | Verify RESTCONF root discovery if supported | RESTCONF client | `curl -k -u <USER>:<SECRET> https://<DEVICE_IP>/.well-known/host-meta` | Response identifies the RESTCONF root |
| 13 | Verify RESTCONF data root | RESTCONF client | `curl -k -u <USER>:<SECRET> -H "Accept: application/yang-data+json" https://<DEVICE_IP>/restconf/data/` | Device returns RESTCONF JSON response or valid RESTCONF data/error response |
| 14 | Verify JSON GET against a selected model path | RESTCONF client | `curl -k -u <USER>:<SECRET> -H "Accept: application/yang-data+json" https://<DEVICE_IP>/restconf/data/<MODULE>:<PATH>` | Device returns YANG-modeled JSON data |
| 15 | Verify XML GET if XML encoding is tested | RESTCONF client | `curl -k -u <USER>:<SECRET> -H "Accept: application/yang-data+xml" https://<DEVICE_IP>/restconf/data/<MODULE>:<PATH>` | Device returns YANG-modeled XML data |
| 16 | Verify HTTP response code for read | RESTCONF client | `curl -k -i -u <USER>:<SECRET> -H "Accept: application/yang-data+json" https://<DEVICE_IP>/restconf/data/<MODULE>:<PATH>` | HTTP status is in the expected 2xx success range |
| 17 | Verify write payload media type | RESTCONF client | `curl -k -i -u <USER>:<SECRET> -X PATCH -H "Content-Type: application/yang-data+json" -d '<JSON_PAYLOAD>' https://<DEVICE_IP>/restconf/data/<MODULE>:<CONFIG_PATH>` | Device accepts the request with expected 2xx code |
| 18 | Verify configuration effect from CLI | IOS XE | `show running-config | section <FEATURE>` | Running config reflects the RESTCONF write |
| 19 | Verify operational effect from CLI | IOS XE | `show <EQUIVALENT_OPERATIONAL_COMMAND>` | Operational state matches the RESTCONF result |
| 20 | Verify RESTCONF path and model selection | RESTCONF client or YANG tool | `pyang -f tree <MODULE>.yang` | Module tree confirms the URI path being used |
# IOS_XE_RESTCONF_YANG_HTTP_Rollback
! =========================
! Remove HTTP source restriction only
! =========================
configure terminal
!
no ip http access-class <ACL_NUM>
no access-list <ACL_NUM>
!
end
write memory
! =========================
! Disable RESTCONF only
! =========================
configure terminal
!
no restconf
!
end
write memory
! =========================
! Disable HTTPS only if this lab enabled it and no other service depends on it
! =========================
configure terminal
!
no ip http secure-server
!
end
write memory
! =========================
! Restore cleartext HTTP default only if required by a separate lab
! =========================
configure terminal
!
ip http server
!
end
write memory
! =========================
! Remove lab-only HTTP auth and AAA only if nothing else depends on it
! =========================
configure terminal
!
no ip http authentication local
no aaa authentication login default local
!
end
write memory
! =========================
! Full local-auth teardown only if this lab created the user
! =========================
configure terminal
!
no username <USER>
!
end
write memory
# IOS_XE_RESTCONF_YANG_HTTP_Failure_Checks
| Symptom | Device | Command | Likely Cause | Corrective Action |
|---|---|---|---|---|
| Client cannot reach IOS XE device | RESTCONF client | `ping <DEVICE_IP>` | IP path, interface, SVI, VLAN, route, or gateway problem | Fix management reachability first |
| HTTPS times out | IOS XE | `show running-config | include ^ip http secure-server` | HTTPS server is disabled or TCP/443 is filtered | Configure `ip http secure-server` and permit TCP/443 |
| RESTCONF path returns 404 | IOS XE | `show running-config | include ^restconf` | RESTCONF is disabled or URI path is wrong | Configure `restconf` and recheck root, resource, module, and path |
| Authentication fails | IOS XE | `show running-config | include ^username|^ip http authentication` | Missing local user or HTTP authentication is not local | Configure local user and `ip http authentication local` |
| HTTPS works but RESTCONF fails | IOS XE | `show running-config | include ^restconf|^ip http` | HTTP server is enabled but RESTCONF service is not enabled | Configure `restconf` |
| RESTCONF works from one client but not another | IOS XE | `show access-lists <ACL_NUM>` | HTTP access-class blocks the second source | Correct or remove `ip http access-class <ACL_NUM>` |
| GET returns 401 | RESTCONF client | `curl -k -i -u <USER>:<SECRET> https://<DEVICE_IP>/restconf/data/` | Wrong username, wrong password, or HTTP auth mismatch | Correct credentials and HTTP authentication method |
| GET returns 403 | RESTCONF client | `curl -k -i -u <USER>:<SECRET> https://<DEVICE_IP>/restconf/data/` | User authenticated but lacks authorization or source is restricted | Check AAA, privilege, and HTTP source ACL |
| GET returns 404 for a model path | RESTCONF client | `curl -k -i -u <USER>:<SECRET> https://<DEVICE_IP>/restconf/data/<MODULE>:<PATH>` | Wrong module, wrong namespace, wrong path, missing list key, or unsupported model | Rebuild the URI from the YANG tree |
| GET returns 406 | RESTCONF client | Request headers | Unsupported `Accept` header | Use `Accept: application/yang-data+json` or `Accept: application/yang-data+xml` |
| POST, PUT, or PATCH returns 415 | RESTCONF client | Request headers | Missing or wrong `Content-Type` header | Use `Content-Type: application/yang-data+json` or `application/yang-data+xml` |
| Write returns 400 | RESTCONF client | HTTP response body | Payload structure does not match the YANG model | Rebuild payload from the module tree and required keys |
| Write returns method not allowed | RESTCONF client | HTTP response code | Wrong HTTP method for target resource | Use GET for read, PATCH for update, PUT for replace, POST for create or operations, DELETE for delete |
| Write fails against operational state | RESTCONF client | HTTP response body | Target node is read-only or `config false` | Use a read-write config node only |
| DELETE fails for list item | RESTCONF client | RESTCONF URI | Missing list key in the URI | Include the list key in the RESTCONF path |
| RESTCONF output does not match CLI | IOS XE | `show <EQUIVALENT_CLI_COMMAND>` | Wrong model family or wrong subtree selected | Compare Cisco native, IETF, and OpenConfig paths |
| YANG process appears unhealthy | IOS XE | `show platform software yang-management process` | RESTCONF/YANG management process issue | Disable and re-enable RESTCONF during lab maintenance |
| User expects vty `transport input ssh` to control RESTCONF | IOS XE | `show running-config | section line vty` | Confusion between CLI SSH and HTTPS RESTCONF | Troubleshoot RESTCONF through HTTP/HTTPS settings, not vty transport |
| Browser warning appears | RESTCONF client | Browser or curl output | Self-signed certificate is being used | For lab curl tests use `-k`; for production use proper PKI and trust chain |
##### Source_Basis
# IOS_XE_RESTCONF_YANG_HTTP_Mental_Model
# IOS_XE_RESTCONF_YANG_HTTP_Configuration_Checklist
# IOS_XE_RESTCONF_YANG_HTTP_Skeleton
# IOS_XE_RESTCONF_YANG_HTTP_Verification_Commands
# IOS_XE_RESTCONF_YANG_HTTP_Rollback
# IOS_XE_RESTCONF_YANG_HTTP_Failure_Checks

