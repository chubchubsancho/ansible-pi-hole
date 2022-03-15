# ansible-pi-hole

[![Build Status](https://travis-ci.com/chubchubsancho/ansible-pi-hole.svg?branch=master)](https://travis-ci.com/chubchubsancho/ansible-pi-hole)
[![License](https://img.shields.io/badge/license-MIT-blue.svg?logo=github&style=flat)](https://raw.githubusercontent.com/chubchubsancho/ansible-pi-hole/master/LICENSE)

An ansible role to install [Pi-Hole](https://pi-hole.net/)

## Role configuration

* `pi_hole_repo` (default: "https://github.com/pi-hole/pi-hole.git") - define `pi-hole` repository
* `pi_hole_prerequisites` (default: "git") - define `pi-hole` prerequisites

## Define pi-hole configuration

Add `pi-hole` variable for config. These vars can be omited

* `pi_hole_version` - (default: "HEAD") - define branch/tag/commit of `pi-hole` repository which will be cloned
* `pi_hole_download_dir` - (default: "/home/pihole") - define base directory for clone `pi-hole`
* `pi_hole_install_dir` - (default: "pi-hole") - define directory where to clone `pi-hole`
* `pi_hole_query_logging` - (default: true) - define if `pi-hole` will log query
* `pi_hole_install_web_server` - (default: true) - define if `pi-hole` web server need to be installed
* `pi_hole_install_web_interface` - (default: true) - define if `pi-hole` web interface need to be installed
* `pi_hole_enable_lighttpd` - (default: true) - does `lighttpd` need to be enabled
* `pi_hole_webpasswd` - (default: omit) - define `pi-hole` web password if omited one will be generated
* `pi_hole_dnsmasq_listening` - (default: "single") -
* `pi_hole_dns_1` - (default: "8.8.8.8") - `pi-hole` upstream dns server
* `pi_hole_dns_1_port` - (default: "53") - `pi-hole` upstream dns server port
* `pi_hole_dns_2` - (default: "8.8.4.4") - `pi-hole` upstream dns server
* `pi_hole_dns_1_port` - (default: "53") - `pi-hole` upstream dns server port
* `pi_hole_fqdn_required` - (default: false) - does `pi-hole` required fqdn queries
* `pi_hole_bogus_priv` - (default: false) - does `pi-hole` will aswer bogus on private zone
* `pi_hole_dnssec` - (default: false) - `pi-hole` use dnssec
* `pi_hole_rev_server` - (default: true) - does `pi-hole` answer to reverse queries
* `pi_hole_rev_server_domain` - (default: "") - `pi-hole` reverse domain name
* `pi_hole_rev_server_target` - (default: "") - `pi-hole` reverse server target
* `pi_hole_rev_server_cidr` - (default: "") - `pi-hole` reverse cidr
* `pi_hole_blocking_enabled` - (default: true) - enable `pi-hole` blocking

## Define pi-hole dhcp configuration

* `pi_hole_dhcp_active` - (default: false) - enable `pi-hole` dhcp server
* `pi_hole_dhcp_start` - (default: "") - first ip address to hand out
* `pi_hole_dhcp_end` - (default: "") - last ip address to hand out
* `pi_hole_dhcp_router` - (default: "") - gateway ip address
* `pi_hole_dhcp_leasetime` - (default: "24") - dhcp lease time
* `pi_hole_domain` - (default: "lan") - `pi-hole` domain  name
* `pi_hole_dhcp_ipv6` - (default: false) - enable IPv6 support
* `pi_hole_dhcp_rapid_commit` - (default: false) - enable dhcp v4 rapid commit

## Define pi-hole local DNS configuration
* `pi_hole_local_dns_records` - (default: "") - define `pi-hole` local DNS records. Expects a list of dictionnaries in which each items contains a key `ip` and `name`.

Example:
```yaml
vars:
  pi_hole_local_dns_records:
    - name: db.lan
      ip: 10.0.13.37
    - name: web.lan
      ip: 10.0.13.38
```

## Define pi-hole FTL configuration

Add `pi-hole-FTL` variable config. These vars can be omited

* `pi_hole_ftl_blockingmode` - (default: "NULL") - How should FTL reply to blocked queries?
* `pi_hole_ftl_cname_deep_inspect` - (default: true) - Use this option to disable deep CNAME inspection. This might be beneficial for very low-end devices
* `pi_hole_ftl_block_esni` - (default: true) - Encrypted Server Name Indication (ESNI) is certainly a good step into the right direction to enhance privacy on the web. It prevents on-path observers, including ISPs, coffee shop owners and firewalls, from intercepting the TLS Server Name Indication (SNI) extension by encrypting it. This prevents the SNI from being used to determine which websites users are visiting.
* `pi_hole_ftl_maxlogage` - (default: "24.0") - Up to how many hours of queries should be imported from the database and logs?
* `pi_hole_ftl_privacylevel` - (default: "0") - Which privacy level is used?
* `pi_hole_ftl_ignore_localhost` - (default: "no") - Should FTL ignore queries coming from the local machine?
* `pi_hole_ftl_query_analysis` - (default: "yes) - Should FTL analyze AAAA queries? The DNS server will handle AAAA queries the same way, reglardless of this setting. All this does is ignoring AAAA queries when computing the statistics of Pi-hole. This setting is considered obsolete and may be removed in a future version.
* `pi_hole_ftl_analyze_only_a_and_aaaa` - (default: false) - Should FTL only analyze A and AAAA queries?
* `pi_hole_ftl_socket_listening` - (default: "localonly") - Listen only for local socket connections or permit all connections
* `pi_hole_ftl_port` - (default: "4711") - On which port should FTL be listening?
* `pi_hole_ftl_resolve_ipv6` - (default: "yes") - Should FTL try to resolve IPv6 addresses to hostnames?
* `pi_hole_ftl_resolve_ipv4` - (default: "yes") - Should FTL try to resolve IPv4 addresses to hostnames?
* `pi_hole_ftl_delay_startup` - (default: "0") - In certain configurations, you may want FTL to wait a given amount of time before trying to start the DNS revolver. This is typically found when network interfaces appear only late during system startup and the interface startup priorities are configured incorrectly. This setting takes any integer value between 0 and 300 seconds
* `pi_hole_ftl_nice` - (default: "-10") - Can be used to change the niceness of Pi-hole FTL. Defaults to -10 and can be disabled altogether by setting a value of -999.
* `pi_hole_ftl_names_from_netdb` - (default: true) - Control whether FTL should use the fallback option to try to obtain client names from checking the network table. This behavior can be disabled with this option
* `pi_hole_refresh_hostnames` - (default: "IPV4") - With this option, you can change how (and if) hourly PTR requests are made to check for changes in client and upstream server hostnames
* `pi_hole_ftl_dbimport` - (default: "true") - Should FTL load information from the database on startup to be aware of the most recent history?
* `pi_hole_ftl_maxdbdays` - (default: "365") - How long should queries be stored in the database? Setting this to 0 disables the database
* `pi_hole_ftl_dbinterval` - (default: "1.0") - How often do we store queries in FTL's database [minutes]?
* `pi_hole_ftl_dbfile` - (default: "/etc/pihole/pihole-FTL.db") - Specify the path and filename of FTL's SQLite3 long-term database. Setting this to DBFILE= disables the database altogether
* `pi_hole_ftl_logfile` - (default: "/var/log/pihole-FTL.log") - The location of FTL's log file. If you want to move the log file to a different place
* `pi_hole_ftl_pidfile` - (default: "/run/pihole-FTL.pid") - The file which contains the PID of FTL's main process.
* `pi_hole_ftl_portfile` - (default: "/run/pihole-FTL.port") - The file containing the port FTL's API is listening on.
* `pi_hole_ftl_socket_file` - (default: "/run/pihole/FTL.sock") - The file containing the socket FTL's API is listening on.
* `pi_hole_ftl_setupvarsfile` - (default: "/etc/pihole/setupVars.conf") - The config file of Pi-hole containing, e.g., the current blocking status (do not change).
* `pi_hole_ftl_macvendordb` - (default: "/etc/pihole/macvendor.db") - The database containing MAC -> Vendor information for the network table.
* `pi_hole_ftl_gravitydb` - (default: "/etc/pihole/gravity.db") - Specify path and filename of FTL's SQLite3 gravity database. This database contains all domains relevant for Pi-hole's DNS blocking
* `pi_hole_ftl_debug_all` - (default: false) - Enable all debug flags. If this is set to true, all other debug config options are ignored.
* `pi_hole_ftl_debug_database` - (default: false) - Print debugging information about database actions. This prints performed SQL statements as well as some general information such as the time it took to store the queries and how many have been saved to the database.
* `pi_hole_ftl_debug_networking` - (default: false) - Prints a list of the detected interfaces on the startup of pihole-FTL. Also, prints whether these interfaces are IPv4 or IPv6 interfaces.
* `pi_hole_ftl_debug_locks` - (default: false) - Print information about shared memory locks. Messages will be generated when waiting, obtaining, and releasing a lock.
* `pi_hole_ftl_debug_queries` - (default: false) - Print extensive query information (domains, types, replies, etc.). This has always been part of the legacy debug mode of pihole-FTL.
* `pi_hole_ftl_debug_flags` - (default: false) - Print flags of queries received by the DNS hooks. Only effective when DEBUG_QUERIES is enabled as well.
* `pi_hole_ftl_debug_shmem` - (default: false) - Print information about shared memory buffers. Messages are either about creating or enlarging shmem objects or string injections.
* `pi_hole_ftl_debug_gc` - (default: false) - Print information about garbage collection (GC): What is to be removed, how many have been removed and how long did GC take.
* `pi_hole_ftl_debug_arp` - (default: false) - Print information about ARP table processing: How long did parsing take, whether read MAC addresses are valid, and if the macvendor.db file exists.
* `pi_hole_ftl_debug_regex` - (default: false) - Controls if FTLDNS should print extended details about regex matching into pihole-FTL.log.
Due to legacy reasons, we also support the following setting to be used for enabling the same functionality:
REGEX_DEBUGMODE=false|true Note that if one of them is set to true, the other one cannot be used to disable this setting again.
* `pi_hole_ftl_debug_api` - (default: false) - Print extra debugging information during telnet API calls. Currently only used to send extra information when getting all queries.
* `pi_hole_ftl_debug_overtime` - (default: false) - Print information about overTime memory operations, such as initializing or moving overTime slots.
* `pi_hole_ftl_debug_extbloqued` - (default: false) - Print information about why FTL decided that certain queries were recognized as being externally blocked.
* `pi_hole_ftl_debug_caps` - (default: false) - Print information about capabilities granted to the pihole-FTL process. The current capabilities are printed on receipt of SIGHUP, i.e., the current set of capabilities can be queried without restarting pihole-FTL (by setting DEBUG_CAPS=true and thereafter sending killall -HUP pihole-FTL).
* `pi_hole_ftl_debug_dnsmasq_lines` - (default: false) - Print file and line causing a dnsmasq event into FTL's log files. This is handy to implement additional hooks missing from FTL.
* `pi_hole_ftl_debug_vectors` - (default: false) - FTL uses dynamically allocated vectors for various tasks. This config option enables extensive debugging information such as information about allocation, referencing, deletion, and appending.
* `pi_hole_ftl_debug_resolver` - (default: false) - Extensive information about hostname resolution like which DNS servers are used in the first and second hostname resolving tries (only affecting internally generated PTR queries).
