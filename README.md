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

## Define pi-hole resolver configuration

* `pi_hole_resolver_resolveipv4` - (default: true) - Should FTL try to resolve IPv4 addresses to hostnames?
* `pi_hole_resolver_resolveipv6` - (default: true) - Should FTL try to resolve IPv6 addresses to hostnames?
* `pi_hole_resolver_networknames` - (default: true) - Control whether FTL should use the fallback option to try to obtain client names from checking the network table. This behavior can be disabled with this option. Assume an IPv6 client without a host names. However, the network table knows - though the client's MAC address - that this is the same device where we have a host name for another IP address (e.g., a DHCP server managed IPv4 address). In this case, we use the host name associated to the other address as this is the same device.
* `pi_hole_resolver_refreshnames` - (default: "IPV4_ONLY") - With this option, you can change how (and if) hourly PTR requests are made to check for changes in client and upstream server hostnames. Possible values are:
  * "IPV4_ONLY" Do hourly PTR lookups only for IPv4 addresses. This is the new default since Pi-hole FTL v5.3.2. It should resolve issues with more and more very short-lived PE IPv6 addresses coming up in a lot of networks
  * "ALL" Do hourly PTR lookups for all addresses. This was the default until FTL v5.3(.1). It has been replaced as it can create a lot of PTR queries for those with many IPv6 addresses in their networks.
  * "UNKNOWN" Only resolve unknown hostnames. Already existing hostnames are never refreshed i.e., there will be no PTR queries made for clients where hostnames are known. This also means that known hostnames will not be updated once known
  * "NONE" Don't do any hourly PTR lookups. This means we look host names up exactly once (when we first see a client) and never again. You may miss future changes of host names.

## Define pi-hole database configuration

* `pi_hole_database_dbimport` - (default: true) - Should FTL load information from the database on startup to be aware of the most recent history?
* `pi_hole_database_maxdbdays` - (default: 91) - How long should queries be stored in the database [days]? Setting this value to 0 will disable the database.
* `pi_hole_database_dbinterval` - (default: 60) - How often do we store queries in FTL's database [seconds]?
* `pi_hole_database_usewal` - (default: true) - Should FTL enable Write-Ahead Log (WAL) mode for the on-disk query database (configured via files.database)? It is recommended to leave this setting enabled for performance reasons. About the only reason to disable WAL mode is if you are experiencing specific issues with it e.g., when using a database that is accessed from multiple hosts via a network share. When this setting is disabled, FTL will use SQLite3's default journal mode (rollback journal in DELETE mode).
* `pi_hole_database_network_parsearpcache` - (default: true) - Should FTL analyze the local ARP cache? When disabled, client identification and the network table will stop working reliably.
* `pi_hole_database_network_expire` - (default: 91) - How long should IP addresses be kept in the network_addresses table [days]? IP addresses (and associated host names) older than the specified number of days are removed to avoid dead entries in the network overview table.

## Define pi-hole webserver configuration

* `pi_hole_webserver_domain` - (default: "pi.hole") - On which domain is the web interface served? Possible values are \<valid domain\>
* `pi_hole_webserver_acl` - (default: "") -Webserver access control list (ACL) allowing for restrictions to be put on the list of IP addresses which have access to the web server. The ACL is a comma separated list of IP subnets, where each subnet is prepended by either a - or a + sign. A plus sign means allow, where a minus sign means deny. If a subnet mask is omitted, such as -1.2.3.4, this means to deny only that single IP address. If this value is not set (empty string), all accesses are allowed. Otherwise, the default setting is to deny all accesses. On each request the full list is traversed, and the last (!) match wins. IPv6 addresses may be specified in CIDR-form [a:b::c]/64. Possible values are \<valid ACL\>. Examples:

```yaml
pi_hole_webserver_acl: "+127.0.0.1,+[::1],-192.168.0.0/16"
```

* `pi_hole_webserver_port` - (default: "80o,443os,[::]:80o,[::]:443os") - Ports to be used by the webserver. Comma-separated list of ports to listen on. It is possible to specify an IP address to bind to. In this case, an IP address and a colon must be prepended to the port number. For example, to bind to the loopback interface on port 80 (IPv4) and to all interfaces port 8080 (IPv4), use "127.0.0.1:80,8080". "[::]:80" can be used to listen to IPv6 connections to port 80. IPv6 addresses of network interfaces can be specified as well, e.g. "[::1]:80" for the IPv6 loopback interface. [::]:80 will bind to port 80 IPv6 only. In order to use port 80 for all interfaces, both IPv4 and IPv6, use either the configuration "80,[::]:80" (create one socket for IPv4 and one for IPv6 only), or "+80" (create one socket for both, IPv4 and IPv6). The '+' notation to use IPv4 and IPv6 will only work if no network interface is specified. Depending on your operating system version and IPv6 network environment, some configurations might not work as expected, so you have to test to find the configuration most suitable for your needs. In case "+80" does not work for your environment, you need to use "80,[::]:80". If the port is TLS/SSL, a letter 's' (secure) must be appended, for example "80,443s" will open port 80 and port 443, and connections on port 443 will be encrypted. For non-encrypted ports, it is allowed to append letter 'r' (as in redirect). Redirected ports will redirect all their traffic to the first configured SSL port. For example, if webserver.port is "80r,443s", then all HTTP traffic coming at port 80 will be redirected to HTTPS port 443. When specifying 'o' (optional) behind a port, inability to use this port is not considered an error. For instance, specifying "80o,8080o" will allow the webserver to listen on either 80, 8080, both or even none of the two ports. This flag may be combined with 'r' and 's' like "80or,443os,8080,4443s" (80 redirecting to SSL if available, 443 encrypted if available, 8080 mandatory and unencrypted, 4443 mandatory and encrypted). If this value is not set (empty string), the web server will not be started and hence, the API will not be available. Possible values are comma-separated list of \<[ip_address:]port\>.
* `pi_hole_webserver_threads` - (default: 50) -Maximum number of worker threads allowed. The Pi-hole web server handles each incoming connection in a separate thread. Therefore, the value of this option is effectively the number of concurrent HTTP connections that can be handled. Any other connections are queued until they can be processed by a unoccupied thread The total number of threads you see may be lower than the configured value as threads are only created when needed due to incoming connections. The value 0 means the number of threads is 50 (as per default settings of CivetWeb) for backwards-compatible behavior.
* `pi_hole_webserver_headers` - (default: [ "X-DNS-Prefetch-Control: off", "Content-Security-Policy: default-src 'self' 'unsafe-inline';", "X-Frame-Options: DENY", "X-XSS-Protection: 0", "X-Content-Type-Options: nosniff", "Referrer-Policy: strict-origin-when-cross-origin" ]) - Additional HTTP headers added to the web server responses. The headers are added to all responses, including those for the API. Note about the default additional headers:
  * X-DNS-Prefetch-Control: off: Usually browsers proactively perform domain name resolution on links that the user may choose to follow. We disable DNS prefetching here.
  * Content-Security-Policy: [...] 'unsafe-inline' is both required by Chart.js styling some elements directly, and index.html containing some inlined Javascript code.
  * X-Frame-Options: DENY: The page can not be displayed in a frame, regardless of the site attempting to do so
  * X-Xss-Protection: 0: Disables XSS filtering in browsers that support it. This header is usually enabled by default in browsers, and is not recommended as it can hurt the security of the site [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection]
  * X-Content-Type-Options: nosniff: Marker used by the server to indicate that the MIME types advertised in the  Content-Type headers should not be changed and be followed. This allows to opt-out of MIME type sniffing, or, in other words, it is a way to say that the webmasters knew what they were doing. Site security testers usually expect this header to be set.
  * Referrer-Policy: strict-origin-when-cross-origin: A referrer will be sent for same-site origins, but cross-origin requests will send no referrer information.

  The latter four headers are set as expected by <https://securityheaders.io>. Possible values are array of HTTP headers.

* `pi_hole_webserver_serve_all` - (default: false) - Should the web server serve all files in webserver.paths.webroot directory? If disabled, only files within the path defined through webserver.paths.webhome and /api will be served.
* `pi_hole_webserver_session_timeout` - (default: 1800) - Session timeout in seconds. If a session is inactive for more than this time, it will be terminated. Sessions are continuously refreshed by the web interface, preventing sessions from timing out while the web interface is open. This option may also be used to make logins persistent for long times, e.g. 86400 seconds (24 hours), 604800 seconds (7 days) or 2592000 seconds (30 days). Note that the total number of concurrent sessions is limited so setting this value too high may result in users being rejected and unable to log in if there are already too many sessions active.
* `pi_hole_webserver_session_restore` - (default: true) - Should Pi-hole backup and restore sessions from the database? This is useful if you want to keep your sessions after a restart of the web interface.
* `pi_hole_webserver_tls_cert` - (default: "/etc/pihole/tls.pem") - Path to the TLS (SSL) certificate file. All directories along the path must be readable and accessible by the user running FTL (typically 'pihole'). This option is only required when at least one of webserver.port is TLS. The file must be in PEM format, and it must have both, private key and certificate (the *.pem file created must contain a 'CERTIFICATE' section as well as a 'RSA PRIVATE KEY' section). The \*.pem file can be created using cp server.crt server.pem cat server.key >> server.pem if you have these files instead. Possible values are \<valid TLS certificate file (*.pem)\>
* `pi_hole_webserver_paths_webroot` - (default: "/var/www/html") - Server root on the host. Possible values are \<valid path\>
* `pi_hole_webserver_paths_webhome` - (default: "/admin/") - Sub-directory of the root containing the web interface. Possible values are \<valid subpath\>, both slashes are needed.
* `pi_hole_webserver_paths_prefix` - (default: "") - Prefix where the web interface is served. This is useful when you are using a reverse proxy serving the web interface, e.g at http://\<ip\>/pihole/admin/ instead of http://\<ip\>/admin/. In this example, the prefix would be "/pihole". Note that the prefix has to be stripped away by the reverse proxy, e.g., for traefik:
  * traefik.http.routers.pihole.rule=PathPrefix(`/pihole`)
  * traefik.http.middlewares.piholehttp.stripprefix.prefixes=/pihole

  The prefix should start with a slash. If you don't use a prefix, leave this field empty. Setting this field to an incorrect value may result in the web interface not being accessible. Don't use this setting if you are not using a reverse proxy. Possible values are valid URL prefix or empty.
* `pi_hole_webserver_boxed` - (default: true) -Should the web interface use the boxed layout?
* `pi_hole_webserver_interface` - (default: "default-auto") - Theme used by the Pi-hole web interface. Possible values are:
  * "default-auto". Pi-hole auto
  * "default-light". Pi-hole day
  * "default-dark". Pi-hole midnight
  * "default-darker". Pi-hole deep-midnight
  * "high-contrast". High-contrast light
  * "high-contrast-dark". High-contrast dark
  * "lcars". Star Trek LCARS
* `pi_hole_webserver_api_max_sessions` - (default: 16) - Number of concurrent sessions allowed for the API. If the number of sessions exceeds this value, no new sessions will be allowed until the number of sessions drops due to session expiration or logout. Note that the number of concurrent sessions is irrelevant if authentication is disabled as no sessions are used in this case.
* `pi_hole_webserver_api_prettyjson` - (default: false) -Should FTL prettify the API output (add extra spaces, newlines and indentation)?
* `pi_hole_webserver_api_pwhash` - (default: "") - API password hash. Possible values are \<valid Pi-hole password hash\>
* `pi_hole_webserver_api_totp_secret` - (default: "") - Pi-hole 2FA TOTP secret. When set to something different than "", 2FA authentication will be enforced for the API and the web interface. This setting is write-only, you can not read the secret back. Possible values are \<valid TOTP secret (20 Bytes in Base32 encoding)\>
* `pi_hole_webserver_api_app_pwhash` - (default: "") - Pi-hole application password. After you turn on two-factor (2FA) verification and set up an Authenticator app, you may run into issues if you use apps or other services that don't support two-step verification. In this case, you can create and use an app password to sign in. An app password is a long, randomly generated password that can be used instead of your regular password + TOTP token when signing in to the API. The app password can be generated through the API and will be shown only once. You can revoke the app password at any time. If you revoke the app password, be sure to generate a new one and update your app with the new password. Possible values are \<valid Pi-hole password hash\>
* `pi_hole_webserver_api_app_sudo` - (default: false) - Should application password API sessions be allowed to modify config settings? Setting this to true allows third-party applications using the application password to modify settings, e.g., the upstream DNS servers, DHCP server settings, or changing passwords. This setting should only be enabled if really needed and only if you trust the applications using the application password.
* `pi_hole_webserver_api_cli_pw` - (default: true) - Should FTL create a temporary CLI password? This password is stored in clear in /etc/pihole and can be used by the CLI (pihole ...  commands) to authenticate against the API. Note that the password is only valid for the current session and regenerated on each FTL restart. Sessions initiated with this password cannot modify the Pi-hole configuration (change passwords, etc.) for security reasons but can still use the API to query data and manage lists.
* `pi_hole_webserver_api_excludeclients` - (default: []) - Array of clients to be excluded from certain API responses (regex):
  * Query Log (/api/queries)
  * Top Clients (/api/stats/top_clients)

  This setting accepts both IP addresses (IPv4 and IPv6) as well as hostnames. Note that backslashes "\" need to be escaped, i.e. "\\" in this setting. Possible values are array of regular expressions describing clients. Example:

  ```yaml
  pi_hole_webserver_api_excludeclients:
    - "^192\\.168\\.2\\.56$"
    - "^fe80::341:[0-9a-f]*$"
    - "^localhost$"
  ```

* `pi_hole_webserver_api_excludedomains` - (default: []) - Array of domains to be excluded from certain API responses (regex):
  * Query Log (/api/queries)
  * Top Clients (/api/stats/top_domains)

  Note that backslashes "\" need to be escaped, i.e. "\\" in this . Possible values are array of regular expressions describing domains. Example:

  ```yaml
  pi_hole_webserver_api_excludedomains:
    - "(^|\\.)\\.google\\.de$"
    - "\\.pi-hole\\.net$"
  ```

* `pi_hole_webserver_api_maxhistory` - (default: 86400) - How much history should be imported from the database and returned by the API [seconds]? (max 24*60*60 = 86400).
* `pi_hole_webserver_api_maxclients` - (default: 10) - Up to how many clients should be returned in the activity graph endpoint (/api/history/clients)? This setting can be overwritten at run-time using the parameter N. Setting this to 0 will always send all clients. Be aware that this may be challenging for the GUI if you have many (think > 1.000 clients) in your network.
* `pi_hole_webserver_api_client_history_global_max` - (default: true) - How should the API compute the most active clients? If set to true, the API will return the clients with the most queries globally (within 24 hours). If set to false, the API will return the clients with the most queries per time slot individually.
* `pi_hole_webserver_api_allow_destructive` - (default: true) - Allow destructive API calls (e.g. deleting all queries, powering off the system, ...)
* `pi_hole_webserver_api_temp_limit` - (default: 60.000000) - Which upper temperature limit should be used by Pi-hole? Temperatures above this limit will be shown as "hot". The number specified here is in the unit defined below.
* `pi_hole_webserver_api_temp_unit` - (default: "C") - Which temperature unit should be used for temperatures processed by FTL? Possible values are:
  * "C" Celsius
  * "F" Fahrenheit
  * "K" Kelvin

## Define pi-hole files configuration

* `pi_hole_files_pid` - (default: "/run/pihole-FTL.pid") - The file which contains the PID of FTL's main process. Possible values are \<any writable file\>
* `pi_hole_files_database` - (default: "/etc/pihole/pihole-FTL.db") - The location of FTL's long-term database. Possible values are \<any FTL database\>
* `pi_hole_files_gravity` - (default:"/etc/pihole/gravity.db") -The location of Pi-hole's gravity database. Possible values are \<any Pi-hole gravity database\>
* `pi_hole_files_gravity_tmp` - (default: "/tmp") - A temporary directory where Pi-hole can store files during gravity updates. This directory must be writable by the user running gravity (typically pihole). Possible values are \<any existing world-writable writable directory\>
* `pi_hole_files_macvendor` - (default: "/etc/pihole/macvendor.db") - The database containing MAC -> Vendor information for the network table. Possible values are \<any Pi-hole macvendor database\>
* `pi_hole_files_setupVars` - (default: "/etc/pihole/setupVars.conf") - The old config file of Pi-hole used before v6.0. Possible values are \<any Pi-hole setupVars file\>
* `pi_hole_files_pcap` - (default: "") - An optional file containing a pcap capture of the network traffic. This file is used for debugging purposes only. If you don't know what this is, you don't need it. Setting this to an empty string disables pcap recording. The file must be writable by the user running FTL (typically pihole). Failure to write to this file will prevent the DNS resolver from starting. The file is appended to if it already exists. Possible values are \<any writable pcap file\>
* `pi_hole_files_log_ftl` - (default: "/var/log/pihole/FTL.log") - # The location of FTL's log file. Possible values are \<any writable file\>
* `pi_hole_files_log_dnsmasq` - (default: "/var/log/pihole/pihole.log") - The log file used by the embedded dnsmasq DNS server. Possible values are \<any writable file\>
* `pi_hole_files_log_webserver` - (default: "/var/log/pihole/webserver.log") - The log file used by the webserver. Possible values are \<any writable file\>

## Define pi-hole misc configuration

* `pi_hole_misc_privacylevel` - (default: 0) -  Using privacy levels you can specify which level of detail you want to see in your Pi-hole statistics. Changing this setting will trigger a restart of FTL. Possible values are:
  * 0 Don't hide anything, all statistics are available
  * 1 Hide domains. This setting disables Top Domains and Top Ads
  * 2 Hide domains and clients. This setting disables Top Domains, Top Ads, Top Clients and Clients over time
  * 3 Anonymize everything. This setting disabled almost any statistics and query analysis. There will be no long-term database logging and no Query Log. You will also loose most regex features.
* `pi_hole_misc_delay_startup` - (default: 0) - During startup, in some configurations, network interfaces appear only late during system startup and are not ready when FTL tries to bind to them. Therefore, you may want FTL to wait a given amount of time before trying to start the DNS revolver. This setting takes any integer value between 0 and 300 seconds. To prevent delayed startup while the system is already running and FTL is restarted, the delay only takes place within the first 180 seconds (hard-coded) after booting.
* `pi_hole_misc_nice` - (default: -10) - Set niceness of pihole-FTL. Defaults to -10 and can be disabled altogether by setting a value of -999. The nice value is an attribute that can be used to influence the CPU scheduler to favor or disfavor a process in scheduling decisions. The range of the nice value varies across UNIX systems. On modern Linux, the range is -20 (high priority = not very nice to other processes) to +19 (low priority).
* `pi_hole_misc_addr2line` - (default: true) - Should FTL translate its own stack addresses into code lines during the bug backtrace? This improves the analysis of crashed significantly. It is recommended to leave the option enabled. This option should only be disabled when addr2line is known to not be working correctly on the machine because, in this case, the malfunctioning addr2line can prevent from generating any backtrace at all.
* `pi_hole_misc_etc_dnsmasq_d` - (default: false) - Should FTL load additional dnsmasq configuration files from /etc/dnsmasq.d/? Warning: This is an advanced setting and should only be used with care. Incorrectly formatted or config files specifying options which can only be defined once can result in conflicts with the automatic configuration of Pi-hole (see /etc/pihole/dnsmasq.conf) and may stop DNS resolution from working.
* `pi_hole_misc_dnsmasq_lines` - (default: []) - Additional lines to inject into the generated dnsmasq configuration. Warning: This is an advanced setting and should only be used with care. Incorrectly formatted or duplicated lines as well as lines conflicting with the automatic configuration of Pi-hole can break the embedded dnsmasq and will stop DNS resolution from working. Use this option with extra care. Possible values are array of valid dnsmasq config line options.
* `pi_hole_misc_extralogging` - (default: false) - Log additional information about queries and replies to pihole.log. When this setting is enabled, the log has extra information at the start of each line. This consists of a serial number which ties together the log lines associated with an individual query, and the IP address of the requestor. This setting is only effective if dns.queryLogging is enabled, too. This option is only useful for debugging and is not recommended for normal use.
* `pi_hole_misc_readonly` - (default: false) - Put configuration into read-only mode. This will prevent any changes to the configuration file via the API or CLI. This setting useful when a configuration is to be forced/modified by some third-party application (like infrastructure-as-code providers) and should not be changed by any means.
* `pi_hole_misc_check_load` - (default: true) -  Pi-hole is very lightweight on resources. Nevertheless, this does not mean that you should run Pi-hole on a server that is otherwise extremely busy as queuing on the system can lead to unnecessary delays in DNS operation as the system becomes less and less usable as the system load increases because all resources are permanently in use. To account for this, FTL regularly checks the system load. To bring this to your attention, FTL warns about excessive load when the 15 minute system load average exceeds the number of cores. This check can be disabled with this setting
* `pi_hole_misc_check_load_shmem` - (default: 90) - FTL stores history in shared memory to allow inter-process communication with forked dedicated TCP workers. If FTL runs out of memory, it cannot continue to work as queries cannot be analyzed any further. Hence, FTL checks if enough shared memory is available on your system and warns you if this is not the case. By default, FTL warns if the shared-memory usage exceeds 90%. You can set any integer limit between 0 to 100 (interpreted as percentages) where 0 means that checking of shared-memory usage is disabled.
* `pi_hole_misc_check_disk` - (default: 90) - FTL stores its long-term history in a database file on disk. Furthermore, FTL stores log files. By default, FTL warns if usage of the disk holding any crucial file exceeds 90%. You can set any integer limit between 0 to 100 (interpreted as percentages) where 0 means that checking of disk usage is disabled.
