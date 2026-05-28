IOS_XE_REST_API_Client_Workflow.md

IOS_XE_REST_API_Client_Workflow

# IOS_XE_REST_API_Client_Workflow_Mental_Model
| Concept | Operational Meaning |
|---|---|
| REST API client workflow | The client builds and sends HTTP requests to a RESTCONF endpoint, then validates status code, response body, and device state |
| IOS XE REST API in this lab set | Treat this as RESTCONF unless the lab explicitly proves it is a different IOS XE API surface |
| Client, not protocol note | This note is about how to drive the API from curl, Postman, Python, or pyATS. RESTCONF service configuration belongs in the RESTCONF note |
| Server prerequisite | IOS XE must already have management reachability, credentials, HTTPS, HTTP authentication, and `restconf` enabled |
| URL | The URL identifies the device, RESTCONF root, resource type, module, and YANG path |
| Method | GET reads, PATCH updates, PUT creates or replaces, POST creates or invokes operations, DELETE removes, HEAD checks metadata |
| Headers | `Accept` controls response format. `Content-Type` controls request payload format |
| Authentication | Lab workflows usually use HTTP basic authentication with a local privilege user |
| TLS | IOS XE lab devices commonly use self-signed certificates, so curl tests often use `-k` |
| Payload | A write request must match the YANG model structure exactly |
| Status code | The status code tells whether the HTTP request succeeded before you interpret the payload |
| Response body | The response body should be structured JSON or XML, not unstructured CLI text |
| Read before write | Run a GET against the target path before POST, PUT, PATCH, or DELETE |
| Verify after write | REST success is not the end. Verify with REST GET and IOS XE CLI |
| Idempotence | Prefer repeatable requests and predictable payload files for lab automation |
| Related labs | `ios-xe-rest-api-final`, `restconf-r1-bridge-final`, `ios-xe-netconf-restconf-final` |
# IOS_XE_REST_API_Client_Workflow_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the IOS XE RESTCONF server has a reachable management IP | IOS XE | `show ip interface brief` | Management interface or SVI has the expected IP and is up/up |
| 2 | Confirm the IOS XE device has a return route to the API client | IOS XE | `show ip route <CLIENT_IP>` | Route exists back toward the API client |
| 3 | Confirm client reachability to the IOS XE device | API client | `ping <DEVICE_IP>` | API client can reach the IOS XE management IP |
| 4 | Confirm local API user exists | IOS XE | `show running-config | include ^username` | Expected local user exists |
| 5 | Confirm HTTP local authentication is enabled | IOS XE | `show running-config | include ^ip http authentication` | Output shows `ip http authentication local` |
| 6 | Confirm HTTPS server is enabled | IOS XE | `show running-config | include ^ip http secure-server` | HTTPS server is enabled |
| 7 | Confirm RESTCONF is enabled | IOS XE | `show running-config | include ^restconf` | `restconf` is present |
| 8 | Confirm YANG management processes are running | IOS XE | `show platform software yang-management process` | YANG management processes are running |
| 9 | Set client-side variables for repeatable curl testing | API client | `export DEVICE=<DEVICE_IP>` | Device IP is stored in client shell |
| 10 | Set the REST username variable | API client | `export REST_USER=<USER>` | Username is stored in client shell |
| 11 | Set the REST password variable | API client | `export REST_PASS='<SECRET>'` | Password is stored in client shell for lab testing |
| 12 | Define the RESTCONF base URL | API client | `export BASE_URL="https://${DEVICE}/restconf"` | Base URL points to the RESTCONF root |
| 13 | Define JSON Accept header | API client | `export ACCEPT_JSON="Accept: application/yang-data+json"` | Client can request JSON output consistently |
| 14 | Define JSON Content-Type header | API client | `export CONTENT_JSON="Content-Type: application/yang-data+json"` | Client can send JSON payloads consistently |
| 15 | Test HTTPS reachability before testing RESTCONF paths | API client | `curl -k -i -u "${REST_USER}:${REST_PASS}" https://${DEVICE}/` | Client receives an HTTPS response instead of timeout |
| 16 | Test RESTCONF root discovery if supported | API client | `curl -k -i -u "${REST_USER}:${REST_PASS}" https://${DEVICE}/.well-known/host-meta` | Client receives root discovery response or a valid HTTP response |
| 17 | Test RESTCONF data root | API client | `curl -k -i -u "${REST_USER}:${REST_PASS}" -H "${ACCEPT_JSON}" "${BASE_URL}/data/"` | Client receives JSON data or valid RESTCONF response |
| 18 | Choose the operation before building the request | Operator | `Operation: <READ | CREATE | UPDATE | REPLACE | DELETE>` | Method choice is explicit before the call is sent |
| 19 | Map the operation to an HTTP method | Operator | `Method: <GET | POST | PATCH | PUT | DELETE | HEAD>` | Method matches intended CRUD behavior |
| 20 | Identify the target YANG module and path | Operator | `Target: <MODULE>:<PATH>` | RESTCONF path is known before request construction |
| 21 | Build a read URL first | API client | `export TARGET_URL="${BASE_URL}/data/<MODULE>:<PATH>"` | Target URL points to one YANG-modeled resource |
| 22 | Run a GET against the target path | API client | `curl -k -i -u "${REST_USER}:${REST_PASS}" -H "${ACCEPT_JSON}" "${TARGET_URL}"` | Client receives expected 2xx response and structured JSON |
| 23 | Compare GET output to CLI truth | IOS XE | `show <EQUIVALENT_CLI_COMMAND>` | CLI output matches the RESTCONF data |
| 24 | Create a JSON payload file for write testing | API client | `cat > payload.json <<'EOF'` | Client begins a reusable payload file |
| 25 | Add model-correct JSON payload content | API client | `<JSON_PAYLOAD>` | Payload follows the YANG model structure |
| 26 | Close the payload file | API client | `EOF` | `payload.json` exists on the client |
| 27 | Validate payload is readable before sending | API client | `cat payload.json` | Payload content is visible and correct |
| 28 | Send PATCH for a partial update | API client | `curl -k -i -u "${REST_USER}:${REST_PASS}" -X PATCH -H "${ACCEPT_JSON}" -H "${CONTENT_JSON}" -d @payload.json "${TARGET_URL}"` | Device returns expected 2xx success code |
| 29 | Send PUT for create-or-replace behavior only when intended | API client | `curl -k -i -u "${REST_USER}:${REST_PASS}" -X PUT -H "${ACCEPT_JSON}" -H "${CONTENT_JSON}" -d @payload.json "${TARGET_URL}"` | Device creates or replaces the target resource |
| 30 | Send POST for create or operation endpoints only when intended | API client | `curl -k -i -u "${REST_USER}:${REST_PASS}" -X POST -H "${ACCEPT_JSON}" -H "${CONTENT_JSON}" -d @payload.json "${BASE_URL}/data/<MODULE>:<PARENT_PATH>"` | Device creates a child resource or invokes the intended operation |
| 31 | Send DELETE only against the exact resource to remove | API client | `curl -k -i -u "${REST_USER}:${REST_PASS}" -X DELETE "${TARGET_URL}"` | Device deletes the target resource if allowed |
| 32 | Verify the write with RESTCONF GET | API client | `curl -k -i -u "${REST_USER}:${REST_PASS}" -H "${ACCEPT_JSON}" "${TARGET_URL}"` | Returned data shows the intended change or expected absence after DELETE |
| 33 | Verify the write with IOS XE CLI | IOS XE | `show running-config | section <FEATURE>` | Running configuration reflects the API change |
| 34 | Verify the operational effect with IOS XE CLI | IOS XE | `show <EQUIVALENT_OPERATIONAL_COMMAND>` | Operational state reflects the intended result |
| 35 | Save IOS XE configuration if the API change modified running config | IOS XE | `write memory` | Startup-config preserves the working configuration |
| 36 | Record method, URL, headers, payload, status code, and verification command | Operator | `Request record: <METHOD> <URL> <STATUS> <VERIFY_COMMAND>` | Lab result is repeatable |
# IOS_XE_REST_API_Client_Workflow_Skeleton
! =========================
! IOS XE server prerequisite check
! =========================
show ip interface brief
show ip route <CLIENT_IP>
show running-config | include ^username|^ip http authentication|^ip http secure-server|^restconf
show platform software yang-management process
! =========================
! Minimal IOS XE RESTCONF server baseline
! Use only if the RESTCONF note has not already configured the server
! =========================
configure terminal
!
username <USER> privilege 15 algorithm-type scrypt secret <SECRET>
! If unsupported:
! username <USER> privilege 15 secret <SECRET>
!
ip http authentication local
ip http secure-server
restconf
!
end
write memory
! =========================
! API client variables
! =========================
export DEVICE=<DEVICE_IP>
export REST_USER=<USER>
export REST_PASS='<SECRET>'
export BASE_URL="https://${DEVICE}/restconf"
export ACCEPT_JSON="Accept: application/yang-data+json"
export CONTENT_JSON="Content-Type: application/yang-data+json"
! =========================
! Basic HTTPS test
! =========================
curl -k -i -u "${REST_USER}:${REST_PASS}" \
  https://${DEVICE}/
! =========================
! RESTCONF data root test
! =========================
curl -k -i -u "${REST_USER}:${REST_PASS}" \
  -H "${ACCEPT_JSON}" \
  "${BASE_URL}/data/"
! =========================
! GET pattern
! =========================
export TARGET_URL="${BASE_URL}/data/<MODULE>:<PATH>"
curl -k -i -u "${REST_USER}:${REST_PASS}" \
  -H "${ACCEPT_JSON}" \
  "${TARGET_URL}"
! =========================
! JSON payload file pattern
! =========================
cat > payload.json <<'EOF'
<JSON_PAYLOAD>
EOF
cat payload.json
! =========================
! PATCH pattern
! =========================
curl -k -i -u "${REST_USER}:${REST_PASS}" \
  -X PATCH \
  -H "${ACCEPT_JSON}" \
  -H "${CONTENT_JSON}" \
  -d @payload.json \
  "${TARGET_URL}"
! =========================
! PUT pattern
! =========================
curl -k -i -u "${REST_USER}:${REST_PASS}" \
  -X PUT \
  -H "${ACCEPT_JSON}" \
  -H "${CONTENT_JSON}" \
  -d @payload.json \
  "${TARGET_URL}"
! =========================
! POST pattern
! =========================
curl -k -i -u "${REST_USER}:${REST_PASS}" \
  -X POST \
  -H "${ACCEPT_JSON}" \
  -H "${CONTENT_JSON}" \
  -d @payload.json \
  "${BASE_URL}/data/<MODULE>:<PARENT_PATH>"
! =========================
! DELETE pattern
! =========================
curl -k -i -u "${REST_USER}:${REST_PASS}" \
  -X DELETE \
  "${TARGET_URL}"
! =========================
! Verify after API operation
! =========================
curl -k -i -u "${REST_USER}:${REST_PASS}" \
  -H "${ACCEPT_JSON}" \
  "${TARGET_URL}"
! On IOS XE:
show running-config | section <FEATURE>
show <EQUIVALENT_OPERATIONAL_COMMAND>
write memory
# IOS_XE_REST_API_Client_Workflow_Verification_Commands
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Verify IOS XE management IP | IOS XE | `show ip interface brief` | Management IP is correct and interface is up/up |
| 2 | Verify route back to client | IOS XE | `show ip route <CLIENT_IP>` | Device has a return path |
| 3 | Verify RESTCONF server config | IOS XE | `show running-config | include ^restconf|^ip http authentication|^ip http secure-server` | RESTCONF, local HTTP auth, and HTTPS are configured |
| 4 | Verify local API user | IOS XE | `show running-config | include ^username` | Expected API user exists |
| 5 | Verify YANG management processes | IOS XE | `show platform software yang-management process` | YANG management processes are running |
| 6 | Verify client variables | API client | `env | grep -E 'DEVICE|REST_USER|BASE_URL'` | Client variables are present |
| 7 | Verify HTTPS response | API client | `curl -k -i -u "${REST_USER}:${REST_PASS}" https://${DEVICE}/` | Client receives HTTP response, not timeout |
| 8 | Verify RESTCONF data root | API client | `curl -k -i -u "${REST_USER}:${REST_PASS}" -H "${ACCEPT_JSON}" "${BASE_URL}/data/"` | Client receives valid RESTCONF response |
| 9 | Verify target URL is correct | API client | `echo "${TARGET_URL}"` | URL contains `/restconf/data/<MODULE>:<PATH>` |
| 10 | Verify GET response code | API client | `curl -k -s -o /tmp/rest.out -w "%{http_code}\n" -u "${REST_USER}:${REST_PASS}" -H "${ACCEPT_JSON}" "${TARGET_URL}"` | Status code is expected 2xx for a successful read |
| 11 | Verify GET response body | API client | `cat /tmp/rest.out` | Body contains structured JSON or expected RESTCONF error details |
| 12 | Verify response headers and body together | API client | `curl -k -i -u "${REST_USER}:${REST_PASS}" -H "${ACCEPT_JSON}" "${TARGET_URL}"` | Output includes HTTP status, headers, and body |
| 13 | Verify payload file exists | API client | `ls -l payload.json` | Payload file exists before write testing |
| 14 | Verify payload content | API client | `cat payload.json` | Payload matches the intended YANG model structure |
| 15 | Verify write response code | API client | `curl -k -s -o /tmp/write.out -w "%{http_code}\n" -u "${REST_USER}:${REST_PASS}" -X PATCH -H "${ACCEPT_JSON}" -H "${CONTENT_JSON}" -d @payload.json "${TARGET_URL}"` | Status code is expected 2xx for a successful write |
| 16 | Verify write response body | API client | `cat /tmp/write.out` | Body is empty for valid no-content response or contains structured result details |
| 17 | Verify change with RESTCONF readback | API client | `curl -k -i -u "${REST_USER}:${REST_PASS}" -H "${ACCEPT_JSON}" "${TARGET_URL}"` | Returned JSON reflects the intended state |
| 18 | Verify change with IOS XE running config | IOS XE | `show running-config | section <FEATURE>` | Running config reflects the API change |
| 19 | Verify operational state | IOS XE | `show <EQUIVALENT_OPERATIONAL_COMMAND>` | Operational state matches the intended result |
| 20 | Verify saved config if required | IOS XE | `show startup-config | section <FEATURE>` | Startup-config contains the intended persistent configuration |
# IOS_XE_REST_API_Client_Workflow_Rollback
! =========================
! Client-side cleanup
! =========================
rm -f payload.json
rm -f /tmp/rest.out
rm -f /tmp/write.out
unset DEVICE
unset REST_USER
unset REST_PASS
unset BASE_URL
unset ACCEPT_JSON
unset CONTENT_JSON
unset TARGET_URL
! =========================
! Roll back an API-created config object with DELETE
! Use only when TARGET_URL points to the exact object created by the lab
! =========================
export DEVICE=<DEVICE_IP>
export REST_USER=<USER>
export REST_PASS='<SECRET>'
export BASE_URL="https://${DEVICE}/restconf"
export TARGET_URL="${BASE_URL}/data/<MODULE>:<CONFIG_PATH>"
curl -k -i -u "${REST_USER}:${REST_PASS}" \
  -X DELETE \
  "${TARGET_URL}"
! =========================
! Roll back an API-updated object with a known-good payload
! =========================
cat > rollback_payload.json <<'EOF'
<KNOWN_GOOD_JSON_PAYLOAD>
EOF
curl -k -i -u "${REST_USER}:${REST_PASS}" \
  -X PATCH \
  -H "Accept: application/yang-data+json" \
  -H "Content-Type: application/yang-data+json" \
  -d @rollback_payload.json \
  "${TARGET_URL}"
! =========================
! IOS XE verification after rollback
! =========================
show running-config | section <FEATURE>
show <EQUIVALENT_OPERATIONAL_COMMAND>
write memory
! =========================
! Do not disable RESTCONF here unless this workflow note enabled it
! RESTCONF service rollback belongs in IOS_XE_RESTCONF_YANG_HTTP
! =========================
# IOS_XE_REST_API_Client_Workflow_Failure_Checks
| Symptom | Device | Command | Likely Cause | Corrective Action |
|---|---|---|---|---|
| Client cannot ping the IOS XE device | API client | `ping <DEVICE_IP>` | IP path, interface, VLAN, route, or gateway problem | Fix management reachability first |
| HTTPS test times out | IOS XE | `show running-config | include ^ip http secure-server` | HTTPS server disabled or TCP/443 blocked | Enable `ip http secure-server` and permit TCP/443 |
| RESTCONF data root fails | IOS XE | `show running-config | include ^restconf` | RESTCONF is not enabled | Configure `restconf` |
| API returns 401 | API client | `curl -k -i -u "${REST_USER}:${REST_PASS}" "${BASE_URL}/data/"` | Wrong username, wrong password, or HTTP auth mismatch | Verify local user and `ip http authentication local` |
| API returns 403 | IOS XE | `show access-lists` | Auth succeeded but authorization or HTTP access-class blocks the request | Check user privilege, AAA, and HTTP source restrictions |
| API returns 404 | API client | `echo "${TARGET_URL}"` | Wrong RESTCONF root, module, path, list key, or unsupported model | Rebuild the URL from the YANG model tree |
| API returns 405 | API client | Request method | Wrong HTTP method for the target resource | Use GET for read, PATCH for update, PUT for replace, POST for create or operation, DELETE for remove |
| API returns 406 | API client | Request headers | Unsupported `Accept` header | Use `Accept: application/yang-data+json` or `application/yang-data+xml` |
| API returns 415 | API client | Request headers | Missing or wrong `Content-Type` header on a write | Use `Content-Type: application/yang-data+json` or `application/yang-data+xml` |
| API returns 400 on write | API client | `cat payload.json` | Payload does not match the YANG model structure | Correct JSON keys, nesting, list keys, and value types |
| API returns 409 | API client | RESTCONF response body | Resource conflict, duplicate key, or invalid current state | GET the current resource, adjust payload, and retry with correct method |
| API returns 500 | IOS XE | `show platform software yang-management process` | Device-side RESTCONF or YANG management process issue | Check process health and re-enable RESTCONF during lab maintenance |
| GET works but PATCH fails | API client | RESTCONF response body | Target path is read-only or marked `config false` | Choose a read-write config path |
| DELETE fails | API client | `echo "${TARGET_URL}"` | URL points to parent path instead of exact list entry or resource | Include the full resource path and list key |
| POST creates duplicate or fails | API client | `cat payload.json` | POST was used against the wrong parent path or existing key | Use correct parent path, unique key, or use PUT/PATCH instead |
| PUT overwrites more than expected | API client | Request method and target URL | PUT replaced the full resource | Use PATCH for partial updates |
| curl command breaks because of shell quoting | API client | `cat payload.json` | Inline JSON quoting failed | Put payload in `payload.json` and use `-d @payload.json` |
| curl fails certificate validation | API client | curl output | IOS XE is using a self-signed certificate | Use `-k` for lab testing or install trusted certificate chain |
| Output is XML when JSON was expected | API client | Request headers | Missing or wrong `Accept` header | Add `Accept: application/yang-data+json` |
| REST response looks good but CLI does not change | IOS XE | `show running-config | section <FEATURE>` | Request read state only, targeted wrong path, or change was operational only | Verify method, target path, and whether node is config or state |
| Change works but disappears after reload | IOS XE | `show startup-config | section <FEATURE>` | Running config was changed but not saved | Run `write memory` or use an API save workflow if supported |
| User treats this as separate from RESTCONF | Operator | Source note review | IOS XE lab is likely RESTCONF API workflow, not a different API protocol | Keep this note as the client workflow layered on `IOS_XE_RESTCONF_YANG_HTTP` |
##### Source_Basis
# IOS_XE_REST_API_Client_Workflow_Mental_Model
# IOS_XE_REST_API_Client_Workflow_Configuration_Checklist
# IOS_XE_REST_API_Client_Workflow_Skeleton
# IOS_XE_REST_API_Client_Workflow_Verification_Commands
# IOS_XE_REST_API_Client_Workflow_Rollback
# IOS_XE_REST_API_Client_Workflow_Failure_Checks
