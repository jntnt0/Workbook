
IOS ZBF URL filtering is a layered model: define URL match objects/classes, build a policy-map type inspect urlfilter, attach that URL-filter policy under an HTTP inspect class, then attach the parent inspect policy to the zone pair. Cisco’s IOS ZBF content filtering guide shows that local URL filtering uses parameter-map type urlf-glob and class-map type urlfilter, then policy-map type inspect urlfilter, and finally service-policy urlfilter under the HTTP inspect class.  

Zone_Based_Firewall_URL_Filtering.md
# Zone_Based_Firewall_URL_Filtering
# Zone_Based_Firewall_URL_Filtering_Mental_Model
| Concept | Operational Meaning |
|---|---|
| ZBFW URL filtering | Layer 7 HTTP URL enforcement attached inside a Zone-Based Firewall policy |
| Parent ZBFW policy | The normal `policy-map type inspect` that controls traffic between zones |
| URL filter policy | A child `policy-map type inspect urlfilter` that decides what URL requests are allowed, reset, or logged |
| HTTP inspect class | The parent class that matches HTTP traffic before the URL filter policy can be invoked |
| `service-policy urlfilter` | Attaches the URL filter policy under the HTTP inspection class |
| Local URL filtering | Uses locally configured trusted domains, blocked domains, and blocked keywords |
| Trusted domain class | Allows matched domains before broader block logic is evaluated |
| Untrusted domain class | Blocks or resets matched domains |
| Blocked keyword class | Blocks URL paths or strings that contain forbidden keywords |
| URL parameter map | Stores URL glob patterns used by URL filter class maps |
| `reset` action | Blocks URL traffic by resetting the connection |
| `allow` action | Allows URL traffic that matches the class |
| `log` action | Records matching URL filter activity |
| Zone pair direction | URL filtering only applies in the direction where the parent zone-pair policy is attached |
| HTTP limitation | Basic IOS URL filtering targets HTTP visibility; HTTPS URL path filtering normally requires decryption or a different security stack |
| Lab purpose | Prove normal HTTP works, blocked domains or keywords fail, allowed domains still work, and zone-pair counters show the policy path |
# Zone_Based_Firewall_URL_Filtering_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm routed ZBFW baseline is already working | R1 | `show zone security`<br>`show zone-pair security`<br>`show policy-map type inspect zone-pair` | Zones, zone pairs, and base inspect policy exist |
| 2 | Confirm inside and outside interfaces are assigned to zones | R1 | `show zone security` | Inside interface appears in `INSIDE`; outside interface appears in `OUTSIDE` |
| 3 | Confirm routing before URL filtering | R1 | `show ip route`<br>`show ip cef` | Router has valid routes for inside clients and outside web server |
| 4 | Confirm HTTP baseline before filtering | Inside Host | `curl http://<web_server>/<allowed_path>`<br>`copy http://<web_server>/<allowed_file> null:` | HTTP succeeds before URL filter enforcement |
| 5 | Check existing URL filter configuration | R1 | `show running-config \| include urlfilter\|urlf-glob\|trend\|websense` | Existing URL filter objects are known |
| 6 | Check existing class maps and policy maps | R1 | `show class-map type inspect`<br>`show policy-map type inspect` | Parent ZBFW policy names are known |
| 7 | Enter global configuration mode | R1 | `configure terminal` | CLI enters global configuration mode |
| 8 | Create trusted-domain parameter map | R1 | `parameter-map type urlf-glob URLF_TRUSTED_DOMAINS` | CLI enters URL glob parameter-map mode |
| 9 | Add trusted domain patterns | R1 | `pattern <trusted_domain>`<br>`pattern *.<trusted_domain>` | Trusted domains are stored as local URL match patterns |
| 10 | Exit trusted-domain parameter map | R1 | `exit` | CLI returns to global configuration mode |
| 11 | Create blocked-domain parameter map | R1 | `parameter-map type urlf-glob URLF_BLOCKED_DOMAINS` | CLI enters URL glob parameter-map mode |
| 12 | Add blocked domain patterns | R1 | `pattern <blocked_domain>`<br>`pattern *.<blocked_domain>` | Blocked domains are stored as local URL match patterns |
| 13 | Exit blocked-domain parameter map | R1 | `exit` | CLI returns to global configuration mode |
| 14 | Create blocked-keyword parameter map | R1 | `parameter-map type urlf-glob URLF_BLOCKED_KEYWORDS` | CLI enters URL glob parameter-map mode |
| 15 | Add blocked keyword patterns | R1 | `pattern <blocked_keyword>`<br>`pattern *<blocked_keyword>*` | URL keyword patterns are stored locally |
| 16 | Exit blocked-keyword parameter map | R1 | `exit` | CLI returns to global configuration mode |
| 17 | Create trusted-domain URL filter class | R1 | `class-map type urlfilter match-any CM_URLF_TRUSTED_DOMAINS` | CLI enters URL filter class-map mode |
| 18 | Match trusted-domain parameter map | R1 | `match server-domain urlf-glob URLF_TRUSTED_DOMAINS` | Trusted domain class matches configured allowlist domains |
| 19 | Exit trusted URL filter class | R1 | `exit` | CLI returns to global configuration mode |
| 20 | Create blocked-domain URL filter class | R1 | `class-map type urlfilter match-any CM_URLF_BLOCKED_DOMAINS` | CLI enters URL filter class-map mode |
| 21 | Match blocked-domain parameter map | R1 | `match server-domain urlf-glob URLF_BLOCKED_DOMAINS` | Blocked domain class matches configured denylist domains |
| 22 | Exit blocked-domain URL filter class | R1 | `exit` | CLI returns to global configuration mode |
| 23 | Create blocked-keyword URL filter class | R1 | `class-map type urlfilter match-any CM_URLF_BLOCKED_KEYWORDS` | CLI enters URL filter class-map mode |
| 24 | Match blocked-keyword parameter map | R1 | `match url-keyword urlf-glob URLF_BLOCKED_KEYWORDS` | Keyword class matches configured URL strings |
| 25 | Exit blocked-keyword URL filter class | R1 | `exit` | CLI returns to global configuration mode |
| 26 | Create local URL filter policy | R1 | `policy-map type inspect urlfilter PM_URL_FILTER_LOCAL` | CLI enters URL filter policy-map mode |
| 27 | Add local URL filter parameter statement | R1 | `parameter type urlfpolicy local` | URL filter policy uses local filtering behavior |
| 28 | Allow trusted domains first | R1 | `class type urlfilter CM_URLF_TRUSTED_DOMAINS`<br>`allow`<br>`log` | Trusted domains are allowed and logged |
| 29 | Block denied domains | R1 | `class type urlfilter CM_URLF_BLOCKED_DOMAINS`<br>`reset`<br>`log` | Blocked domains are reset and logged |
| 30 | Block denied keywords | R1 | `class type urlfilter CM_URLF_BLOCKED_KEYWORDS`<br>`reset`<br>`log` | URLs containing blocked keywords are reset and logged |
| 31 | Exit URL filter policy mode | R1 | `exit` | CLI returns to global configuration mode |
| 32 | Create HTTP inspect class for parent ZBFW policy | R1 | `class-map type inspect match-all CM_HTTP_URL_FILTER` | CLI enters inspect class-map mode |
| 33 | Match HTTP traffic | R1 | `match protocol http` | HTTP traffic is selected for URL filtering |
| 34 | Exit inspect class-map mode | R1 | `exit` | CLI returns to global configuration mode |
| 35 | Enter parent inside-to-outside inspect policy | R1 | `policy-map type inspect PM_INSIDE_TO_OUTSIDE` | CLI enters parent ZBFW policy-map mode |
| 36 | Add HTTP URL filter class before broader permit classes | R1 | `class type inspect CM_HTTP_URL_FILTER` | HTTP traffic hits the URL-filter class before generic inspect logic |
| 37 | Inspect HTTP traffic | R1 | `inspect` | HTTP receives stateful inspection |
| 38 | Attach URL filter child policy | R1 | `service-policy urlfilter PM_URL_FILTER_LOCAL` | URL filter policy is invoked for matched HTTP traffic |
| 39 | Keep or add class-default drop | R1 | `class class-default`<br>`drop log` | Unmatched interzone traffic is dropped and logged |
| 40 | Exit parent policy-map mode | R1 | `exit`<br>`exit` | CLI returns to global configuration mode |
| 41 | Confirm zone pair uses parent inspect policy | R1 | `zone-pair security ZP_INSIDE_TO_OUTSIDE source INSIDE destination OUTSIDE`<br>`service-policy type inspect PM_INSIDE_TO_OUTSIDE` | Parent policy containing URL filter attachment is active on the zone pair |
| 42 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 43 | Verify URL parameter maps | R1 | `show running-config \| section parameter-map type urlf-glob` | Trusted, blocked, and keyword parameter maps are present |
| 44 | Verify URL filter class maps | R1 | `show class-map type urlfilter` | URL filter classes reference the correct parameter maps |
| 45 | Verify URL filter policy | R1 | `show policy-map type inspect urlfilter PM_URL_FILTER_LOCAL` | URL filter policy shows allow, reset, and log actions |
| 46 | Verify parent inspect policy | R1 | `show policy-map type inspect PM_INSIDE_TO_OUTSIDE` | HTTP class includes `inspect` and `service-policy urlfilter PM_URL_FILTER_LOCAL` |
| 47 | Verify zone-pair attachment | R1 | `show zone-pair security` | Zone pair references the parent inspect policy |
| 48 | Test allowed HTTP URL | Inside Host | `curl http://<allowed_domain>/<allowed_path>`<br>`copy http://<allowed_domain>/<allowed_file> null:` | Allowed HTTP URL succeeds |
| 49 | Test blocked domain | Inside Host | `curl http://<blocked_domain>/<path>` | Blocked domain fails or resets |
| 50 | Test blocked keyword | Inside Host | `curl http://<web_server>/<blocked_keyword_path>` | URL containing blocked keyword fails or resets |
| 51 | Verify policy counters | R1 | `show policy-map type inspect zone-pair ZP_INSIDE_TO_OUTSIDE` | HTTP URL filter class counters increment |
| 52 | Verify logs | R1 | `show logging` | URL filter allow/reset activity appears if logging is supported and enabled |
| 53 | Save configuration | R1 | `write memory` | URL filtering configuration is saved |
# Zone_Based_Firewall_URL_Filtering_Simple_HTTP_URL_String_Block_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the lab only needs simple HTTP URL string matching | R1 | `show version`<br>`show parser dump \| include match protocol http url` | Image supports NBAR HTTP URL matching syntax |
| 2 | Confirm HTTP baseline works | Inside Host | `curl http://<web_server>/<allowed_file>` | Allowed file succeeds before filtering |
| 3 | Enter global configuration mode | R1 | `configure terminal` | CLI enters global configuration mode |
| 4 | Create inspect class for blocked URL strings | R1 | `class-map type inspect match-any CM_HTTP_BLOCKED_URLS` | CLI enters inspect class-map mode |
| 5 | Match blocked URL extension or string | R1 | `match protocol http url *.<blocked_extension>` | HTTP URL containing the blocked pattern is matched |
| 6 | Add second URL pattern if required | R1 | `match protocol http url *.<second_blocked_extension>` | Multiple blocked URL patterns can match the class |
| 7 | Exit class-map mode | R1 | `exit` | CLI returns to global configuration mode |
| 8 | Enter parent ZBFW policy | R1 | `policy-map type inspect PM_INSIDE_TO_OUTSIDE` | CLI enters parent inspect policy |
| 9 | Add blocked URL class before generic HTTP allow class | R1 | `class type inspect CM_HTTP_BLOCKED_URLS` | Block logic is evaluated before broader HTTP inspection |
| 10 | Drop matching URL traffic | R1 | `drop log` | Matched HTTP URL traffic is blocked and logged |
| 11 | Add generic HTTP inspect class below block class | R1 | `class type inspect CM_HTTP_ALLOWED`<br>`inspect` | Other HTTP traffic is statefully allowed |
| 12 | Keep explicit default drop | R1 | `class class-default`<br>`drop log` | Unmatched interzone traffic is dropped |
| 13 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 14 | Test blocked URL | Inside Host | `curl http://<web_server>/<blocked_file>` | HTTP URL matching blocked pattern fails |
| 15 | Test allowed URL | Inside Host | `curl http://<web_server>/<allowed_file>` | HTTP URL not matching blocked pattern succeeds |
| 16 | Verify counters | R1 | `show policy-map type inspect zone-pair ZP_INSIDE_TO_OUTSIDE` | Block class counters increment for blocked URLs |
| 17 | Save configuration | R1 | `write memory` | Configuration is saved |
# Zone_Based_Firewall_URL_Filtering_Skeleton
configure terminal
parameter-map type urlf-glob URLF_TRUSTED_DOMAINS
 pattern <trusted_domain>
 pattern *.<trusted_domain>
exit
parameter-map type urlf-glob URLF_BLOCKED_DOMAINS
 pattern <blocked_domain>
 pattern *.<blocked_domain>
exit
parameter-map type urlf-glob URLF_BLOCKED_KEYWORDS
 pattern <blocked_keyword>
 pattern *<blocked_keyword>*
exit
class-map type urlfilter match-any CM_URLF_TRUSTED_DOMAINS
 match server-domain urlf-glob URLF_TRUSTED_DOMAINS
exit
class-map type urlfilter match-any CM_URLF_BLOCKED_DOMAINS
 match server-domain urlf-glob URLF_BLOCKED_DOMAINS
exit
class-map type urlfilter match-any CM_URLF_BLOCKED_KEYWORDS
 match url-keyword urlf-glob URLF_BLOCKED_KEYWORDS
exit
policy-map type inspect urlfilter PM_URL_FILTER_LOCAL
 parameter type urlfpolicy local
 class type urlfilter CM_URLF_TRUSTED_DOMAINS
  allow
  log
 class type urlfilter CM_URLF_BLOCKED_DOMAINS
  reset
  log
 class type urlfilter CM_URLF_BLOCKED_KEYWORDS
  reset
  log
exit
class-map type inspect match-all CM_HTTP_URL_FILTER
 match protocol http
exit
policy-map type inspect PM_INSIDE_TO_OUTSIDE
 class type inspect CM_HTTP_URL_FILTER
  inspect
  service-policy urlfilter PM_URL_FILTER_LOCAL
 class class-default
  drop log
exit
zone-pair security ZP_INSIDE_TO_OUTSIDE source INSIDE destination OUTSIDE
 service-policy type inspect PM_INSIDE_TO_OUTSIDE
exit
end
write memory
# Zone_Based_Firewall_URL_Filtering_Trend_Category_Skeleton
configure terminal
parameter-map type trend-global TREND_GLOBAL
 server <trend_server_fqdn_or_ip> retrans <count> timeout <seconds>
 alert on
 cache-entry-lifetime <hours>
 cache-size maximum-memory <kilobytes>
exit
parameter-map type urlfpolicy trend TREND_POLICY_PARAMS
 allow-mode <on_or_off>
 block-page message "<block_message>"
 max-request <request_count>
 max-resp-pak <response_packet_count>
 truncate hostname
exit
class-map type urlfilter trend match-any CM_URLF_TREND_BLOCKED_CATEGORIES
 match url category <category_name>
 match url category <second_category_name>
exit
class-map type urlfilter trend match-any CM_URLF_TREND_BAD_REPUTATION
 match url reputation <reputation_name>
 match url reputation <second_reputation_name>
exit
policy-map type inspect urlfilter PM_URL_FILTER_TREND
 parameter type urlfpolicy trend TREND_POLICY_PARAMS
 class type urlfilter trend CM_URLF_TREND_BLOCKED_CATEGORIES
  reset
  log
 class type urlfilter trend CM_URLF_TREND_BAD_REPUTATION
  reset
  log
exit
class-map type inspect match-all CM_HTTP_URL_FILTER
 match protocol http
exit
policy-map type inspect PM_INSIDE_TO_OUTSIDE
 class type inspect CM_HTTP_URL_FILTER
  inspect
  service-policy urlfilter PM_URL_FILTER_TREND
 class class-default
  drop log
exit
zone-pair security ZP_INSIDE_TO_OUTSIDE source INSIDE destination OUTSIDE
 service-policy type inspect PM_INSIDE_TO_OUTSIDE
exit
end
write memory
# Zone_Based_Firewall_URL_Filtering_Websense_Skeleton
configure terminal
parameter-map type urlfpolicy websense WEBSENSE_POLICY_PARAMS
 server <websense_server_ip>
 port <websense_port> retrans <count> timeout <seconds>
 alert on
 allow-mode <on_or_off>
 urlf-server-log on
 max-request <request_count>
 max-resp-pak <response_packet_count>
 truncate hostname
 cache-size <cache_size>
 cache-entry-lifetime <hours>
 block-page "<block_message>"
exit
policy-map type inspect urlfilter PM_URL_FILTER_WEBSENSE
 parameter type urlfpolicy websense WEBSENSE_POLICY_PARAMS
 class type urlfilter websense <websense_class_name>
  server-specified-action
  log
exit
class-map type inspect match-all CM_HTTP_URL_FILTER
 match protocol http
exit
policy-map type inspect PM_INSIDE_TO_OUTSIDE
 class type inspect CM_HTTP_URL_FILTER
  inspect
  service-policy urlfilter PM_URL_FILTER_WEBSENSE
 class class-default
  drop log
exit
zone-pair security ZP_INSIDE_TO_OUTSIDE source INSIDE destination OUTSIDE
 service-policy type inspect PM_INSIDE_TO_OUTSIDE
exit
end
write memory
# Zone_Based_Firewall_URL_Filtering_Simple_HTTP_URL_String_Block_Skeleton
configure terminal
class-map type inspect match-any CM_HTTP_BLOCKED_URLS
 match protocol http url *.<blocked_extension>
 match protocol http url *<blocked_keyword>*
exit
class-map type inspect match-all CM_HTTP_ALLOWED
 match protocol http
exit
policy-map type inspect PM_INSIDE_TO_OUTSIDE
 class type inspect CM_HTTP_BLOCKED_URLS
  drop log
 class type inspect CM_HTTP_ALLOWED
  inspect
 class class-default
  drop log
exit
zone-pair security ZP_INSIDE_TO_OUTSIDE source INSIDE destination OUTSIDE
 service-policy type inspect PM_INSIDE_TO_OUTSIDE
exit
end
write memory
# Zone_Based_Firewall_URL_Filtering_Verification_Commands
| Check | Device | Command | Expected Result |
|---|---|---|---|
| Zone existence | R1 | `show zone security` | `INSIDE` and `OUTSIDE` zones exist |
| Zone membership | R1 | `show zone security` | Inside and outside interfaces appear under correct zones |
| Zone-pair attachment | R1 | `show zone-pair security` | `ZP_INSIDE_TO_OUTSIDE` references the parent inspect policy |
| Parent inspect policy | R1 | `show policy-map type inspect PM_INSIDE_TO_OUTSIDE` | HTTP class has `inspect` and `service-policy urlfilter` |
| Parent policy counters | R1 | `show policy-map type inspect zone-pair ZP_INSIDE_TO_OUTSIDE` | HTTP class counters increment during tests |
| URL glob parameter maps | R1 | `show running-config \| section parameter-map type urlf-glob` | Trusted, blocked, and keyword patterns are present |
| URL filter class maps | R1 | `show class-map type urlfilter` | URL filter classes match server domains or URL keywords |
| URL filter policy | R1 | `show policy-map type inspect urlfilter PM_URL_FILTER_LOCAL` | URL filter policy shows allow, reset, and log actions |
| Trend global parameter map | R1 | `show running-config \| section parameter-map type trend-global` | Trend server and cache parameters appear if Trend filtering is used |
| Trend policy parameter map | R1 | `show running-config \| section parameter-map type urlfpolicy trend` | Trend per-policy parameters appear if Trend filtering is used |
| Websense policy parameter map | R1 | `show running-config \| section parameter-map type urlfpolicy websense` | Websense server parameters appear if Websense is used |
| Simple URL string class | R1 | `show class-map type inspect CM_HTTP_BLOCKED_URLS` | `match protocol http url` entries appear if using simple URL string matching |
| Allowed URL test | Inside Host | `curl http://<allowed_domain>/<allowed_path>` | Allowed URL succeeds |
| Blocked domain test | Inside Host | `curl http://<blocked_domain>/<path>` | Blocked domain fails or resets |
| Blocked keyword test | Inside Host | `curl http://<web_server>/<blocked_keyword_path>` | URL containing blocked keyword fails or resets |
| Allowed file test | Inside Host | `copy http://<web_server>/<allowed_file> null:` | File not matching block pattern succeeds |
| Blocked file test | Inside Host | `copy http://<web_server>/<blocked_file> null:` | File matching block pattern fails |
| Logging check | R1 | `show logging` | URL filtering reset or drop events appear if logging is enabled |
| Routing check | R1 | `show ip route <web_server_ip>` | Route exists to web server |
| CEF check | R1 | `show ip cef <web_server_ip>` | CEF path exists toward web server |
| Generic ZBFW sanity check | R1 | `show policy-map type inspect zone-pair` | Policy counters confirm traffic is using the intended zone pair |
# Zone_Based_Firewall_URL_Filtering_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enter parent inspect policy | R1 | `configure terminal`<br>`policy-map type inspect PM_INSIDE_TO_OUTSIDE` | CLI enters parent ZBFW policy-map |
| 2 | Remove URL filter attachment from HTTP class | R1 | `class type inspect CM_HTTP_URL_FILTER`<br>`no service-policy urlfilter PM_URL_FILTER_LOCAL` | URL filter child policy is detached |
| 3 | Remove HTTP URL filter class from parent policy if no longer needed | R1 | `no class type inspect CM_HTTP_URL_FILTER` | Parent policy no longer references the URL filter HTTP class |
| 4 | Exit parent policy-map mode | R1 | `exit` | CLI returns to global configuration mode |
| 5 | Remove URL filter policy | R1 | `no policy-map type inspect urlfilter PM_URL_FILTER_LOCAL` | Local URL filter policy is deleted |
| 6 | Remove Trend URL filter policy if used | R1 | `no policy-map type inspect urlfilter PM_URL_FILTER_TREND` | Trend URL filter policy is deleted |
| 7 | Remove Websense URL filter policy if used | R1 | `no policy-map type inspect urlfilter PM_URL_FILTER_WEBSENSE` | Websense URL filter policy is deleted |
| 8 | Remove URL filter class maps | R1 | `no class-map type urlfilter CM_URLF_TRUSTED_DOMAINS`<br>`no class-map type urlfilter CM_URLF_BLOCKED_DOMAINS`<br>`no class-map type urlfilter CM_URLF_BLOCKED_KEYWORDS` | Local URL filter class maps are deleted |
| 9 | Remove Trend URL filter class maps if used | R1 | `no class-map type urlfilter trend CM_URLF_TREND_BLOCKED_CATEGORIES`<br>`no class-map type urlfilter trend CM_URLF_TREND_BAD_REPUTATION` | Trend URL filter class maps are deleted |
| 10 | Remove URL glob parameter maps | R1 | `no parameter-map type urlf-glob URLF_TRUSTED_DOMAINS`<br>`no parameter-map type urlf-glob URLF_BLOCKED_DOMAINS`<br>`no parameter-map type urlf-glob URLF_BLOCKED_KEYWORDS` | Local URL glob patterns are deleted |
| 11 | Remove Trend parameter maps if used | R1 | `no parameter-map type trend-global TREND_GLOBAL`<br>`no parameter-map type urlfpolicy trend TREND_POLICY_PARAMS` | Trend URL filtering parameter maps are deleted |
| 12 | Remove Websense parameter map if used | R1 | `no parameter-map type urlfpolicy websense WEBSENSE_POLICY_PARAMS` | Websense URL filtering parameter map is deleted |
| 13 | Remove simple HTTP URL string block class if used | R1 | `no class-map type inspect CM_HTTP_BLOCKED_URLS` | Simple URL string match class is deleted |
| 14 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 15 | Confirm URL filter cleanup | R1 | `show running-config \| include urlfilter\|urlf-glob\|trend\|websense` | Removed URL filter objects are absent |
| 16 | Confirm parent policy is still valid | R1 | `show policy-map type inspect PM_INSIDE_TO_OUTSIDE` | Parent ZBFW policy still has intended non-URL classes |
| 17 | Test HTTP after rollback | Inside Host | `curl http://<web_server>/<previously_blocked_path>` | Traffic follows remaining ZBFW policy without URL filtering |
| 18 | Save rollback | R1 | `write memory` | Rollback is saved |
# Zone_Based_Firewall_URL_Filtering_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| URL filter never triggers | Parent policy does not match HTTP | R1 | `show policy-map type inspect PM_INSIDE_TO_OUTSIDE`<br>`show class-map type inspect CM_HTTP_URL_FILTER` | Add `match protocol http` to the HTTP inspect class |
| URL filter never triggers | URL filter child policy is not attached under HTTP class | R1 | `show policy-map type inspect PM_INSIDE_TO_OUTSIDE` | Add `service-policy urlfilter PM_URL_FILTER_LOCAL` under the HTTP class |
| URL filter never triggers | Parent inspect policy is not attached to zone pair | R1 | `show zone-pair security` | Attach parent policy with `service-policy type inspect PM_INSIDE_TO_OUTSIDE` |
| URL filter counters do not move | Traffic is not crossing the expected zone pair | R1 | `show ip route <web_server_ip>`<br>`show zone security`<br>`show policy-map type inspect zone-pair` | Fix routing, zone membership, or test direction |
| Blocked domain still works | Domain pattern does not match the server domain in the HTTP request | R1 | `show running-config \| section URLF_BLOCKED_DOMAINS` | Add exact domain or wildcard pattern |
| Blocked keyword still works | Keyword pattern is too narrow or matches the wrong URL component | R1 | `show running-config \| section URLF_BLOCKED_KEYWORDS` | Use a broader wildcard pattern such as `*<keyword>*` |
| Trusted domain is blocked | Trusted class is below blocked class or pattern overlaps with denylist | R1 | `show policy-map type inspect urlfilter PM_URL_FILTER_LOCAL` | Put trusted allow class before broader reset classes and correct overlap |
| All HTTP is blocked | `allow-mode` is off or class-default drops all traffic | R1 | `show policy-map type inspect urlfilter`<br>`show running-config \| section urlfpolicy` | Add intended allow class or enable fail-open behavior only if acceptable |
| HTTPS filtering does not work | Router cannot see encrypted URL path inside TLS | R1 | `show policy-map type inspect zone-pair` | Test with HTTP or use a platform with TLS decryption or web security integration |
| Simple `match protocol http url` is rejected | IOS image or platform does not support that NBAR match syntax | R1 | `show parser dump \| include match protocol http url`<br>`show version` | Use official `type urlfilter` policy syntax or a supported image |
| `class-map type urlfilter` command is rejected | IOS image lacks URL filtering feature support | R1 | `show version` | Use an image that supports IOS content filtering with ZBFW |
| Trend filtering fails open or closed unexpectedly | `allow-mode` setting does not match intended fail behavior | R1 | `show running-config \| section parameter-map type urlfpolicy trend` | Set `allow-mode on` for fail-open or `allow-mode off` for fail-closed |
| Trend category filtering does not work | Trend server or registration is missing | R1 | `show running-config \| section trend-global` | Configure Trend server, licensing, certificate, and registration prerequisites |
| Websense filtering does not work | Websense server, port, retransmit, or timeout setting is wrong | R1 | `show running-config \| section websense` | Correct Websense server parameters |
| Logs do not show URL activity | `log` is missing under URL filter class or logging is not enabled | R1 | `show policy-map type inspect urlfilter`<br>`show logging` | Add `log` under URL filter class and enable logging |
| Allowed HTTP fails after URL filter is added | Parent class-default drops non-matched traffic or URL filter class order is wrong | R1 | `show policy-map type inspect PM_INSIDE_TO_OUTSIDE` | Place specific URL filter class correctly and keep broader HTTP inspect allow as needed |
| Block page is not shown | Policy uses `reset` without block-page behavior or client closes early | R1 | `show running-config \| section urlfpolicy` | Configure `block-page message` or `redirect-url` where supported |
| File-extension test does not match | HTTP server response or URL path does not contain expected extension | Inside Host | `curl -v http://<web_server>/<path>` | Confirm the requested URL contains the string being matched |
| Policy works for one client subnet but not another | Class map also matches an ACL limiting source networks | R1 | `show class-map type inspect`<br>`show access-lists` | Add the missing client subnet to the match ACL |
| Traffic still blocked after rollback | Base ZBFW, ACL, NAT, uRPF, host firewall, or server issue remains | R1 | `show policy-map type inspect zone-pair`<br>`show access-lists`<br>`show ip route` | Troubleshoot remaining forwarding and security controls |
# Zone_Based_Firewall_URL_Filtering_Related_Labs
- zone-based-firewall-url-filter-final
# Zone_Based_Firewall_URL_Filtering_Index
Zone_Based_Firewall_URL_Filtering.md
# Zone_Based_Firewall_URL_Filtering
# Zone_Based_Firewall_URL_Filtering_Mental_Model
# Zone_Based_Firewall_URL_Filtering_Configuration_Checklist
# Zone_Based_Firewall_URL_Filtering_Simple_HTTP_URL_String_Block_Checklist
# Zone_Based_Firewall_URL_Filtering_Skeleton
# Zone_Based_Firewall_URL_Filtering_Trend_Category_Skeleton
# Zone_Based_Firewall_URL_Filtering_Websense_Skeleton
# Zone_Based_Firewall_URL_Filtering_Simple_HTTP_URL_String_Block_Skeleton
# Zone_Based_Firewall_URL_Filtering_Verification_Commands
# Zone_Based_Firewall_URL_Filtering_Rollback
# Zone_Based_Firewall_URL_Filtering_Failure_Checks
# Zone_Based_Firewall_URL_Filtering_Related_Labs
# Zone_Based_Firewall_URL_Filtering_Index
