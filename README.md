# ansible-pi-hole

[![molecule](https://github.com/chubchubsancho/ansible-pi-hole/actions/workflows/molecule.yaml/badge.svg)](https://github.com/chubchubsancho/ansible-pi-hole/actions/workflows/molecule.yaml)
[![License](https://img.shields.io/badge/license-MIT-blue.svg?logo=github&style=flat)](https://raw.githubusercontent.com/chubchubsancho/ansible-pi-hole/master/LICENSE)

An ansible role to install [Pi-Hole](https://pi-hole.net/)

## Role configuration

* `pi_hole_repo` (default: "<https://github.com/pi-hole/pi-hole.git>") - define `pi-hole` repository
* `pi_hole_prerequisites` (default: "git") - define `pi-hole` prerequisites

## Define pi-hole configuration

Add `pi-hole` variable for config. These vars can be omited

* `pi_hole_version` - (default: "HEAD") - define branch/tag/commit of `pi-hole` repository which will be cloned
* `pi_hole_download_dir` - (default: "/home/pihole") - define base directory for clone `pi-hole`
* `pi_hole_install_dir` - (default: "pi-hole") - define directory where to clone `pi-hole`

## Define pi-hole dns configuration

* `pi_hole_dns_upstreams` - (default: []) - `pi-hole` upstreams configurations. Expects a list of dictionnaries in wihch each items contains a key `ip` and an optionnal `port`.
  Example:

  ```yaml
  pi_hole_dns_upstreams:
    - 8.8.8.8
    - 127.0.0.1#5053
  ```

* `pi_hole_dns_cname_deep_inspect` - (default: true) - Use this option to control deep CNAME inspection. Disabling it might be beneficial for very low-end devices.
* `pi_hole_dns_block_esni` - (default: true) - Should _esni. subdomains be blocked by default? Encrypted Server Name Indication (ESNI) is certainly a good step into the right direction to enhance privacy on the web. It prevents on-path observers, including ISPs, coffee shop owners firewalls, from intercepting the TLS Server Name Indication (SNI) extension by encrypting it. This prevents the SNI from being used to determine which websites users are visiting ESNI will obviously cause issues for pixelserv-tls which will be unable to generate matching certificates on-the-fly when it cannot read the SNI. Cloudflare and Firefox are already enabling ESNI. According to the IEFT draft (link above), we can easily restore piselserv-tls's operation by replying NXDOMAIN to_esni. subdomains of blocked domains as this mimics a "not configured for this domain" behavior.
* `pi_hole_dns_edns0_ecs` - (default: true) - Should we overwrite the query source when client information is provided through EDNS0 client subnet (ECS) information? This allows Pi-hole to obtain client IPs even if they are hidden behind the NAT of a router. This feature has been requested and discussed on Discourse where further information how to use it can be found <https://discourse.pi-hole.net/t/support-for-add-subnet-option-from-dnsmasq-ecs-edns0-client-subnet/35940>
* `pi_hole_dns_ignore_localhost` - (default: false) - Should FTL hide queries made by localhost?
* `pi_hole_show_dnssec` - (default: true) -Should FTL analyze and show internally generated DNSSEC queries?
* `pi_hole_dns_analyze_only_a_and_aaaa` - (default: false) - Should FTL analyze *only* A and AAAA queries?
* `pi_hole_dns_pihole_ptr` - (default: "PI.HOLE") - Controls whether and how FTL will reply with for address for which a local interface exists. Changing this setting causes FTL to restart   Possible values are:
  * "NONE"
    Pi-hole will not respond automatically on PTR requests to local interface addresses. Ensure pi.hole and/or hostname records exist elsewhere
  * "HOSTNAME" Serve the machine's hostname. The hostname is queried from the kernel through uname(2)->nodename. If the machine has multiple network interfaces, it can also have multiple nodenames. In this case, it is unspecified and up to the kernel which one will be returned. On Linux, the returned string is what has been set using sethostname(2) which is typically what has been set in /etc/hostname
  * "HOSTNAMEFQDN" Serve the machine's hostname (see limitations above) as fully qualified domain by adding the local domain. If no local domain has been defined (config option dns.domain), FTL tries to query the domain name from the kernel using getdomainname(2). If this fails, FTL appends ".no_fqdn_available" to the hostname
  * "PI.HOLE" Respond with "pi.hole"
* `pi_hole_dns_reply_when_busy` - (default: "ALLOW") - How should FTL handle queries when the gravity database is not available?
  Possible values are
  * "BLOCK" Block all queries when the database is busy
  * "ALLOW" Allow all queries when the database is busy
  * "REFUSE" Refuse all queries which arrive while the database is busy
  * "DROP" Just drop the queries, i.e., never reply to them at all. Despite "REFUSE" sounding similar to "DROP", it turned out that many clients will just immediately retry, causing up to several thousands of queries per second. This does not happen in "DROP" mode
* `pi_hole_block_ttl` - (default: 2) - FTL's internal TTL to be handed out for blocked queries in seconds. This settings allows users to select a value different from the dnsmasq config option local-ttl This is useful in context of locally used hostnames that are known to stay constant over long times (printers, etc.) Note that large values may render whitelisting ineffective due to client-side caching of blocked queries
* `pi_hole_dns_hosts` - (default: []) - Array of custom DNS records.
  Example:

  ```yaml
  hosts:
    - "127.0.0.1 mylocal"
    - "192.168.0.1 therouter
  ```

  Possible values are Array of custom DNS records each one in HOSTS form: "IP HOSTNAME"

* `pi_hole_dns_domain_needed` - (default: false) - If set, A and AAAA queries for plain names, without dots or domain parts, are never forwarded to upstream nameservers.
* `pi_hole_dns_expand_hosts` - (default: false) - If set, the domain is added to simple names (without a period) in /etc/hosts in the same way as for DHCP-derived names.
* `pi_hole_dns_domain` - (default: "lan") - The DNS domain used by your Pi-hole. This DNS domain is purely local. FTL may answer queries from its local cache and configuration but *never* forwards any requests upstream *unless* you have configured a dns.revServer exactly for this domain. In the latter case, all queries for this domain are sent exclusively to this server (including reverse lookups) For DHCP, this has two effects; firstly it causes the DHCP server to return the domain to any hosts which request it, and secondly it sets the domain which it is legal for DHCP-configured hosts to claim. The intention is to constrain hostnames so that an untrusted host on the LAN cannot advertise its name via DHCP as e.g "google.com" and capture traffic not meant for it. If no domain suffix is specified then any DHCP hostname with a domain part (ie with a period) will be disallowed and logged. If a domain is specified, then hostnames with a domain part are allowed provided the domain part matches the suffix. In addition, when a suffix is set then hostnames without a domain part have the suffix added as an optional domain part For instance, we can set domain=mylab.com and have a machine whose DHCP hostname is "laptop". The IP address for that machine is available both as "laptop" and "laptop.mylab.com" You can disable setting a domain by setting this option to an empty string Possible values are `<any valid domain>`.
* `pi_hole_dns_bogus_priv` - (default: true) - Should all reverse lookups for private IP ranges (i.e., 192.168.x.y, etc) which are not found in /etc/hosts or the DHCP leases file be answered with "no such domain" rather than being forwarded upstream?
* `pi_hole_dns_dnssec` - (default: false) - Validate DNS replies using DNSSEC?
* `pi_hole_dns_interface` - (default: ansible_default_ipv4.interface) - Interface to use for DNS (see also dnsmasq.listening.mode) and DHCP (if enabled). Possible values are: a valid interface name.
* `pi_hole_dns_host_record` - (default: "") - Add A, AAAA and PTR records to the DNS. This adds one or more names to the DNS with associated IPv4 (A) and IPv6 (AAAA) records. Possible values are `<name>[,<name>....],[<IPv4-address>],[<IPv6-address>][,<TTL>]`
* `pi_hole_dns_listening_mode` - (default: "LOCAL") - Pi-hole interface listening modes Possible values are:
  * "LOCAL" Allow only local requests. This setting accepts DNS queries only from hosts whose address is on a local subnet, i.e., a subnet for which an interface exists on the server. It is intended to be set as a default on installation to allow unconfigured installations to be useful but also safe from being used for DNS amplification attacks if (accidentally) running public.
  * "SINGLE" Permit all origins, accept only on the specified interface. Respond only to queries arriving on the specified interface. The loopback (lo) interface is automatically added to the list of interfaces to use when this option is used. Make sure your Pi-hole is properly firewalled
  * "BIND" By default, FTL binds the wildcard address. If this is not what you want, you can use this option as it forces FTL to really bind only the interfaces it is listening on. Note that this may result in issues when the interface may go down (cable unplugged, etc.). About the only time when this is useful is when running another nameserver on the same port on the same machine. This may also happen if you run a virtualization API such as libvirt. When this option is used, IP alias interface labels (e.g. enp2s0:0) are checked rather than interface names
  * "ALL" Permit all origins, accept on all interfaces. Make sure your Pi-hole is properly firewalled! This truly allows any traffic to be replied to and is a dangerous thing to do as your Pi-hole could become an open resolver. You should always ask yourself if the first option doesn't work for you as well
  * "NONE" Do not add any configuration concerning the listening mode to the dnsmasq configuration file. This is useful if you want to manually configure the listening mode in auxiliary configuration files. This option is really meant for advanced users only, support for this option may be limited
* `pi_hole_dns_query_logging` - (default: true) - Log DNS queries and replies to pihole.log
* `pi_hole_dns_cname_records` - (default: []) - List of CNAME records which indicate that `<cname>` is really `<target>`. If the `<TTL>` is given, it overwrites the value of local-ttl Possible values are Array of CNAMEs each on in one of the following forms: `"<cname>,<target>[,<TTL>]"`
* `pi_hole_dns_port` - (default: 53) - Port used by the DNS server
* `pi_hole_dns_rev_servers` - (default: []) - Reverse server (former also called "conditional forwarding") feature. Array of reverse servers each one in one of the following forms `"<enabled>,<ip-address>[/<prefix-len>],<server>[#<port>][,<domain>]"`. Individual components:
  * `<enabled>`: either "true" or "false"
  * `<ip-address>[/<prefix-len>]`: Address range for the reverse server feature in CIDR notation. If the prefix length is omitted, either 32 (IPv4) or 128 (IPv6) are substituted (exact address match). This is almost certainly not what you want here
    Example: "192.168.0.0/24" for the range 192.168.0.1 - 192.168.0.255
  * `<server>[#<port>]`: Target server to be used for the reverse server feature
    Example: "192.168.0.1#53"
  * `<domain>`: Domain used for the reverse server feature (e.g., "fritz.box")
    Example: "fritz.box"

  Possible values are array of reverse servers each one in one of the following forms **"\<enabled\>,\<ip-address\>[/\<prefix-len\>],\<server\>[#\<port\>][,\<domain\>]"**.
  Example:

  ```yaml
  pi_hole_dns_rev_servers:
    - "true,192.168.0.0/24,192.168.0.1,fritz.box"
    - "true,192.168.1.0/24,192.168.0.1,foo.bar"
  ```

* `pi_hole_dns_cache_size` - (default: 10000) - Cache size of the DNS server. Note that expiring cache entries naturally make room for new insertions over time. Setting this number too high will have an adverse effect as not only more space is needed, but also lookup speed gets degraded in the 10,000+ range. dnsmasq may issue a warning when you go beyond 10,000+ cache entries.
* `pi_hole_dns_cache_optimizer` - (default: 3600) - Query cache optimizer: If a DNS name exists in the cache, but its time-to-live has expired only recently, the data will be used anyway (a refreshing from upstream is triggered). This can improve DNS query delays especially over unreliable Internet connections. This feature comes at the expense of possibly sometimes returning out-of-date data and less efficient cache utilization, since old data cannot be flushed when its TTL expires, so the cache becomes mostly least-recently-used. To mitigate issues caused by massively outdated DNS replies, the maximum overaging of cached records is limited. We strongly recommend staying below 86400 (1 day) with this option. Setting the TTL excess time to zero will serve stale cache data regardless how long it has expired. This is not recommended as it may lead to stale data being served for a long time. Setting this option to any negative value will disable this feature altogether.
* `pi_hole_dns_cache_upstream_blocked_ttl` - (default: 86400) - This setting allows you to specify the TTL used for queries blocked upstream. Once the TTL expires, the query will be forwarded to the upstream server again to check if the block is still valid. Defaults to caching for one day (86400 seconds) Setting this value to zero disables caching of queries blocked upstream.
* `pi_hole_dns_blocking_active` - (default: true) - Should FTL block queries?
* `pi_hole_dns_blocking_mode` - (default: "NULL") - How should FTL reply to blocked queries? Possible values are:
  * "NULL" In NULL mode, which is both the default and recommended mode for Pi-hole FTLDNS, blocked queries will be answered with the "unspecified address" (0.0.0.0 or ::). The "unspecified address" is a reserved IP address specified by RFC 3513 - Internet Protocol Version 6 (IPv6) Addressing Architecture section 2.5.2.
  * "IP_NODATA_AAAA" In IP-NODATA-AAAA mode, blocked queries will be answered with the local IPv4 addresses of your Pi-hole. Blocked AAAA queries will be answered with NODATA-IPV6 and clients will only try to reach your Pi-hole over its static IPv4 address
  * "IP" In IP mode, blocked queries will be answered with the local IP addresses of your Pi-hole.
  * "NX" In NXDOMAIN mode, blocked queries will be answered with an empty response (i.e., there won't be an answer section) and status NXDOMAIN. A NXDOMAIN response should indicate that there is no such domain to the client making the query.
  * "NODATA" In NODATA mode, blocked queries will be answered with an empty response (no answer section) and status NODATA. A NODATA response indicates that the domain exists, but there is no record for the requested query type.
* `pi_hole_dns_blocking_edns` - (default: "TEXT) - Should FTL enrich blocked replies with EDNS0 information? Possible values are:
  * "NONE" In NONE mode, no additional EDNS information is added to blocked queries.
  * "CODE" In CODE mode, blocked queries will be enriched with EDNS info-code BLOCKED (15).
  * "TEXT" In TEXT mode, blocked queries will be enriched with EDNS info-code BLOCKED (15) and a text message describing the reason for the block.
* `pi_hole_dns_special_domain_mozilla_canary` - (default: true) - Should Pi-hole always reply with NXDOMAIN to A and AAAA queries of use-application-dns.net to disable Firefox automatic DNS-over-HTTP? This is following the recommendation on <https://support.mozilla.org/en-US/kb/configuring-networks-disable-dns-over-https>
* `pi_hole_dns_special_domain_icloud_private_relay` - (default: true) - Should Pi-hole always reply with NXDOMAIN to A and AAAA queries of mask.icloud.com and mask-h2.icloud.com to disable Apple's iCloud Private Relay to prevent Apple devices from bypassing Pi-hole? This is following the recommendation on <https://developer.apple.com/support/prepare-your-network-for-icloud-private-relay>
* `pi_hole_dns_special_domain_designated_resolver` - (default: true) - Should Pi-hole always reply with NODATA to all queries to zone resolver.arpa to prevent devices from bypassing Pi-hole using Discovery of Designated Resolvers? This is based on recommendations at the end of RFC 9462, section 4
* `pi_hole_dns_reply_host_force4` - (default: false) - Use a specific IPv4 address for the Pi-hole host? By default, FTL determines the address of the interface a query arrived on and uses this address for replying to A queries with the most suitable address for the requesting client. This setting can be used to use a fixed, rather than the dynamically obtained, address when Pi-hole responds to the following names: [ "pi.hole", "\<the device's hostname\>", "pi.hole.\<local domain\>", "\<the device's hostname\>.\<local domain\>" ]
* `pi_hole_dns_reply_host_ipv4` - (default: "") - Custom IPv4 address for the Pi-hole host Possible values are: \<valid IPv4 address\> or empty string ("")
* `pi_hole_dns_reply_host_force4` - (default: false) - Use a specific IPv6 address for the Pi-hole host? See description for the IPv4 variant above for further details.
* `pi_hole_dns_reply_host_ipv6` - (default: "") - Custom IPv6 address for the Pi-hole host Possible values are: \<valid IPv6 address\> or empty string ("")
* `pi_hole_dns_blocking_force4` - (default: false) - Use a specific IPv4 address in IP blocking mode? By default, FTL determines the address of the interface a query arrived on and uses this address for replying to A queries with the most suitable address for the requesting client. This setting can be used to use a fixed, rather than the dynamically obtained, address when Pi-hole responds in the following cases: IP blocking mode is used and this query is to be blocked, regular expressions with the ;reply=IP regex extension.
* `pi_hole_dns_blocking_ipv4` - (default: "") - Custom IPv4 address for IP blocking mode Possible values are: \<valid IPv4 address\> or empty string ("")
* `pi_hole_dns_blocking_force4` - (default: false) - Use a specific IPv6 address in IP blocking mode? See description for the IPv4 variant above for further details.
* `pi_hole_dns_blocking_ipv6` - (default: "") - Custom IPv6 address for IP blocking mode Possible values are: \<valid IPv6 address\> or empty string ("")
* `pi_hole_dns_rate_limit_count` - (default: 1000) - Rate-limited queries are answered with a REFUSED reply and not further processed by FTL. The default settings for FTL's rate-limiting are to permit no more than 1000 queries in 60 seconds. Both numbers can be customized independently. It is important to note that rate-limiting is happening on a per-client basis. Other clients can continue to use FTL while rate-limited clients are short-circuited at the same time. For this setting, both numbers, the maximum number of queries within a given time and the length of the time interval (seconds) have to be specified. For instance, if you want to set a rate limit of 1 query per hour, the option should look like RATE_LIMIT=1/3600. The time interval is relative to when FTL has finished starting (start of the daemon + possible delay by DELAY_STARTUP) then it will advance in steps of the rate-limiting interval. If a client reaches the maximum number of queries it will be blocked until the end of the current interval. This will be logged to /var/log/pihole/FTL.log, e.g. Rate-limiting 10.0.1.39 for at least 44 seconds. If the client continues to send queries while being blocked already and this number of queries during the blocking exceeds the limit the client will continue to be blocked until the end of the next interval (FTL.log will contain lines like Still rate-limiting 10.0.1.39 as it made additional 5007 queries). As soon as the client requests less than the set limit, it will be unblocked (Ending rate-limitation of 10.0.1.39). Rate-limiting may be disabled altogether by setting both values to zero (this results in the same behavior as before FTL v5.7). How many queries are permitted
* `pi_hole_dns_rate_limit_interval` - (default: 60) - ... in the set interval before rate-limiting?

## Define pi-hole dhcp configuration

* `pi_hole_dhcp_active` - (default: false) - Is the embedded DHCP server enabled?
* `pi_hole_dhcp_start` - (default: "") - Start address of the DHCP address pool. Possible values are: \<valid IPv4 address\> or empty string (""), e.g., "192.168.0.10"
* `pi_hole_dhcp_end` - (default: "") - End address of the DHCP address pool. Possible values are \<valid IPv4 address\> or empty string (""), e.g., "192.168.0.250"
* `pi_hole_dhcp_router` - (default: "") - Address of the gateway to be used (typically the address of your router in a home installation). Possible values are \<valid IPv4 address\> or empty string (""), e.g., "192.168.0.1"
* `pi_hole_dhcp_netmasq` - (default: "") - The netmask used by your Pi-hole. For directly connected networks (i.e., networks on which the machine running Pi-hole has an interface) the netmask is optional and may be set to an empty string (""): it will then be determined from the interface configuration itself. For networks which receive DHCP service via a relay agent, we cannot determine the netmask itself, so it should explicitly be specified, otherwise Pi-hole guesses based on the class (A, B or C) of the network address Possible values are \<any valid netmask\> (e.g., "255.255.255.0") or empty string ("") for auto-discovery.
* `pi_hole_dhcp_leasetime` - (default: "24h") - If the lease time is given, then leases will be given for that length of time. If not given, the default lease time is one hour for IPv4 and one day for IPv6 Possible values are: The lease time can be in seconds, or minutes (e.g., "45m") or hours (e.g., "1h") or days (like "2d") or even weeks ("1w"). You may also use "infinite" as string but be aware of the drawbacks.
* `pi_hole_dhcp_ipv6` - (default: false) - Should Pi-hole make an attempt to also satisfy IPv6 address requests (be aware that IPv6 works a whole lot different than IPv4).
* `pi_hole_dhcp_rapid_commit` - (default: false) - Enable DHCPv4 Rapid Commit Option specified in RFC 4039. Should only be enabled if either the server is the only server for the subnet to avoid conflicts.
* `pi_hole_dhcp_multidns` - (default: false) - Advertise DNS server multiple times to clients. Some devices will add their own proprietary DNS servers to the list of DNS servers, which can cause issues with Pi-hole. This option will advertise the Pi-hole DNS server multiple times to clients, which should prevent this from happening.
* `pi_hole_dhcp_logging` - (default: false) - Enable logging for DHCP. This will log all relevant DHCP-related activity, including, e.g., all the options sent to DHCP clients and the tags used to determine them (if any). This can be useful for debugging DHCP issues. The generated output is saved to the file specified by files.log.dnsmasq below.
* `pi_hole_dhcp_ignoreunknownclients` - (default: false) - Ignore unknown DHCP clients. If this option is set, Pi-hole ignores all clients which are not explicitly configured through dhcp.hosts. This can be useful to prevent unauthorized clients from getting an IP address from the DHCP server. It should be noted that this option is not a security feature, as clients can still assign themselves an IP address and use the network. It is merely a convenience feature to prevent unknown clients from getting a valid IP configuration assigned automatically. Note that you will need to configure new clients manually in dhcp.hosts before they can use the network when this feature is enabled.
* `pi_hole_dhcp_hosts` - (default: []) - Per host parameters for the DHCP server. This allows a machine with a particular hardware address to be always allocated the same hostname, IP address and lease time or to specify static DHCP leases. Possible values are: Array of static leases each on in one of the following forms: **"[\<hwaddr\>][,id:\<client_id\>|*][,set:\<tag\>][,tag:\<tag\>][,\<ipaddr\>][,\<hostname\>][,\<lease_time\>][,ignore]"**. Example:

```yaml
pi_hole_dhcp_host:
  - 00:11:22:33:44:55,192.168.1.1,foo
```

* `pi_hole_dhcp_ntp_ipv4_active` - (default: true) - Should FTL act as network time protocol (NTP) server (IPv4)?
* `pi_hole_dhcp_ntp_ipv4_address` - (default: "") - IPv4 address to listen on for NTP requests. Possible values are \<valid IPv4 address\> or empty string ("") for wildcard (0.0.0.0).
* `pi_hole_dhcp_ntp_ipv6_active` - (default: true) - Should FTL act as network time protocol (NTP) server (IPv6)?
* `pi_hole_dhcp_ntp_ipv6_address` - (default: "") - IPv6 address to listen on for NTP requests. Possible values are \<valid IPv4 address\> or empty string ("") for wildcard (::).
* `pi_hole_dhcp_ntp_sync_active` - (default: true) - Should FTL try to synchronize the system time with an upstream NTP server?
* `pi_hole_dhcp_ntp_sync_server` - (default: "pool.ntp.org") - NTP upstream server to sync with, e.g., "pool.ntp.org". Note that the NTP server should be located as close as possible to you in order to minimize the time offset possibly introduced by different routing paths. Possible values are: valid NTP upstream server.
* `pi_hole_dhcp_ntp_sync_interval` - (default: 3600) - Interval in seconds between successive synchronization attempts with the NTP server.
* `pi_hole_dhcp_ntp_sync_count` - (default: 8) - Number of NTP syncs to perform and average before updating the system time.
* `pi_hole_dhcp_ntp_sync_rtc_set` - (default: false) - Should FTL update a real-time clock (RTC) if available?
* `pi_hole_dhcp_ntp_sync_rtc_device` - (default: "") - Path to the RTC device to update. Leave empty for auto-discovery. Possible values are: Path to the RTC device, e.g., "/dev/rtc0"
* `pi_hole_dhcp_ntp_sync_rtc_utc` - (default: true) - Should the RTC be set to UTC?
