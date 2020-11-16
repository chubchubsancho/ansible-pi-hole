# ansible-pi-hole

[![Build Status](https://travis-ci.com/chubchubsancho/ansible-pi-hole.svg?branch=master)](https://travis-ci.com/chubchubsancho/ansible-pi-hole)
[![License](https://img.shields.io/badge/license-MIT-blue.svg?logo=github&style=flat)](https://raw.githubusercontent.com/chubchubsancho/ansible-pi-hole/master/LICENSE)

An ansible role to install [Pi-Hole](https://pi-hole.net/)

## Role configuration

* `pi_hole_repo` (default: "https://github.com/pi-hole/pi-hole.git") - define `pi-hole` repository
* `pi_hole_prerequisites` (default: "git") - define `pi-hole` prerequisites

## Define pi-hole configuration

Add `pi-hole` variable for config. These vars can be omited

* `pi_hole_download_dir` (default: "/home/pihole") - define base directory for clone `pi-hole`
* `pi_hole_install_dir` (default: "pi-hole") - define directory where to clone `pi-hole`
* `pi_hole_query_logging` (default: true) - define if `pi-hole` will log query
* `pi_hole_install_web_server` (default: true) - define if `pi-hole` web server need to be installed
* `pi_hole_install_web_interface` (default: true) - define if `pi-hole` web interface need to be installed
* `pi_hole_enable_lighttpd` (default: true) - does `lighttpd` need to be enabled
* `pi_hole_webpasswd` (default: omit) - define `pi-hole` web password if omited one will be generated
* `pi_hole_dnsmasq_listening` (default: "single") -
* `pi_hole_dns_1` (default: "8.8.8.8") - `pi-hole` upstream dns server
* `pi_hole_dns_1_port` (default: "53") - `pi-hole` upstream dns server port
* `pi_hole_dns_2` (default: "8.8.4.4") - `pi-hole` upstream dns server
* `pi_hole_dns_1_port` (default: "53") - `pi-hole` upstream dns server port
* `pi_hole_fqdn_required` (default: false) - does `pi-hole` required fqdn queries
* `pi_hole_bogus_priv` (default: false) - does `pi-hole` will aswer bogus on private zone
* `pi_hole_dnssec` (default: false) - `pi-hole` use dnssec
* `pi_hole_rev_server` (default: true) - does `pi-hole` answer to reverse queries
* `pi_hole_rev_server_domain` (default: "") - `pi-hole` reverse domain name
* `pi_hole_rev_server_target` (default: "") - `pi-hole` reverse server target
* `pi_hole_rev_server_cidr` (default: "") - `pi-hole` reverse cidr
* `pi_hole_blocking_enabled` (default: true) - enable `pi-hole` blocking

## Define pi-hole FTL configuration

Add `pi-hole-FTL` variable config. Thes vars can be omited

*
